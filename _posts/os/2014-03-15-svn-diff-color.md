---
layout: post
title:  给svn diff加上颜色
date:   2014-03-15 15:23:00 +0800
categories: os
tags: svn
---
svn diff命令没有任何颜色区分，难以审查，有一款开源的软件：[colordiff](http://www.colordiff.org/) 可以给svn diff加上颜色。

# 安装步骤  

* Centos

<pre>
$ wget http://www.colordiff.org/colordiff-1.0.13.tar.gz
$ tar -xvf colordiff-1.0.13.tar.gz
$ cd colordiff-1.0.13
$ sudo make install
</pre>

* Ubuntu

<pre>
sudo apt-get install colordiff
</pre>

# 设置svn

<pre>
$ vi ~/.subversion/config 
</pre>

修改diff-cmd = colordiff

{% highlight ini %}
### Section for configuring external helper applications.
[helpers]
### Set diff-cmd to the absolute path of your 'diff' program.
###   This will override the compile-time default, which is to use
###   Subversion's internal diff implementation.
diff-cmd = colordiff
{% endhighlight %}
