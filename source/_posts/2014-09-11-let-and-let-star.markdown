---
layout: post
title: "let and let*"
date: 2014-09-11 13:43:28 +0800
comments: true
categories: elisp
---
### 什么是变量绑定？
先看一段代码。
``` lisp
(let* ((this-dir (file-name-directory (or load-file-name buffer-file-name)))
       (util-dir (file-name-as-directory (expand-file-name "util" this-dir)))
       (inf-ruby-dir (file-name-as-directory (expand-file-name "inf-ruby" util-dir)))
       (jump-dir (file-name-as-directory (expand-file-name "jump" util-dir))))
  (dolist (dir (list util-dir inf-ruby-dir jump-dir))
    (when (file-exists-p dir)
      (add-to-list 'load-path dir))))
```
它有这样的格式：(let VARLIST BODY...)，说得容易理解就是这样：
(let ((varname value) (varname value)...)
  (body1)
  (body2)...)
这些被绑定的变量是在这个let之内的，一旦let中的语句执行完，就不见了，不会造成变量名污染。
### let 和 let* 的异同。
let和let\*在elisp中，都是局部变量绑定。不同的是，let是平行绑定（parallel binding），let\*是顺序绑定。
  
例子如下：
``` lisp
(setf x 'outside)
(let ((x 'inside) (y x)
  (list x y))
  ;输出结果为(INSIDE OUTSIDE)
(let* ((x 'inside) (y x)
  (list x y))
  ;输出结果为(INSIDE INSIDE)
```
