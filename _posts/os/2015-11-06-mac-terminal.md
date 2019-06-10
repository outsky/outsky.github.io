---
layout: post
title:  打磨mac终端
date:   2015-11-06 23:53:00 +0800
categories: os
tags: mac
---
mac终端有两点不爽：  

* 执行exit命令不退出窗口  
* 命令行头部信息太啰嗦

- - -

# 让exit直接退出  
![](/res/img/20151106/1.c.png)  
打开终端的Profiles > Shell，设置When the shell exits选项为 Close  
![](/res/img/20151106/2.c.png)  
**可以修改默认终端配色方案，如图所示，将自己喜欢的配色方案设置为Default**


# 简化命令行行首
![](/res/img/20151106/3.c.png)  
这么罗嗦的行首，提供的信息这么无用，真是浪费。  

打开~/.bash_profile，添加：

<pre>
export PS1="\u: \w$ "
</pre>

![](/res/img/20151106/5.c.png)  
结果如图：  
![](/res/img/20151106/4.c.png)  
