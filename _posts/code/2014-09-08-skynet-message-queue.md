---
layout: post
title:  Skynet之消息队列 - 消息的存储与分发
date:   2014-09-08 20:07:00 +0800
categories: code
tags: skynet
---
按我的理解，消息队列是Skynet的核心，Skynet就是围绕着消息队列来工作的。   
这个消息队列分为两部分：全局队列和服务队列。每个服务都有一个自己的服务队列，服务队列被全局队列引用。主进程通过多个线程来不断的从全局队列中取出服务队列，然后分发服务队列中的消息到对应的服务。  

今天，我将拨开消息队列的面纱，一探究竟。   

既然是数据结构，就是用来存储数据的，伴随着它的就要有添加、删除、访问接口。由于它是用来存储消息的，不难想到：向某服务发送消息，就是向服务的服务队列中添加消息。而Skynet是通过多线程来分发消息的，线程的工作就是遍历全局队列，分发服务队列中的消息到服务。  

我就按照这个思路，带着问题，去看看Skynet的实现：

 1. 全局队列和服务队列的结构
 2. 全局队列和服务队列的生成
 4. 如何向全局队列添加/删除服务队列
 5. 如何向服务队列添加/删除消息
 6. 工作线程如何分发消息


----------


## 结构

### 服务队列结构

{% highlight c %}
struct message_queue {
	uint32_t handle;
	int cap;
	int head;
	int tail;
	int lock;
	int release;
	int in_global;
	struct skynet_message *queue;
	struct message_queue *next;
};
{% endhighlight %}

初看此结构，感觉很像链表：next指向下一个节点，queue存储消息数据。其实是错的，稍微思考一下：如果是链表的话，那`message_queue`的其他数据（handle,cap等）岂不是要被复制多份？这显然不符合大神对代码质量的要求。   
既然不是通过链表的方式去实现的，那么很容易就会想到：是通过数组的形式来实现的，`queue`其实是一个动态申请的数组，里面存了很多条消息，而cap（容量）、head（头）、tail（尾）是为`queue`服务的。但是`next`指针又有什么用呢？   
先不管这么多了，继续读代码找答案吧。   

### 全局队列结构

{% highlight c %}
struct global_queue {
	uint32_t head;
	uint32_t tail;
	struct message_queue ** queue;
	struct message_queue *list;
};
{% endhighlight %}


----------


### 生成

#### 全局队列

一个Skynet进程中，只有一个全局队列，在系统启动的时候就会通过`skynet_mq_init`生成它：  

{% highlight c %}
void 
skynet_mq_init() {
	struct global_queue *q = skynet_malloc(sizeof(*q));
	memset(q,0,sizeof(*q));
	q->queue = skynet_malloc(MAX_GLOBAL_MQ * sizeof(struct message_queue *));
	memset(q->queue, 0, sizeof(struct message_queue *) * MAX_GLOBAL_MQ);
	Q=q;
}
{% endhighlight %}

需要注意的是：它直接申请了`MAX_GLOBAL_MQ`个`message_queue`用于存储服务队列，所以服务队列的总数不能超过`MAX_GLOBAL_MQ`。

#### 服务队列

由于服务队列是属于服务的，所以服务队列的生命周期应和服务一致：载入服务的时候生成，卸载服务的时候删除。   
服务是通过`skynet_context_new`载入的，在此函数中，可以找到对应的服务队列的生成语句：

{% highlight c %}
struct message_queue * queue = ctx->queue = skynet_mq_create(ctx->handle);

struct message_queue * 
skynet_mq_create(uint32_t handle) {
	struct message_queue *q = skynet_malloc(sizeof(*q));
	q->handle = handle;
	q->cap = DEFAULT_QUEUE_SIZE;
	q->head = 0;
	q->tail = 0;
	q->lock = 0;
	q->in_global = MQ_IN_GLOBAL;
	q->release = 0;
	q->queue = skynet_malloc(sizeof(struct skynet_message) * q->cap);
	q->next = NULL;

	return q;
}
{% endhighlight %}

在Skynet内部，是通过handle来定位服务的，handle就相当与服务的地址，此函数保存了服务的handle，这样，以后就可以通过服务队列的handle，直接找到对应的服务了。   
默认的容量是`DEFAULT_QUEUE_SIZE`（64），从这里就可以印证我们上面的判断了：`message_queue`是通过数组保存消息的，不是通过链表。


----------


### 全局队列操作

全局队列是一个用固定大小的数组模拟的循环队列，此循环队列向尾部添加，从头部删除，分别用head、tail记录其首尾下标。   
全局队列保存所有的服务队列，worker线程向全局队列索取服务队列。为了效率，并不是简单的把所有的服务队列都塞到全局队列中，而是只塞入非空的服务队列，这样worker线程就不会得到空的服务队列而浪费资源。  
由于工作线程有多个，为了避免冲突，Skynet运用了这样的策略：每次worker线程取得一个服务队列的时候，都把这个服务队列从全局队列中删除，这样其他的worker线程就没法获取到这个服务队列了，当此worker线程操作完毕后，再将此服务队列添加到全局队列（若服务队列非空的话）。

可能触发全局队列添加操作的情况有：

- 向服务队列中添加消息（空变非空）
- worker线程处理完毕，服务队列非空

可能触发全局队列删除操作的情况有：

- 从服务队列中删除消息（非空变空）
- worker线程获取消息队列

#### 添加

{% highlight c %}
void 
skynet_globalmq_push(struct message_queue * queue) {
	struct global_queue *q= Q;
	uint32_t tail = GP(__sync_fetch_and_add(&q->tail,1));
	if (!__sync_bool_compare_and_swap(&q->queue[tail], NULL, queue)) {
		// The queue may full seldom, save queue in list
		assert(queue->next == NULL);
		struct message_queue * last;
		do {
			last = q->list;
			queue->next = last;
		} while(!__sync_bool_compare_and_swap(&q->list, last, queue));

		return;
	}
}
{% endhighlight %}

不要被那些原子操作函数吓倒，它们其实要做的很简单，只是为了保证操作的原子性，防止多线程冲突问题，才单独封装成一个API，详细解释见：[GCC内置原子内存存取函数](http://outsky.org/article.php?id=67)。   
当向这样的固定大小的循环队列添加元素的时候，会遇到如下情况：

- tail溢出
- 队列满了

上述代码中，tail溢出的问题是通过`GP`取模操作来解决的：

{% highlight c %}
#define GP(p) ((p) % MAX_GLOBAL_MQ)
{% endhighlight %}

如果队列满了，怎么办呢？一般的解决办法有：扩大容量、直接返回操作失败等。Skynet没有采用这样的方法，它是这么做的：  

{% highlight c %}
struct message_queue * last;
do {
	last = q->list;
	queue->next = last;
} while(!__sync_bool_compare_and_swap(&q->list, last, queue));
{% endhighlight %}

因为要考虑多线程的问题，代码显的比较难读，我们简化一下：  

{% highlight c %}
queue->next = q->list;
q->list = queue;
{% endhighlight %}

这样就很清晰了，实际上就是：将新的服务队列`queue`添加到全局队列的额外服务队列链表`list`中。这样，`global_queue`的`list`中，就存放了所有没有成功添加的服务队列（因为全局队列满了）。  

#### 删除

删除的算法就很简单了：

 1. 非空检查
 2. 取得head下标，做溢出处理（GP）
 3. 取出当前的头节点
 4. 将head下标对应的指针值空
 5. head加1

这里有一个细节，还记得上面的添加操作有可能遇到全局队列满的情况吗？这里会尝试将那些添加失败的队列添加到全局队列中：

{% highlight c %}
struct message_queue * list = q->list;
if (list) {
	struct message_queue * newhead = list->next;
	if (__sync_bool_compare_and_swap(&q->list, list, newhead)) {
		list->next = NULL;
		skynet_globalmq_push(list);
	}
}
{% endhighlight %}

因为每次都只会pop一个，所以，每次只从list中取一个push进全局队列。

----------


### 服务队列操作

服务队列中存储了所有发给此服务的消息。  
服务队列是可变大小的循环队列，其容量会在运行时动态增加。

#### 添加

通过调用`skynet_mq_push`来将消息添加到服务队列：  

{% highlight c %}
void 
skynet_mq_push(struct message_queue *q, struct skynet_message *message) {
	q->queue[q->tail] = *message;
	if (++ q->tail >= q->cap)
		q->tail = 0;

	if (q->head == q->tail)
		expand_queue(q);

	if (q->in_global == 0) {
		q->in_global = MQ_IN_GLOBAL;
		skynet_globalmq_push(q);
	}
}
{% endhighlight %}

同全局队列一样，它也会遇到：下标溢出、队列满的情况，由于它是可扩容的循环队列，当队列满的时候，就调用`expand_queue`来扩容（当前容量的两倍）。   
这里需要注意的是，最后做了这样的处理：如果当前的服务队列没有被添加到全局队列，则将它添加进去，这是为worker线程而做的优化。

#### 删除

删除的操作就很简单了：head+1。   
细节上考虑了下标溢出的问题，并会在队列为空的时候，将队列的`in_global`值为false。   
为什么这里只设置一个标记呢？为什么不从全局队列中删除呢？    
哈哈！因为只有worker线程才会操作服务队列，而当worker线程获取到服务队列的时候，已经将它从全局队列中删除了。   

----------

### 消息分发

消息分发是通过启动多个worker线程来做的，而worker线程则不断的循环调用`skynet_context_message_dispatch`，为了便于理解，我删掉了一些细节：

{% highlight c %}
struct message_queue * 
skynet_context_message_dispatch(struct message_queue *q) {
	if (q == NULL) {
		q = skynet_globalmq_pop();
		if (q==NULL)
			return NULL;
	}

	uint32_t handle = skynet_mq_handle(q);
	struct skynet_context * ctx = skynet_handle_grab(handle);

	struct skynet_message msg;
	if (skynet_mq_pop(q,&msg)) {
		skynet_context_release(ctx);
		return skynet_globalmq_pop();
	}
	_dispatch_message(ctx, &msg);

	struct message_queue *nq = skynet_globalmq_pop();
	if (nq) {
		skynet_globalmq_push(q);
		q = nq;
	} 
	skynet_context_release(ctx);

	return q;
}
{% endhighlight %}

这个函数有两种情况：

1. 传入的`message_queue`为NULL
2. 传入的`message_queue`非NULL

对于第一种情况，它会到全局队列中pop一个出来，后面的和第二种情况一样了。   

分发步骤如下：

1. 通过`message_queue`获得服务的handle
2. 通过handle查找到服务的`skynet_context`
3. 从`message_queue`中pop一个元素
4. 调用`_dispatch_message`进行消息分发
5. 如果全局队列为空，则直接返回此队列（这样下次就会继续处理这个队列，此函数是循环调用的）
6. 如果全局队列非空，则pop全局队列，得到下一个服务队列
7. 将此队列插入全局队列，返回下一个服务队列

只所以不一次性处理玩当前队列，而要用5～7的步骤，是为了消息调度的公平性，对每一个服务都公平。  

`_dispatch_message`如下：

{% highlight c %}
static void
_dispatch_message(struct skynet_context *ctx, struct skynet_message *msg) {
	int type = msg->sz >> HANDLE_REMOTE_SHIFT;
	size_t sz = msg->sz & HANDLE_MASK;
	if (!ctx->cb(ctx, ctx->cb_ud, type, msg->session, msg->source, msg->data, sz))
		skynet_free(msg->data);
}
{% endhighlight %}

它从`skynet_message`消息中分解出类型和大小，然后调用服务的callback。  
这里需要注意的是：如果消息的callback返回0，则消息的`data`将被释放。