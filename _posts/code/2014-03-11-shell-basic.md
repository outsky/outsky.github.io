---
layout: post
title:  shell基础语法
date:   2014-03-11 15:15:00 +0800
categories: code
tags: shell
---
## 文件头部
\#!/bin/sh

## 注释
\# line comment

## 参数

* 参数列表: $* 或 $@
* 获取参数: $0(脚本名), $1 .. $9(参数1 .. 参数9)
* 参数个数: $#

## 重定向

* cmd > file
    cmd输出到文件file中(创建或覆盖)
* cmd >> file
    cmd输出到文件结尾（创建或追加）
* cmd < file
    读取file中的数据作为cmd的输入

## 后台执行

<pre>
$ cmd &
</pre>

## if语句

<pre>
if condition ; then
    ...
elif condition ; then
    ...
else
    ...
fi
</pre>

#### condition语法

* if test 5 -eq 2
* if [ 5 -eq 2]
* if cat a.txt

#### 数字操作符

* -eq: ==
* -ne: !=
* -lt: <
* -le: <=
* -gt: >
* -ge: >=

#### 字符串操作符

* str1 = str2
* str1 != str2
* str1: str1已定义且不为NULL
* -n str1: str1存在且不为NULL
* -z str1: str1存在但为NULL

#### 文件和目录

* -s file: 文件非空
* -f file: 文件存在，且不是目录
* -d dir: 目录存在，且不是文件
* -w file: 文件可写
* -r file: 文件只读
* -x file: 文件可执行

#### 逻辑操作符

* ! exp: 逻辑非
* exp1 -a exp2: 逻辑与
* exp1 -o exp2: 逻辑或

## for循环

#### 格式一

<pre>
for { var name } in { list }
do
    ...
done

for i in 1 2 3 4
do
    echo $i
done
</pre>

#### 格式二

<pre>
for (( exp1; exp2; exp3 ))
do
    ...
done

for (( i=0; i<10; ++i))
do
    echo $i
done
</pre>

## while循环

<pre>
while [ condition ]
do
    ...
done

i=1
while [ $i -le 10 ]
do
    echo $i
    i=`expr $i + 1`
done
</pre>

## case语句

<pre>
case $var in
    pattern1)   cmd1;;
    pattern2)   cmd2;;
    *)  default cmd;;
esac

case $1 in
    1) echo "one";;
    3) echo "three";;
    "hello") echo "world";;
    *) echo "unknown $1";;
esac
</pre>

## 丢弃不需要的输出

<pre>
cmd > /dev/null
</pre>

## 条件执行

* cmd1 && cmd2
    只有在cmd1的exit status为0的时候，才执行cmd2
* cmd1 || cmd2
    只有在cmd1的exit status不为0的时候，才执行cmd2

## I/O重定向

* stdin(0)
* stdout(1)
* stderr(2)  

<pre>
2>err.log #重定向stderr到文件err.log
1>&2 #重定向stdout到stderr
</pre>

## function语法

<pre>
function-name ()
{
    ...
}

function-name # call function
</pre>

## trap命令
trap {commands} {signal number}  
在脚本收到某信号的时候，调用设置的命令(函数)

* 0 shell exit
* 1 hangup
* 2 interrupt(ctrl+c)
* 3 quit

<pre>
test()
{
    while [ 0 ]
    do
        echo "hello"
        touch "`date`.txt"
        sleep 1
    done
}

rm_txt()
{
    rm -f *.txt
    exit -1
}

trap rm_txt 2
test
</pre>

## getopts命令

getopts {optstring} {variable}  
optstring: 

* 包含想要识别的选项字母列表
* 若某选项需要一个参数，则字母后面要加个冒号
* 在传参的时候，要用空格将选项字母和选项值分开
* 选项值将被保存在OPTARG中
* 若某个非法的参数被传入，则variable变量中存问号(?)

<pre>
while getopts a:bc:d opt
do
    case "$opt" in
        a) echo "about $OPTARG";;
        b) echo "brother";;
        c) echo "china $OPTARG";;
        d) echo "design $OPTARG";;
    esac
done


$ ./test.sh -a hello -b -c world
about hello
brother
china world
</pre>
