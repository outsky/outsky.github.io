---
layout: post
title:  修改win10中ubuntu子系统的vim字体
date:   2020-02-16 17:17:00 +0800
categories: os
tags: vim
---





修改wsl ubuntu子系统的vim字体为consolas



1. 打开注册表编辑器，选择：`HKEY_CURRENT_USER\Console\C:_Program Files_WindowsApps_CanonicalGroupLimited.UbuntuonWindows_xxxx.ubuntu.exe`
2. 新建DWORD（32位）值：`CodePage->(DWORD)FDE9(65001)`



参考链接：https://github.com/Microsoft/WSL/issues/757#issuecomment-359867729