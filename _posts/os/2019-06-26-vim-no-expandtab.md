---
layout: post
title:  vim临时不展开tab为空格
date:   2019-06-26 10:55:00 +0800
categories: os
tags: vim
---

使用vim的时候，习惯设置默认将<tab>转为空格，但在有些情况下，并不希望<tab>被转为空格，比如makefile要求必须用<tab>缩进，这时候可以通过如下方式临时禁止<tab>转为空格：

- 进入编辑模式
- 按<control> + <v>
- 按tab
