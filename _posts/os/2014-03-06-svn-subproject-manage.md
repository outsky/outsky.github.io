---
layout: post
title:  svn项目子集管理
date:   2014-03-06 13:40:00 +0800
categories: os
tags: svn
---
在项目开发过程中，因为职位的不同，往往只负责维护整个项目的其中一小部分，即项目子集。  
假设项目整体结构为：
<pre>
proj\
    |_common\
    |_server\
    |    |_dir1\
    |    |_dir2\
    |    |_unwanted_dir\
    |_client\
    |	|_folder1\
    |    |_folder2\
    |    |_folder3\
    |_art\
    |_sound\
    |_etc\
</pre>
作为服务端开发人员，我只关心：
<pre>
proj/common/
proj/server/dir1/
proj/server/dir2/
</pre>
为了删繁就简，我希望删掉比如proj/client这类我不需关心的子集，而只保留上述3个文件夹。  
svn提供了这类需求的支持：通过设置文件夹的depth，来达到此目的。  
操作命令如下：
{% highlight shell %}
$ svn co svn://192.168.1.1/proj --depth empty
$ svn up proj/common --set-depth infinity
$ svn up proj/server --set-depth infinity
$ svn up proj/server/unwanted_dir --set-depth exclude
{% endhighlight %}
这样，就达到了维护项目子集的目的，只关心我们所维护的子集的更改删除等状态了。

详细资料：[http://svnbook.red-bean.com/nightly/en/svn.advanced.sparsedirs.html](http://svnbook.red-bean.com/nightly/en/svn.advanced.sparsedirs.html)
