---
layout: post
title:  Skynet之斗转星移 - 将控制权交给Lua
date:   2014-09-08 03:28:00 +0800
categories: code
tags: skynet
---
在我看来，Skynet的一个重要优势是与Lua的高度结合，完全可以用Lua写服务。用C写服务的原理很简单：通过动态链接库的形式，提供create、init和release接口，供主进程在需要的时候载入服务，并将处理消息的回调函数一并注入主进程，这样，当主进程给此服务发消息时，消息就进入此回调函数处理。  

由此可见，服务最重要的部分就是这个回调函数了，如果要用Lua来写服务的话，如何注册消息回调函数呢？且听我娓娓道来 :-)  


----------


## 从skynet.newservice看起

从Skynet的例子程序入口，它是这么启动一个Lua服务的：  

{% highlight lua %}
skynet.newservice("simpledb")
{% endhighlight %}

跟踪发现，它最终调用了一个C导出函数：   

{% highlight lua %}
-- destination: ".launcher"
-- type: PTYPE_LUA
-- session: 0
-- message: pack("LAUNCH", "snlua", "simpledb")
c.send(".launcher", skynet.PTYPE_LUA, nil, 
	skynet.pack("LAUNCH", "snlua", "simpledb"))
{% endhighlight %}

此函数的C源码在`lualib-src/lua-skynet.c:_send`，以上调用，意思就是：将数据包封装成消息发送给`.launcher`服务处理。  


----------

## 半路杀出个.launcher

接下来，就要找到`.launcher`服务，看看它怎么处理这个消息。  
经过查找，发现`.launcher`服务是在`service/bootstrap.lua`中被注册的：  

{% highlight lua %}
local launcher = skynet.launch("snlua", "launcher")
skynet.name(".launcher", launcher)
{% endhighlight %}

原来`.launcher`服务也是用Lua写的，大家不要慌，不要在乎这些细节，保持头脑清（蛋）醒（疼）。  
虽然不明白这个`skynet.launch`是干什么的，但是感觉很厉害的样子。OK，跟进去看看。  

不看不知道，一看吓一跳！  
原来，它载入了Lua服务，并将注册后的服务的handle返回。  
看来它是关键了，还等什么呢，快跟进去看看吧 :-)  

它调用C接口`skynet-src/skynet_server.c:cmd_launch`:  

{% highlight c %}
cmd_launch(ctx, "snlua launcher")
{% endhighlight %}

此函数通过分析，调用`skynet_context_new`：

{% highlight c %}
skynet_context_new("snlua", "launcher")
{% endhighlight %}

`skynet_context_new`就很简单了：

- 获取/加载C服务的动态库（snlua.so）
- 调用动态库的create函数（`snlua_create`），获取服务的实例(`snlua*`)
- 调用动态库的init函数（`snlua_init`），并将参数（`"launcher"`）传入
- 为服务分配`skynet_context`，并生成一个唯一的`handle`
- 为服务分配消息队列，并将`skynet_context`通过`handle`与之绑定
- 将消息队列压入全局消息队列中

由此可见，服务snlua是用C写的，且其create和init函数被相继调用，这两个函数做了服务的初始化工作。  

{% highlight c %}
struct snlua *
snlua_create(void) {
	struct snlua * l = skynet_malloc(sizeof(*l));
	memset(l,0,sizeof(*l));
	l->L = lua_newstate(skynet_lalloc, NULL);
	return l;
}
{% endhighlight %}

值得注意的是：`snlua_create`创建了全新的`lua_State`，这样就保证了每个Lua服务有自己独立的Lua虚拟机，互不影响。  

{% highlight c %}
// 在我们的跟踪中，args为"launcher"
int
snlua_init(struct snlua *l, struct skynet_context *ctx, const char * args) {
	...
	skynet_callback(ctx, l , _launch);
	...
	// it must be first message
	// handle_id是它自己的handle
	skynet_send(ctx, 0, handle_id, PTYPE_TAG_DONTCOPY,0, tmp, sz);
	return 0;
}
{% endhighlight %}

和其他C服务一样，`snlua_init`设置了它的消息回调函数（`_launch`），这样以后发给它的消息都会交给`_launch`处理。  
然后，它向自己发送了消息：`"launcher"`，此消息将被`_launch`处理：  

{% highlight c %}
// 在我们的跟踪过程中args为"launcher"
static int
_launch(struct skynet_context * context, void *ud, int type, int session, uint32_t source , const void * msg, size_t sz) {
	struct snlua *l = ud;
	skynet_callback(context, NULL, NULL);
	_init(l, context, msg, sz);
	...
}
{% endhighlight %}

这里将snlua服务的callback值空，就导致下次发给此服务的消息不会进入`_launch`，那以后的消息怎么办？直接丢掉吗？跟下去就知道了。  

{% highlight c %}
static int
_init(struct snlua *l, struct skynet_context *ctx, const char * args, size_t sz) {
	lua_State *L = l->L;
	...
	lua_pushlightuserdata(L, ctx);
	lua_setfield(L, LUA_REGISTRYINDEX, "skynet_context");
	...

	const char * loader = optstring(ctx, "lualoader", "./lualib/loader.lua");
	int r = luaL_loadfile(L,loader);
	lua_pushlstring(L, args, sz); // 在我们的跟踪过程中args为"launcher"
	r = lua_pcall(L,1,0,1);
	...
}
{% endhighlight %}

`_init`对新生成的`lua_State`做必要的设置，比如查找路径等。并将`skynet_context`压入注册表（registry），供以后使用（其实是为Lua提供接口的skynet.so需要它）。   
然后，它载入`loader.lua`并将参数（`"launcher"`）传递过去。  

`loader.lua`的功能很简单：找到参数（`"launcher"`）指定的Lua文件，运行它。  

好了，终于出来了，现在找到`launcher.lua`，看看它是干啥的。  
它主要调用了这三个函数：`skynet.register_protocol`、`skynet.dispatch`和`skynet.start`。  

`skynet.register_protocol`将指定的协议记录到一张表里面，后续通过协议的名字（如：`"lua"`）或id（如：`skynet.PTYPE_LUA`）都可以找到此协议。  

`skynet.dispatch`设置指定协议的`dispatch`方法。  

`skynet.start`比较有意思：

{% highlight lua %}
function skynet.start(start_func)
	c.callback(dispatch_message)
	skynet.timeout(0, function()
		init_service(start_func)
	end)
end
{% endhighlight %}

之前`snlua`将callback函数值空，现在在这里被重新赋值了，以后发过来的消息都会被`dispatch_message`处理。  
然后就调用传入`skynet.start`的函数。
（`skynet.lua`比较复杂，看来有必要好好研究研究再写一篇研究报告出来，现在先略过吧 - -）  

至此，`.launcher`服务我们已经跟到底了：

- 通过snlua（C服务）为自己生成一套服务所需的数据（skynet_context、handle、message queue、callback等），并将自己注册进主进程，接受消息调度
- 偷梁换柱的将snlua的callback换成自己的Lua版本
- 通过`skynet.register_protocol`和`skynet.dispatch`注册某种消息的处理函数


----------


## 不忘初心

虽然上面的跟踪，已经可以看出Skynet斗转星移的过程了，可是我们开头是从例子`skynet.newservice("simpledb")`开始的，所以，还得回去看看。  

{% highlight lua %}
c.send(".launcher", skynet.PTYPE_LUA, nil, 
	skynet.pack("LAUNCH", "snlua", "simpledb"))
{% endhighlight %}

这句会被塞入`.launcher`的消息队列，进而被dispatch到它的`skynet.PTYPE_LUA`协议处理函数中处理：

{% highlight lua %}
-- cmd: "LAUNCH"
-- ...: "snlua", "simpledb"
skynet.dispatch("lua", function(session, address, cmd, ...)
	...
	local f = command[cmd]
	...
	local ret = f(address, ...)
	...
end)

-- service: "snlua"
-- ...: "simpledb"
function command.LAUNCH(_, service, ...)
	launch_service(service, ...)
	...
end

-- service: "snlua"
-- ...: "simpledb"
local function launch_service(service, ...)
	...
	local inst = skynet.launch(service, param)
	...
end
{% endhighlight %}

看到了吧，最终还是回到了上面跟过的`skynet.launch`，只不过上次传的是`"launcher"`这次传的是`"simpledb"`。  
结果也一样，通过snlua，将自己注册到主进程，然后偷偷的把消息回调函数换成自己的。


----------


## 总结一下

- 通过`skynet.newservice`载入一个Lua服务
- snlua帮助Lua服务做一些底层的工作
	- 生成一个独立的`lua_State`
	- 生成一个`skynet_context`
	- 分配一个`handle`
	- 生成一个私有的消息队列
	- `skynet_context`和消息队列通过`handle`关联
- 将消息回调从snlua转移到Lua层（`skynet.lua:dispatch_message`)

