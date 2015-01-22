---
layout: post
title: "清空空格"
date: 2014-11-30 21:57:07 +0800
comments: true
categories: elisp emacs
---
编辑代码时，使用Tab还是Whitespace缩进，保证没有右边的空格，不仅令自己看起来整洁，也会令代码增色。

Sublime Text提供一键选择Tab或WS，缩进几个字符，保存时自动删除右边的空格。然而，Emacs有更强大的功能。

## 使用Emacs自带的库：whitespace

whitespace十分强大，可以自定义配置如何显示空格字符，还可以一行代码将Tab转换为Space，一行代码删除所有右边的空格。

``` lisp
  (add-hook 'before-save-hook 'whitespace-cleanup nil t)
  (setq-local whitespace-style '(empty indentation::space
                                 space-befure-tab::space
                                 trailing
                                 whitespace-style::space))
```

使用add-hook，令文件在保存时触发清理空格，第三个参数nil，意味着先执行，如果没有更多函数加到这个hook。第四个参数意味着不是全局所有的buffer保存时都触发。因为只希望这个mode会这样。比如没人希望markdown-mode也触发清空whitespace。
