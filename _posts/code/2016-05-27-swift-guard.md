---
layout: post
title:  guard和if的区别
date:   2016-05-27 16:32:00 +0800
categories: code
tags: swift
---

guard与if类似，根据条件判断执行对应的语句。  
guard可以使代码更整洁、易读。  

分别用if和guard来实现函数greet，进行对比，这个函数的功能是：  
接收一个dictionary参数，参数中可设置name和location，根据参数内容，输出不同的问候语：  

{% highlight swift %}
greet([:])
// Prints "Hello stranger!"

greet(["name": "John"])
// Prints "Hello John!"
// Prints "I hope the weather is nice near you."

greet(["name": "Jane", "location": "Cupertino"])
// Prints "Hello Jane!"
// Prints "I hope the weather is nice in Cupertino."
{% endhighlight %}

## if版本：  

{% highlight swift %}
func greet(person: [String: String]) {
	if let name = person["name"] {
		print("Hello \(name)!")
		
		if let location = person["location"] {
			print("I hope the weather is nice in \(location).")
		} else {
			print("I hope the weather is nice near you.")
		}
	} else {
		print("Hello stranger!")
	}
}
{% endhighlight %}
 
## guard版本：

{% highlight swift %}
func greet(person: [String: String]) {
	guard let name = person["name"] else {
		print("Hello stranger!")
		return
	}
	print("Hello \(name)!")

	guard let location = person["location"] else {
		print("I hope the weather is nice near you.")
		return
	}
	print("I hope the weather is nice in \(location).")
}
{% endhighlight %}

对比可以看出，guard版本更清晰易读，if版本的一连串的if/else让人头晕。

--------

## guard语法

{% highlight swift %}
guard condition else {
    statements
}
{% endhighlight %}

如果condition为true，则继续往下执行，否则，执行else。  
else必须退出当前作用域或调用noreturn函数，以处理非法情况。  

---------

## 区别

### 判断失败处理位置

- if判断失败的处理在很远的else中，跨度很大
- guard判断失败的处理在紧跟着的else中，阅读上没有断层

### optional binding变量有效范围

- if中只在其括号内有效
- guard中在其定义范围内有效

### else

- if中，else可选
- guard中，else必须有，且必须退出当前作用域或调用noreturn函数

---------

## 使用选择

- 当需要对optional binding做判断处理时，选用guard
- 若仅仅是判断bool条件，则选择if可能更好一些(if !condition { return })

---------

## 参考

[Lang Ref Guard Statement](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Statements.html#//apple_ref/swift/grammar/guard-statement)  
[Lang Guide Early Exit](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/ControlFlow.html#//apple_ref/doc/uid/TP40014097-CH9-ID525)