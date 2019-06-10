---
layout: post
title:  有用的shell小功能
date:   2014-03-11 16:30:00 +0800
categories: code
tags: shell
---

### 获取本机内网IP

<pre>
/sbin/ifconfig | grep "inet addr" -m 1 | cut -d ":" -f 2 | cut -d " " -f 1
</pre>

### 输出unix时间戳

<pre>
date +%s
</pre>

### 从路径中获取文件名

<pre>
basename /usr/bin/sort
    Output "sort". 
basename include/stdio.h .h
    Output "stdio".
</pre>

### 遍历指定路径文件

{% highlight shell %}
file_list=`ls dir`
for file in $file_list
do
	echo $file
done
{% endhighlight %}

### gdb attach指定进程名的进程

{% highlight shell %}
#! /bin/sh

if [ $# -lt 1 ]; then
    echo "please give a name"
    exit 1
fi

ENTRY=`ps x | grep $1 -m 1`
PID=`echo $ENTRY | cut -d " " -f 1`
while true; do
    echo -e "The following process will be attached by gdb\n\n  ${ENTRY}\n"
    read -p "Continue? (y or n) " confirm
    case $confirm in
        [y] )  gdb attach $PID; break;;
        [n] )  exit 1;;
        *) echo "Please answer y or n.";;
    esac
done
{% endhighlight %}

### 批量查找替换文本字符串
将当前目录下，所有的.h文件中的wrong替换为right  

<pre>
find . -type f -name "*.h" | xargs perl -pi -e 's|wrong|right|g'
</pre>

### 借助expect实现自动scp

{% highlight shell %}
#!/bin/sh

help()
{
    echo "Usage: $0 local dest"
}

if [ $# != 2 ]; then
    help
    exit 1
fi

echo "[!] uploading, please wait..."

user=root
passwd=abc123
ip=1.2.3.4
local=$1
dest=$2

expect -c "
    set timeout 100000;
    spawn scp -r ${local} ${user}@${ip}:${dest}
    expect {
        \"*assword\" {send \"${passwd}\r\";}
        \"yes/no\" {send \"yes\r\"; exp_contine;}
    }
expect eof"

echo "[ok] uploaded."
{% endhighlight %}
