---
title: Git多步悔棋
date: 2016-06-23 10:26:12
description: Git commit --amend命令的延伸
category: Git
tags: [Git]
---

上一篇讲到`Git commit --amend`单步悔棋的实现，是由`Git reset --soft`别名而来，这篇将讲解如何利用`Git reset --soft`多步悔棋
操作步骤
1.实用soft参数调用重置命令，回到指定的提交之前
``` bash
Git reset --soft HEAD^^
```
2.查看版本状态和提交日志，可以看出最后的两次提交被
``` bash
Git status
Git log -l
```
3.执行提交操作
``` bash
Git commit -m 'this is soft commit'
```
4.看看提交日志，“多步悔棋”操作成功
