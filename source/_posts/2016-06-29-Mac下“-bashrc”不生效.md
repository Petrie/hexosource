---
title: Mac下“.bashrc”不生效
category:
  - Linux
tags:
  - Linux
  - bashrc
date: 2016-06-29 10:54:10
description:
---


新建"~/.bash_profile"，写入下面代码：

``` bash
if [ "${BASH-no}" != "no" ]; then  
    [ -r ~/.bashrc ] && . ~/.bashrc  
fi
```
原因是执行.bashrc需要执行login shell，
而Mac启动时不会执行login shell，打开terminal也不会执行login shell
