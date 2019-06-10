---
layout: post
title:  启用自动生成core文件
date:   2014-03-10 18:51:00 +0800
categories: os
tags: linux
---
core文件对bug的查找调试是极其重要的，linux系统默认是禁用core文件的生成的，需要我们手动设置。  
只需在~/.bash_profile中添加ulimit -c unlimited即可。  
如下：  
<pre>
$ echo "ulimit -c unlimited" >> ~/.bash_profile
$ source ~/.bash_profile
</pre>

查看是否设置成功：
<pre>
$ ulimit -c
unlimited
</pre>
