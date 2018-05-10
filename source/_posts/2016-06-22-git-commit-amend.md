---
layout: post  
title: "Git单步悔棋操作"  
description: "Git单步悔棋操作Git commit --amend 处女座的福音"  
category: Git
tags: [Git]  
---

本篇要点
``` shell
Git commit --amend
```
此命令简单来讲，就是Git的悔棋操作，我工作中最常见的场景
>1.写完一段代码 Git add，Git commit
2.发现一些注释不够完整，或者发现一些错别字
3.改了后再次 Git add，Git commit
4.完事

这个时候问题就来了，对于一个处女座来说，以上的同一件事情产生了两个commit。强迫症就犯了。辣么学会了Git commit的--amend参数你就可以解放了。Git commit --amend可以将本次提交和最后一次提交完美融合。是不是处女座的福音？！


manpage解释：
> --amend
Replace the tip of the current branch by creating a new commit. The recorded tree is prepared as usual(including the effect of the -i and -o options and explicit pathspec), and the message from the original commitis used as the starting point, instead of an empty message, when no other message is specified from the command line via options such as -m, -F, -c, etc. The new commit has the same parents and author as the current one (the --reset-author option can countermand this).

>It is a rough equivalent for:
>
>                     $ Git reset --soft HEAD^
>                     $ ... do something else to come up with the right tree ...
>                     $ Git commit -c ORIG_HEAD

>but can be used to amend a merge commit.

>You should understand the implications of rewriting history if you amend a commit that has already been published. (See the "RECOVERING FROM UPSTREAM REBASE" section in Git-rebase(1).)
