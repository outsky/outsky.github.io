---
layout: post
title:  Git配置
date:   2014-08-28 12:32:00 +0800
categories: os
tags: git
---
1) 设置名字和邮箱地址  

<pre>
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
</pre>

2) 默认编辑器

<pre>
$ git config --global core.editor vim
</pre>

3) 默认终端着色

<pre>
$ git config --global color.ui true
</pre>

4) 命令缩写

<pre>
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.st status
</pre>

5) 设置默认push模式

<pre>
git config --global push.default simple
</pre>