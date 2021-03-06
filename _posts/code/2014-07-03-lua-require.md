---
layout: post
title:  Lua的require机制
date:   2014-07-03 18:06:00 +0800
categories: code
tags: lua
---
今天仔细读了文档，弄清楚了Lua的模块require机制。  
Lua是通过require函数来加载模块的，只需提供模块的名字，即可通过require(modname)来加载模块。  
Lua是如何通过modname来载入.lua或.so的呢？

### 默认加载过程
1. package.loaded[modname]中存了模块的数据，有则直接返回
1. 顺序遍历package.searchers，获取loader
> 1. package.preload[modname]
> 2. Lua Loader, 通过package.searchpath搜索package.path
> 3. C Loader, 通过package.searchpath搜索package.cpath
> 4. All-In-One loader
1. 调用loader载入模块
1. 将载入结果保存至package.loaded[modname]并返回结果

可用lua模拟载入过程：
```
function findloader(modname)
    local loader = nil
    local ext = nil
    for _,s in ipairs(package.searchers) do
        loader,ext = s(modname)
        if type(loader)=="function" then return loader,ext end
    end
    error("findloader fail")
end

function myreq(modname)
    local m = package.loaded[modname]
    if m then return m end

    local loader,ext = findloader(modname)
    local ret = loader(modname, ext) or true
    package.loaded[modname] = ret
    return ret
end
```

### All-In-One Loader
例如:
```
require("a.b.c")
```
1. 查找a的C library(见过程2.3)
1. 检查它是否有luaopen_a_b_c函数

