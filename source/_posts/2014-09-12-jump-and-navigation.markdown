---
layout: post
title: "elisp的跳转和导航"
date: 2014-09-12 15:38:29 +0800
comments: true
categories: [emacs elisp]
---
### 直接跳转
在光标下，M-x find-function
可以将这个绑定到一个快捷的按键。

### imenu
多数mode都支持imenu来跳转到指定函数定义处。
M-x imenu

### 用occur来概览
M-x occur
它会生成一个新的buffer，给出所有匹配。在elisp模式下，用occur搜索defun，会得到一个函数定义的概览。

### 使用hs-minor-mode来折叠代码
使用hs-minor-mode可以折叠和展开。
