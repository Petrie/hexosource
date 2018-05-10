---
title: PHP内存管理ZMM（四）－GDB调试php源码并手动调用ZMM相关函数
tags:
  - php
  - c
  - gdb
category: php
date: 2018-04-11 18:22:53
---


本章讲介绍gdb调试php，并手动调用ZMM中申请内存和查找大内存块的函数

- _zend_mm_alloc_int
- zend_mm_search_large_block

#### 为什么要手动调用函数

在阅读PHP ZMM源码的时候，有许多复杂的逻辑仅仅通过阅读源码很难理解，比如大内存large_free_buckets结构的构造。同构手动调用函数，可以方便的执行要申请的内存大小，从而测试构造large_free_buckets结构

#### 编译安装PHP

从github上下载php源码

编译到目录:~/php5.6-disabledebug，注意不要开启debug选项

#### GDB调试PHP
<!--more-->
```shell
 gdb --args ~/php5.6-disabledebug/bin/php test.php 
```

test.php可以是任意可执行php代码

执行后设置断点

```shell
(gdb) b _zend_mm_alloc_int
Breakpoint 1 at 0x6b2490: file /home/vagrant/php-src/Zend/zend_alloc.c, line 1881.
(gdb) b zend_mm_add_to_free_list
Breakpoint 2 at 0x6b12d0: file /home/vagrant/php-src/Zend/zend_alloc.c, line 734.
```

运行run命令执行test.php脚本

```shell
(gdb) run
Starting program: /home/vagrant/php5.6-disabledebug/bin/php test.php
[Thread debugging using libthread_db enabled]

Breakpoint 1, _zend_mm_alloc_int (heap=0xe88d90, size=8192) at /home/vagrant/php-src/Zend/zend_alloc.c:1881
1881	{
Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.209.el6_9.2.x86_64 libxml2-2.7.6-21.el6_8.1.x86_64 nss-softokn-freebl-3.14.3-23.3.el6_8.x86_64 zlib-1.2.3-29.el6.x86_64
(gdb)
```

#### 执行_zend_mm_alloc_int函数申请500b的内存

```shell
(gdb) call _zend_mm_alloc_int (0xe88d80, 500)

Breakpoint 1, _zend_mm_alloc_int (heap=0xe88d80, size=500) at /home/vagrant/php-src/Zend/zend_alloc.c:1881
1881	{
The program being debugged stopped while in a function called from GDB.
Evaluation of the expression containing the function
(_zend_mm_alloc_int) will be abandoned.
When the function is done executing, GDB will silently stop.
(gdb)
```

执行20行代码

```shell
(gdb) n 20

Program received signal SIGSEGV, Segmentation fault.
_zend_mm_alloc_int (heap=0xe88d80, size=500) at /home/vagrant/php-src/Zend/zend_alloc.c:1945
1945				if (UNEXPECTED(ZEND_MM_FREE_BLOCK_SIZE(p) == true_size)) {
(gdb)
```

查看heap各个字段的值

```shell
(gdb) p size
$1 = 500
(gdb) p true_size
$2 = 520
(gdb) p heap->size
$3 = 0
(gdb) p heap->real_size
$4 = 0
```

执行finish结束_zend_mm_alloc_int函数的执行

```shell
(gdb) finish
Run till exit from #0  _zend_mm_alloc_int (heap=0xe88d80, size=500) at /home/vagrant/php-src/Zend/zend_alloc.c:1945

Program terminated with signal SIGSEGV, Segmentation fault.
The program no longer exists.
```

再次查看heap各个字段的值

```shell
(gdb) p heap->size
$9 = 520
(gdb) p heap->real_size
$10 = 262144
```

#### 如何执行zend_mm_search_large_block

gdb调试过程中不知道什么原因。_zend_mm_alloc_int可以手动调用，但是zend_mm_search_large_block却不行。这里只能绕路调用。因为每次在调用_zend_mm_alloc_int必然会调用zend_mm_search_large_block。所以想调用

zend_mm_search_large_block，调用zend_mm_alloc_int即可。这里就不做演示了。