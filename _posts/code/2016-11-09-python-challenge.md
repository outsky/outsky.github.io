---
layout: post
title:  Python Challenge
date:   2016-11-09 23:07:00 +0800
categories: code
tags: python
---

发现一个有趣的网站：[The Python Challenge](http://www.pythonchallenge.com)

-----

## 0 - warming up

2**38 = 274877906944

## 1 - What about making trans?

看图: K->M, O->Q, E->G，猜规律为：将字母后移两位，带入密文前几位验证可行，写代码解密

{% highlight python %}
def decode(code):
    trans = string.maketrans("abcdefghijklmnopqrstuvwxyz", "cdefghijklmnopqrstuvwxyzab")
    return code.translate(trans)
{% endhighlight %}

解密后得到：

> i hope you didnt translate it by hand. thats what computers are for. doing it in by hand is inefficient and that's why this text is so long. using string.maketrans() is recommended. now apply on the url.

对url执行decode，得到jvvr://yyy.ravjqpejcnngpig.eqo/re/fgh/ocr.jvon

答案：ocr

## 2 - ocr

从字符串中找字母：

{% highlight python %}
def findalpha(code):
    for _,c in enumerate(code):
        if c.isalpha():
            print(c)
{% endhighlight %}

答案：equality

## 3 - re

看图，一个小蜡烛，两边分别有三个大蜡烛，加上文字说明，理解为：找出两边都被刚好3个大写字母包围的小写字母

{% highlight python %}
def func(code):
    letters = re.findall("[^A-Z][A-Z]{3}([a-z])[A-Z]{3}[^A-Z]", code)
    print("".join(letters))
{% endhighlight %}

答案：linkedlist

## 4 - follow the chain

{% highlight python %}
def getnext(n):
    f = urllib.urlopen("http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing={}".format(n))
    txt = f.read()
    f.close()
    print(n, txt)
    match = re.findall("next nothing is (\d+)", txt)
    if len(match)<=0:
        return None
    return match[0]
{% endhighlight %}

答案：peak.html

## 5 - peak hell

比较刁钻，源代码中隐藏着banner.p，peak读音联想到pickle，用pickle载入后得到一个list

<pre>
list = [
    row1, row2, ...
]
row = [(char1, count1), (char2, count2), ...]
</pre>

这是一副图画，按行存储，行的数据表示为：字符+连续个数

{% highlight python %}
def func():
    with open("banner.p") as f:
        o = pickle.load(f)
        for row in o:
            str = ""
            for (c, count) in row:
                str += c*count
            print(str)
{% endhighlight %}

答案：channel

## 6 - now there are pairs

太刁钻了！

源代码中有zip提示，修改url为zip.html，提示find the zip，修改为channel.zip成功下载zip文件

zip文件解压看到readme的提示，按照类似于第4题的方式，跟踪发现提示：Collect the comments.

就是收集zip文件中按指定顺序访问文件的ZipInfo的comment

{% highlight python %}
def func():
    z = zipfile.ZipFile("channel.zip")
    n = 90052
    comments = ""
    while True:
        filename = "{}.txt".format(n)
        with z.open(filename) as f:
            comments += z.getinfo(filename).comment
            txt = f.read()
            print(n, txt)
            match = re.findall("Next nothing is (\d+)", txt)
            if len(match)<=0:
                break
            n = match[0]
    print(comments)
{% endhighlight %}

得到：

<pre>
****************************************************************
****************************************************************
**                                                            **
**   OO    OO    XX      YYYY    GG    GG  EEEEEE NN      NN  **
**   OO    OO  XXXXXX   YYYYYY   GG   GG   EEEEEE  NN    NN   **
**   OO    OO XXX  XXX YYY   YY  GG GG     EE       NN  NN    **
**   OOOOOOOO XX    XX YY        GGG       EEEEE     NNNN     **
**   OOOOOOOO XX    XX YY        GGG       EEEEE      NN      **
**   OO    OO XXX  XXX YYY   YY  GG GG     EE         NN      **
**   OO    OO  XXXXXX   YYYYYY   GG   GG   EEEEEE     NN      **
**   OO    OO    XX      YYYY    GG    GG  EEEEEE     NN      **
**                                                            **
****************************************************************
 **************************************************************
</pre>

输入hockey，提示：it's in the air. look at the letters.

:)

观察“字母”，答案是：oxygen（确实in the air)

## 7 - smarty

[http://www.pythonchallenge.com/pc/def/oxygen.html](http://www.pythonchallenge.com/pc/def/oxygen.html)

