---
title: 深入浅出'Git commit --amend'
description: "深入浅出Git单步悔棋操作"
category: Git
tags: [Git]
---

Git能够提供悔棋的奥秘在于Git的reset命令。`Git commit --amend` 可以理解为`Git reset` 的alias。
``` bash
Git commit --amend ‘this is amend commit’
```
等价于
``` bash
Git reset --soft HEAD^
```
