---
layout: post
title:  Linux忘记密码的解决办法
date:   2014-08-15 10:43:00 +0800
categories: os
tags: linux
---
 - 开机
 - 在grub启动列表中按ESC
 - 按 e 来修改启动选项
 - 在kernel*那一行再按 e
 - 在最后面加上 `init=/bin/bash`
 - 回车确定
 - 按 b 来启动
 - 用 passwd <用户名> 来重设密码
    - 如果出现authentication token lock busy错误，执行：mount -o remount,rw /命令，此命令重新挂载根分区为读写模式(默认为只读模式)
 - 重启
