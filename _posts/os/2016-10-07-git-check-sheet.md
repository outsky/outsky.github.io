---
layout: post
title:  Git常用命令
date:   2016-10-07 15:05:00 +0800
categories: os
tags: git
---

# 修改

{% highlight shell%}
$ git reset [file]
{% endhighlight %}

把待commit的文件置为未add的状态，不修改其内容

-------

{% highlight shell%}
$ git diff --staged
{% endhighlight %}

查看待commit的文件的修改差异

-------

# 分支

{% highlight shell%}
$ git branch
{% endhighlight %}

列出所有本地分支

-------

{% highlight shell%}
$ git branch [branch-name]
{% endhighlight %}

创建新分支

-------

{% highlight shell%}
$ git checkout [branch-name]
{% endhighlight %}

切换到指定分支

-------

{% highlight shell%}
$ git merge [branch]
{% endhighlight %}

合并指定分支到当前分支

-------

{% highlight shell%}
$ git branch -d [branch-name]
{% endhighlight %}

删除指定分支

-------

# 文件改名

{% highlight shell%}
$ git rm [file]
{% endhighlight %}

删除文件，并自动将本次操作置为待提交状态

-------

{% highlight shell%}
$ git rm --cached [file]
{% endhighlight %}

删除仓库内文件，但保留本地

-------

{% highlight shell%}
$ git mv [file-original] [file-renamed]
{% endhighlight %}

重命名文件，并置为待提交状态

-------

# 忽略

{% highlight shell%}
$ git ls-files --other --ignored --exclude-standard
{% endhighlight %}

列出所有忽略的文件

-------

# 临时保存

{% highlight shell%}
$ git stash
{% endhighlight %}

临时保存所有已加入版本库的文件

-------

{% highlight shell%}
$ git stash list
{% endhighlight %}

列出所有临时保存的记录

-------

{% highlight shell%}
$ git stash pop
{% endhighlight %}

恢复最近一次临时保存文件

-------

{% highlight shell%}
$ git stash drop
{% endhighlight %}

丢弃最近一次临时保存

-------

# 日志

{% highlight shell%}
$ git log --follow [file]
{% endhighlight %}

查看指定文件的修改历史，包括重命名记录

-------

{% highlight shell%}
$ git diff [first-branch]...[second-branch]
{% endhighlight %}

查看两个分支的差异

-------

{% highlight shell%}
$ git show [commit]
{% endhighlight %}

查看指定提交的修改内容

-------

# 撤销

{% highlight shell%}
$ git reset [commit]
{% endhighlight %}

撤销指定提交后的所有修改，保留本地的修改

-------

{% highlight shell%}
$ git reset --hard [commit]
{% endhighlight %}

回滚到指定的提交，并丢弃指定提交之后的所有提交

-------

# 资料下载

[github-git-cheat-sheet.pdf](/res/files/20161007/github-git-cheat-sheet.pdf)
