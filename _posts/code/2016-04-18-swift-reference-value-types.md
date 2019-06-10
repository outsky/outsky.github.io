---
layout: post
title:  Swift中的值类型和引用类型
date:   2016-04-18 1:02:00 +0800
categories: code
tags: swift
---

在Swift中，class是按引用类型传递的，struct是按值传递。

## 区别

值类型与引用类型的区别是：

 - 在赋值、传参的时候，值类型的变量是被拷贝过去的，是原变量的一个副本
 - 引用类型的变量是原变量的引用，就像C里面的指针，指向同一个对象

### 值类型

{% highlight swift %}
// Value Types:
struct Cat {
  var wasFed = false
}
 
var cat = Cat()
var kitty = cat
kitty.wasFed = true

print("\(cat.wasFed), \(kitty.wasFed)")	// false, true
{% endhighlight %}

![](/res/img/20160418/1.c.png)

### 引用类型

{% highlight swift %}
// Reference Types:
class Dog {
  var wasFed = false
}

let dog = Dog()
let puppy = dog
puppy.wasFed = true

print("\(dog.wasFed), \(puppy.wasFed)")     // true, true
{% endhighlight %}

![](/res/img/20160418/2.c.png)

## 常量(let)

let在值和引用类型上的作用不同:

 - let修饰引用类型，可以修改所指对象，但不可改变变量本身
 - let修饰值类型，不可以修改变量（变量就是其对象）

注意上例代码中的puppy，它用let定义，但还是可以修改，这是因为let修饰的引用变量，而不是其所指向的对象，在这里，puppy本身不可以修改，但它指向的对象的值可以修改。

而对于值类型而言，如果例子中的kitty被定义为let kitty，则kitty.wasFed = true就会报错。

## 参考
[Value and Reference Types](https://developer.apple.com/swift/blog/?id=10)

[Reference vs Value Types in Swift: Part 1/2](https://www.raywenderlich.com/112027/reference-value-types-in-swift-part-1)
