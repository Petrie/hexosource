---
title: PHP内存管理ZMM（五）－大内存区large_free_bucket的存入
tags:
  - php
  - c
category: php
date: 2018-04-12 18:22:53
---


之前的章节中介绍过large_free_bucket的存入条件。这一篇将介绍large_free_bucket的主要结构包括其中的链表结构和树结构和存入取出流程。本章讲通过图示大内存区域内存分部情况。

### 什么时候会向large_free_bucket存入内存块

这里在复习下存入large_free_bucket流程。在调用emalloc申请能存，且在当前heap中没有找到合适内存块，emalloc函数会调用malloc向内核申请内存。向内核申请每次只能申请 heap->block_size倍数大小内存。所以内核申请到的  heap->block_size倍数 大小的内存并不会全部返回到emalloc调用者，而是有剩余。

举个例子，假设heap->block_size=256k，如果emalloc调用者申请大小为250k时，emalloc内部会向内核申请256k大小的内存。所以会有6k的内存剩余，这6k的内存将会存入到**大内存区large_free_bucket**。

另外一个例子，假设heap->block_size=256k，如果emalloc调用者申请大小为260k（大于256k）时，emalloc会申请256*2=512k大小的内存，此时会有512k－260k＝252k的内存剩余。此时剩下的252k内存会存入到**剩余内存区rest_bucket**。

### 手动构建large_free_bucket结构

#### large_free_bucket结构举例

现在假设有以下内存块在large_free_bucket，分别为：

```c
大小			二进制					alloc_size
	
600			0000 0010 0101 1000		262144-48-600=261496
704			0000 0010 1100 0000		262144-48-704=261392
800			0000 0011 0010 0000		262144-48-800=261296
904			0000 0011 1000 1000		262144-48-904=261192
1000		0000 0011 1110 1000		262144-48-1000=261096
600			0000 0010 0101 1000		262144-48-600=261496
704			0000 0010 1100 0000		262144-48-704=261392
```

上面第一列中是准备存入heap中的内存大小，第二列为其二进制。第三列为_zend_mm_alloc_int传入的内存大小。


<!--more-->
之前提过用gdb手动调用_zend_mm_alloc_int的方法，详情看一下前面的文章“[PHP内存管理ZMM（四）－GDB调试php源码并手动调用ZMM相关函数](http://petrie.github.io/2018/04/11/php-zend-gdb-call/)”。

#### 存入第一个大小为600b的内存

手动调用

```shell
call _zend_mm_alloc_int (heap=0xe88d90, size=261496)
```

申请大小261496b（第一行最后一行的数值），ZMM会向内核申请262144(256k)的内存，去掉部分结构体占用的内存48b。则剩余600b大小的内存，这600b的内存讲放入**大内存区域**中。

用gdb查看heap->large_free_buckets当前的情况

```shell
(gdb) p heap->large_free_bitmap
$26 = 512
(gdb) p heap->large_free_buckets
$10 = {0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x7ffff7fecda8, 0x0 <repeats 54 times>}
(gdb) p &heap->large_free_buckets[9]
$16 = (zend_mm_free_block **) 0xe89470
(gdb) p *heap->large_free_buckets[9]
$15 = {info = {_size = 600, _prev = 261513}, prev_free_block = 0x7ffff7fabda8, next_free_block = 0x7ffff7fabda8, parent = 0xe89470, child = {0x0, 0x0}}
```

图示如下：

![](http://on-img.com/chart_image/5acf199ee4b0518eaca92c2f.png)

如图所示，插入的第一个元素zend_mm_free_block[600]（用zend_mm_free_block[600]表示大小为600的zend_mm_free_block内存结构），其prev_free_block和next_free_block均指向其本身，其parent为&heap->large_free_buckets[9]，heap->large_free_buckets[9]则指向zend_mm_free_block[600]。

相关代码:[引自](https://github.com/php/php-src/blob/9c1d686748cdb46e2f80a9bc800df0015fb709b1/Zend/zend_alloc.c#L739)

```c
		p = &heap->large_free_buckets[index];
		mm_block->child[0] = mm_block->child[1] = NULL;
		if (!*p) {
			*p = mm_block;
			mm_block->parent = p;
			mm_block->prev_free_block = mm_block->next_free_block = mm_block;
			heap->large_free_bitmap |= (ZEND_MM_LONG_CONST(1) << index);
		} else {
```

#### 存入大小为600b的内存

手动调用

```shell
call _zend_mm_alloc_int (heap=0xe88d90, size=261392)
```

用gdb查看heap->large_free_buckets当前的情况

```shell
(gdb) p heap->large_free_bitmap
$27 = 512
(gdb) p heap->large_free_buckets
$28 = {0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x7ffff7fecda8, 0x0 <repeats 54 times>}
(gdb) p *heap->large_free_buckets[9]
$29 = {info = {_size = 600, _prev = 261513}, prev_free_block = 0x7ffff7fabda8, next_free_block = 0x7ffff7fabda8, parent = 0xe89470, child = {0x7ffff7f6ad40, 0x0}}
(gdb) p *heap->large_free_buckets[9]->child[0]
$30 = {info = {_size = 704, _prev = 261409}, prev_free_block = 0x7ffff7f6ad40, next_free_block = 0x7ffff7f6ad40, parent = 0x7ffff7fecdd0, child = {0x0, 0x0}}
```

图示如下：



![](http://on-img.com/chart_image/5acf1923e4b02dfcf99d4a6b.png)

这里要注意的地方是zend_mm_free_block[704]的parent字段是一个指针的指针，其指向zend_mm_free_block[600]的child[0]字段。

相关代码：[](https://github.com/php/php-src/blob/9c1d686748cdb46e2f80a9bc800df0015fb709b1/Zend/zend_alloc.c#L746)

```c
} else {
			size_t m;

			for (m = size << (ZEND_MM_NUM_BUCKETS - index); ; m <<= 1) {
				zend_mm_free_block *prev = *p;

				if (ZEND_MM_FREE_BLOCK_SIZE(prev) != size) {
					p = &prev->child[(m >> (ZEND_MM_NUM_BUCKETS-1)) & 1];
					if (!*p) {
						*p = mm_block;
						mm_block->parent = p;
						mm_block->prev_free_block = mm_block->next_free_block = mm_block;
						break;
					}
				} else {
					...
				}
			}
		}
```



#### 存入大小为800b、904b、1000b的内存

依次手动调用

```shell
call _zend_mm_alloc_int (heap=0xe88d90, size=261296)
call _zend_mm_alloc_int (heap=0xe88d90, size=261192)
call _zend_mm_alloc_int (heap=0xe88d90, size=261096)
```

用gdb查看heap->large_free_buckets当前的情况

```shell
(gdb) p heap->large_free_buckets[9]
$71 = (zend_mm_free_block *) 0x7ffff7fecda8
(gdb) p *heap->large_free_buckets[9]
$72 = {info = {_size = 600, _prev = 261513}, prev_free_block = 0x7ffff7fecda8, next_free_block = 0x7ffff7fecda8, parent = 0xe89470, child = {0x7ffff7fabd40, 0x7ffff7f6ace0}}
(gdb) p *heap->large_free_buckets[9]->child[0]
$73 = {info = {_size = 704, _prev = 261409}, prev_free_block = 0x7ffff7fabd40, next_free_block = 0x7ffff7fabd40, parent = 0x7ffff7fecdd0, child = {0x0, 0x0}}
(gdb) p *heap->large_free_buckets[9]->child[1]
$74 = {info = {_size = 800, _prev = 261313}, prev_free_block = 0x7ffff7f6ace0, next_free_block = 0x7ffff7f6ace0, parent = 0x7ffff7fecdd8, child = {0x0, 0x7ffff7f29c78}}
(gdb) p *heap->large_free_buckets[9]->child[1]->child[1]
$75 = {info = {_size = 904, _prev = 261209}, prev_free_block = 0x7ffff7f29c78, next_free_block = 0x7ffff7f29c78, parent = 0x7ffff7f6ad10, child = {0x0, 0x7ffff7ee8c18}}
(gdb) p *heap->large_free_buckets[9]->child[1]->child[1]->child[1]
$76 = {info = {_size = 1000, _prev = 261113}, prev_free_block = 0x7ffff7ee8c18, next_free_block = 0x7ffff7ee8c18, parent = 0x7ffff7f29ca8, child = {0x0, 0x0}}
```

图示如下：

![](http://on-img.com/chart_image/5acf1a4ce4b0f5fa24d00514.png)

#### 存入大小为600b、704b的内存

依次手动调用

```shell
call _zend_mm_alloc_int (heap=0xe88d90, size=261496)
call _zend_mm_alloc_int (heap=0xe88d90, size=261392)
```

用gdb查看heap->large_free_buckets当前的情况

```shell
(gdb) p *heap->large_free_buckets[9]->child[0]->next_free_block
$101 = {info = {_size = 704, _prev = 261409}, prev_free_block = 0x7ffff7fabd40, next_free_block = 0x7ffff7fabd40, parent = 0x0, child = {0x0, 0x0}}
(gdb) p *heap->large_free_buckets[9]->next_free_block
$102 = {info = {_size = 600, _prev = 261513}, prev_free_block = 0x7ffff7fecda8, next_free_block = 0x7ffff7fecda8, parent = 0x0, child = {0x0, 0x0}}
```

图示如下：

![](http://on-img.com/chart_image/5acf2732e4b0518eaca98596.png)

上图包括了 大内存区域 的两种数据结构：树结构和双向链表结构。

相关代码：[引用](https://github.com/php/php-src/blob/9c1d686748cdb46e2f80a9bc800df0015fb709b1/Zend/zend_alloc.c#L760)

```c
		} else {
			size_t m;

			for (m = size << (ZEND_MM_NUM_BUCKETS - index); ; m <<= 1) {
				zend_mm_free_block *prev = *p;

				if (ZEND_MM_FREE_BLOCK_SIZE(prev) != size) {
					p = &prev->child[(m >> (ZEND_MM_NUM_BUCKETS-1)) & 1];
					...
				} else {
					zend_mm_free_block *next = prev->next_free_block;

					prev->next_free_block = next->prev_free_block = mm_block;
					mm_block->next_free_block = next;
					mm_block->prev_free_block = prev;
					mm_block->parent = NULL;
					break;
				}
			}
		}
```



### END

本章内容较多，可能会比较难理解，通过gdb调试有助于更好的理解。另外前几篇关于ZMM的文章也有助于对本章的理解。