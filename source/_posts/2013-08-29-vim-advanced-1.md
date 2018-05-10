---
layout: post  
title: "Vim 进阶1 "  
description: "Vim 进阶1"  
category: vim
tags: [vim, vi, book]
---
##Vim 进阶[1]##

:g/pattern 寻找（移动）文件中最后一次出现pattern的地方。  
:g/pattern/p 寻找并显示文件中所有包含pattern的行  
:x 写入文件同时离开编辑器。只有文件被修改过时才会写入。  
:g/pattern/s/old/new/g这种语法让你搜索一个模式，在找到包含模式的某一行时，对另外一个字符做替换。  
vaw 选中当前单词  
vawp 选中当前单词并用寄存器中的覆盖  
