---
layout: post
title: "探秘Emacs 24.4"
date: 2014-10-19 14:05:43 +0800
comments: true
categories: emacs elisp
---
这篇文章翻译自[Mastering Emacs](http://www.masteringemacs.org)的[What's New In Emacs 24.4](http://www.masteringemacs.org/article/whats-new-in-emacs-24-4)。原作者是Mickey Petersen。
  
嗯，又一年的这时来到了。一个新的Emacs小版本就要问世了。按我的传统，我要详细解说新版本的特点。当然啦，在这里，“小版本”是一个相对的词：Emacs 24.4是全都是细微的改动和改变，但意义却不小。
  
在写这篇文章的时候，Emacs 24.4的新功能已经基本定下来了；不会有大更新，就算有小变动，也不会跟我写出的差很远。如果某个功能有bug，可能会暂时在新版本中拿掉，以后放回来。
  
如果你有什么不清楚的，你可以拉下来源文件，然后自己编译Emacs。
  
噢，还有一件事，我只是转换了NEWS文件，但没有重新整理它，有些内容可能放在了错误的分类下，我一点都没改变。
  
## Emacs 24.4 的新功能
  
## Emacs 24.4 安装时的改变
  
> Emacs 现在支持ACL。（译者注：ACL就是读写运行权限的管理）
> 如果编译时找到了合适的支持ACL的库，编译时会自动带着ACL。支持ACL的库有libacl等。如果不需要，编译时加上`--disable-acl`。
  
如果你的Linux支持ACL，那么这是好消息。Emacs对ACL支持到什么程度，我不清楚，但它可能还没到很成熟的程度，就像在dired设置ACL。
  
> Emacs 现在支持文件通知。
> 如果编译时找到了合适的支持的库，这个功能就会有。如果不想要这个功能，使用`--with-file-notification-no`。文件通知功能在这篇文章下面。
> **FIXME?** 这个功能是不是不支持Nextstep版本。（？）
  
很好，这个功能我会在本文很下面解释。
  
> 配置选项`--without-compress-info`已经规范了，改名成了`--without-compress-install`。它在安装时阻止压缩*_任何_文件*。
> 配置选项`--with-crt-dir`已经被移掉了。
> 它已经不需要了，因为crt*.o文件再不特别链接了。
> 传递到`--enable-locallisppath`的目录，再也不在安装时创建了。
> Emacs可以在编译时带着zlib支持。
> 如果有zlib（一般系统都有）这个库，就有了`zlib-decompress-region`这个功能，可以解压gzip-与zlip-格式的压缩数据。
  
Emacs支持了更多的数据压缩。没有提到`auto-compression-mode`是不是一开始就支持，我推测是这样。
  
> NS版的Emacs（OS X，GNUStep）编译时可以带着ImageMagick支持。需要pkg-config才能找到ImageMagick库。
> 对于OS X 10.5及以上，支持基于Core Text的字体后端。
> 对于GNUStep和OS X10.4，使用旧的字体后端。
> 如果要使用旧的字体后端，在命令行中输入
> % defauls write org.gnu.Emacs FontBackend ns
  
## Emacs 24.4 启动的变化
  
> 当初始化`load-path`的时候，环境变量EMACSLOADPATH中一个空的元素（无论是前，比如“:/foo”，后，比如“/foo:”，嵌入，比如“/foo::/bar”）会被替换成默认load-path（就是如果EMACSLOADPATH没有被设定时，默认使用的）。这令通过EMACSLOADPATH扩展load-path（以前，在设置EMACSLOADPATH的时候需要给出完整的load-path，包括默认的）变得容易。（在旧版本的Emacs中，一个空元素会被一个“.”代替，所以现在要直截了当地用“.”）。
  
如果你是那种使用命令行设置Emacs load path的－有可能根据不同需求你有好多个Emacs配置，这个功能很有用。这里有点说法，要小心点。
  
> -L选项，一般把它的参数拼接到load-path之前。如果参数第一个字母是`:`（或者是MS Windows的`;`，就是`path-separator`），就会被拼接到load-path之后。
  
如果使用-L选项，还有一个说法。
  
> 如果你使用了site-load.el或者site-init.el来自定义dumped的Emacs，dumping之后，这些文件对`load-path`做的改变都不在。如果不接受`load-path`的永久改变，`configure`时使用`--enable-locallisppath`选项。
> 选项`initial-buffer-choice`现在可以传递一个函数来设置初始buffer。
  
很好！现在你可以让Emacs拉取最新的dilbert漫画，然后在启动Emacs时显示。
  
## Emacs 24.4 的变化
  
> 新选项`gnutls-verify-error`，如果不是nil，意味着Emacs应该驳回GnuTLS认为不合格的SSL/TLS证书。（这个选项现在默认是nil，但是未来可能改变）。
> Emacs现在支持文本模式终端的菜单。
> 如果终端支持鼠标，点击菜单条，或者mode line或header line的敏感区域，相应的菜单会被打开。相似地，在敏感地区点击C-mouse-2或C-mouse-3会弹出相应的菜单。
> 如果文本终端不支持鼠标，你可以按F10，调用`menu-bar-open`，打开菜单栏上的第一个菜单。如果你想要过去的那种，就是F10调用`tmm-menubar`，自定义`tyy-menu-open-use-tmm`为一个非nil值。（无论这个值设不设，按M-\`会调用`tmm-menubar`）
  
这对于误用终端而不是窗口版的Emacs新手是个好消息。
  
> \*Messages\* buffer 现在是`messages-buffer-mode`，一个全新的带有只读的major mode。任何创建 \*Messages\* buffer的代码都应该叫`messages-buffer`函数。
> Emacs现在支持ACL （access control lists）。
> Emacs备份文件时保护文件的ACL信息。
  
如果凭ACL保护文件，这个功能真的很重要。
  
> 新函数`file-acl`和`set-file-acl`获取和设置文件的ACL信息。在GNU/Linux系统下，POSIX ACL接口通过libacl被利用。在MS-Windows系统下，NT安全接口被利用，来模仿POSIX ACL接口。
  
ACL也支持Windows，真的很不错。Windows ACL系统极度复杂，我好奇，为了保持接口统一和易用，做了多少让步？
  
> 加入了多显示器支持。
> 新函数`display-monitor-attributes-list`和`frame-monitor-attributes`可以用来获取每个显示器的信息和多显示器的配置。
  
真有趣，Emacs加入了多显示器支持。支持的程度是怎样呢？或许是把一个frame分开到不同的显示器中显示？－目前还不清楚。
  
> 函数`display-pixel-width`和`display-pixel-height`现在在不同平台表现的一样。它们返回显示器的像素宽度或高度，就像X11那样。要拿到物理显示器的信息，使用上面这两个函数。同样的还有`x-display-pixel-width`, `x-display-pixel-height`, `display-mm-width`, `display-mm-height`, `x-display-mm-width`, `x-display-mm-height`。
  
这潜在地对pos-tip这类package很有用。pos-tip建立一个很好的工具贴士，替代Emacs默认的，所以需要知道在哪里和怎么画自己。
  
> 在X和NS系统中，光标在闪烁10次（默认的）之后，停止闪烁。
> 你可以通过自定义`blink-cursor-blinks`来修改默认值。
  
我想知道，多少调查，可用性研究，协会会议和研究来确定10次闪烁而不是9或11。我个人在哪儿都关闭光标闪烁。
  
> 在keymap中，SPC（译者注：空格）向前滚动，S-SPC向后滚动。
> 这个影响View mode等。
  
有用，还有一个快捷键修饰对应的 alt+tab / shift+alt+tab 用来切换任务。
  
## 帮助变化
  
> 命令`apropos-variable`重命名成了`apropos-user-option`。`apropos-user-option`显示所有用户选项，而`apropos-variable`显示所有变量。当带着一个全局前缀参数调用时，这两个选项调换了行为。当`apropos-do-all`是非nil值的时候，它们的输出是一样的。
  
对于只想自定义显示选项的人，很有用，还有一个开关来重新打开就表现。
> `?`键现在形容绑定前缀，就像`C-h`。
  
太好了。我想不起哪些绑定使用`?`。
> `quail-help`命令已经删掉了。使用`C-h C-\`（`describe-input-method`）代替。
  
## Frame 和 window 的改变
  
> 两个新选项，`toggle-frame-fullscreen`和`toggle-frame-maximized`，分别绑定到<f11>和M-<f10>。
这可能是很多人的胜利。我曾写过一篇文章，关于如何在启动时最大化emacs。实际上，这个值得提起是因为在不同平台达到同样的效果真的很复杂。
  
还不太清楚它的键位。
  
> 新命令`frameset-to-register`已经绑定到了`C-x r f`，替代了`frame-configuration-to-register`。它提供同样的功能，还增加了恢复已删除的frame的功能。旧的命令一样存在，但是没有绑定。
  
好消息，如果你使用register来设置很多window和frame，它大概对package作者很有意义。
  
> 新的hook，`focus-in-hook`，`focus-out-hook`。
> 这是Emacs frame获得活着失去对焦时候运行的normal hook。
  
我期待人们怎么用这个功能！
  
> `split-window`现在是一个不交互的函数，也不是命令。
> 作为一个命令，它是特殊情况`C-x 2`（`split-window-below`），也很多余。使用Lisp重新实现了之后，它的交互性被错误地保持了。
  
一般没什么人直接调用这个命令。
  
> 新选项`scroll-bar-adjust-thumb-portion`。
> 只有X才有，这个选项允许通过滚动条控制over-scrolling。（比如，当buffer最尾可见时，拉下滚动条）
  
## Lisp 运行改动
  
> 在一个已经定义的defcustom中，`eval-defun`调用:set函数，如果有。
> 一个零前缀参数的`eval-last-sexp`（C-x C-e），`eval-expression`（M-:）he`eval-print-last-sexp`（C-j）插入一个列表。这个列表在长度和层次上没有限制（通过使用`print-length`和`print-level`的nil值），插入更多的数字格式（8进制，16进制和字符）。
  
这真的很方便。
  
> `write-region-inhibit-fsync`现在默认是t，在batch mode。
  
这可以加快批量调用，代价是如果没冲刷到磁盘，会有一点数据损失。（译者注：这里我不是很懂。）
  
> `cache-long-line-scans`改名成了`cache-long-scans`，因为它影响段落扫描的缓存结果。选项`set-mark-default-inactive`被删掉了。
> 这个未完成的功能不小心地在Emacs 23.1介绍，关闭transient mark mode作用一样。
> `comment-use-global-state`的默认值改为了t，这个变量被标成了过时。
  
## 新的用户选项
  
> `read-regexp-defaults-function`定义了一个函数读取正则表达式，被`rgrep`，`lgrep`，`occur`，`highlight-regexp`这种命令使用。你可以自定义这个，指定一个提供默认值的函数。从regexp上一个历史元素，或是光标所在的symbol都可以。
  
这对package作者是好消息。当你提供prompt的时候，确定指向一个共享的历史变量。这个很节约时间，会令Emacs hacker 覆写输入正则表达式的方式。
  
> `load-prefer-newer`，影响`load`函数如何选择文件去加载。如果是non-nil，那么.el和.elc同时存在，加载新的，nil意味着总是加载.elc文件。
  
（译者注：这对debug和hacking有点用，免去了编译的一步。）
  
## Emacs 24.4 编辑的变化
  
## 缩进变化
  
（译者注：接下来我可能会继续翻译完它）
