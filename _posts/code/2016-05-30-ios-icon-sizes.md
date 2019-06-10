---
layout: post
title:  iOS各种图标尺寸标准
date:   2016-05-30 17:11:00 +0800
categories: code
tags: swift
---

|Asset|iPhone 6(s) Plus(@3x)|iPhone 5/6(s)(@2x)|iPhone 4s(@2x)|iPad (mini)(@2x)|iPad 2/iPad mini(@1x)|iPad Pro(@2x)|
| --- | --- | --- | --- | --- | --- | --- |
|App图标(必需)|180 x 180|120 x 120|120 x 120|152 x 152|	
76 x 76|167 x 167|
|AppStore上的App图标(必需)|1024 x 1024|1024 x 1024|1024 x 1024|1024 x 1024|1024 x 1024|1024 x 1024|
|启动文件/图片(必需)|使用启动文件|iPhone 6(s)使用启动文件;iPhone 5 640 x 1136|640 x 960|1536 x 2048(portrait);2048 x 1536(landscape)|768 x 1024(portrait);1024 x 768(landscape)|2048 x 2732(portrait);2732 x 2048(landscape)|
|Spotlight搜索结果图标(推荐)|180 x 180|iPhone 6(s) 120 x 120;iPhone 5 80 x 80|80 x 80|120 x 120|60 x 60|120 x 120|
|设置图标(推荐)|87 x 87|58 x 58|58 x 58|58 x 58|29 x 29|58 x 58|
|Toolbar/NavigationBar图标(可选)|66 x 66|44 x 44|44 x44|44 x 44|22 x 22|44 x 44|
|TabBar图标(可选)|75 x 75 (最大: 144 x 96)|50 x 50 (最大: 96 x 64)|50 x 50 (最大: 96 x 64)|50 x 50 (最大: 96 x 64)|25 x 25 (最大: 48 x 32)|50 x 50 (最大: 96 x 64)|

## 注意

- 所有的icon都建议使用PNG格式
- 标准的bit depth是24位

## 参考

[Icon and Image Sizes][1]

  [1]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/IconMatrix.html#//apple_ref/doc/uid/TP40006556-CH27-SW2
