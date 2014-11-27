---
layout: post
title: "团队协作常用的git技巧"
date: 2014-11-27 20:59:21 +0800
comments: true
categories: git
---
在每日团队协作工作中，版本管理控制不可缺少。可是没有技巧地直接去用，就会产生杂乱的分支，以及排错的艰难。这篇文章介绍几个简单却常用的git技巧。

## fetch和rebase
在协作中，最常见的情况就是有太多合并队友主分支，服务器上的主分支被不同队友的主分支切换来切换去。如何减少合并分支并使git历史扁平化呢？

答案是rebase.

``` bash
git fetch
git rebase
```

通常我们拉取代码时使用的``git pull``，其实相当于做这两件事：

``` bash
git fetch
git merge
```

fetch是将remote的代码取下来，放在本地。merge会合并fetch取下来的记录到当前分支中。如果两个分支不能够“快进”，就只有产生合并提交。

rebase是将本地的提交重新基于拉下来的提交。本地提交A会合并另外分支的提交并变成A'，来达到扁平的提交记录。

## branch

当要为项目加一个功能的时候，在主分支做事并不是很好的习惯。使用branch可以创建分支，然后转到那个分支。如下：

``` bash
git branch feature_2048
git checkout feature_2048
```

这两部操作可以简化为：

``` bash
git checkout -b feature_2048
```

在分支提交，可以方便地在分支间切换，未完成的工作也不会影响到主分支。

当一个功能开发完成之后，可以切换到主分支，然后合并这个功能。如下：

``` bash
git checkout master
git merge feature_2048
```

如果将要合并时已经有队友推送了提交，可以在主分支拉取（pull）队友的提交，然后合并这个分支，并删除这个分支，推送到代码仓库：

``` bash
# after some commit in feature_2048
git checkout master
git pull
git merge feature_2048
git branch -d feature_2048
git push
```
在做大功能的时候，这个方式好于前面fetch和rebase的方法。

## stash

有时要做一个功能，或者一个修复，一不小心在主分支开始写了代码。这时领导跑过来要求修复一个紧急漏洞。应该怎么办呢？

stash可以把手头的工作暂时放到一边，回到HEAD初始的状态。

``` bash
# hack, hack, hack
# then,
# leader comes in
git stash
# fix the emergency bug
# run corresponding test
git commit
git push
git stash pop
# haha, my work comes back, whoo!
```

将这个分支没提交的工作切换到另外的分支：

这种情况适合于不小心在主分支写了觉得不该在主分支写的代码。

``` bash
# write some code
git stash
git checkout -b issue_32
git stash pop
```

所有曾在主分支做的工作都顺利转移到了issue_32分支。
虽然多数情况下这跟直接切换分支几乎没有区别，很多人经验证明切换分支的时候不建议保留不干净状态。

## cherry pick

在功能分支做了多次提交，可是其中的一个功能并不需要，是倒数第四个提交带来的，怎样去掉呢？

``` bash
# On branch feature_2030
git checkout master
git cherry-pick 'commit hash'
```

这样就能挑选着选择合并什么不合并什么。

## log

分支多的时候，要弄清本地和远程不同分支的关系。
直接敲``git log``是不够的。

参数 ``--all``，不只查看当前分支的记录。

参数 ``--graph``，左边会带着分支图。

参数 ``--abbrev-commit``，会使提交hash变的简短。

参数 ``--decorate``，会写出HEAD以及分支的位置。

有一个方便快捷的做法是，在``~/.gitconfig``中加入如下配置：
``` text
l = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n'' %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all

```
关于--format参数的配置，可以参考[官方文档](http://git-scm.com/docs/git-log)。
这样``git l``就可以查看更美观的提交记录。

## 总结

git是非常强大的版本控制系统，本文只提及了少许，更多的经验要在实践中发现和总结。能够利用好git的个人和团队，在工作中效率更高。
