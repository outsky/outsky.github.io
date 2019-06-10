---
layout: post
title:  gdb保存断点
date:   2014-02-26 10:08:00 +0800
categories: os
tags: gdb
---
gdb调试程序的时候，有时需要多次启动调试，因此，保存断点信息可以使调试更方便。  
gdb有个-x选项，意思是从文件中执行gdb命令：
<pre>
      -x FILE, -command=FILE
              Execute GDB commands from file file.

</pre>
可以将设置断点的命令保存到文件中，然后-x file，这样就可以从文件中加载并设置断点了。  
如：  
{% highlight shell %}
$ touch break.info
$ echo b file1.cpp:123 >> break.info
$ echo b file2.cpp:456 >> break.info
$ gdb a.out -x break.info
{% endhighlight %}
