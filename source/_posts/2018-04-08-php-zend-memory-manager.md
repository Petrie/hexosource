---
title: PHP内存管理ZMM(一)－基本概念、数据结构和相关初始化函数
tags:
  - PHP
  - C
category: PHP
date: 2018-04-08 06:06:12
---


#### 基本概念

![ZZM架构图](http://flykobe.com/wp-content/uploads/2015/03/php-zend-memory-manager.jpg)

如上图所示，中间部分的zend memory manage由接口层、heap层、存储层(storage)组成。内存管理的主要逻辑在heap层中，后续主要讲解相关的数据结构和函数流程。

#### 基本数据结构

基于PHP-5.6

##### zend_mm_block_info

_zend_mm_block_info是ZMM内存管理中最小的数据单元。各字段含义见代码注释

```c
typedef struct _zend_mm_block_info {
#if ZEND_MM_COOKIES
   size_t _cookie;
#endif
   size_t _size;
   size_t _prev;
} zend_mm_block_info;
```

<!--more-->

##### zend_mm_block

zend_mm_block 对zend_mm_block_info做了一层封装，增加了一些条件编译字段，在相关编译选项关闭后，其和zend_mm_block_info是存储的内容是完全一致的

```c
typedef struct _zend_mm_block {
   zend_mm_block_info info;
#if ZEND_DEBUG
   unsigned int magic;
# ifdef ZTS
   THREAD_T thread_id;
# endif
   zend_mm_debug_info debug;
#elif ZEND_MM_HEAP_PROTECTION
   zend_mm_debug_info debug;
#endif
} zend_mm_block;
```

##### _zend_mm_heap

_zend_mm_heap是ZMM的核心数据结构，ZMM的基本所有数据都和这个结构体相关

```c
struct _zend_mm_heap {
   int                 use_zend_alloc;
   void               *(*_malloc)(size_t);
   void                (*_free)(void*);
   void               *(*_realloc)(void*, size_t);
   size_t              free_bitmap;
   size_t              large_free_bitmap;
   size_t              block_size;
   size_t              compact_size;
   zend_mm_segment    *segments_list;
   zend_mm_storage    *storage;
   size_t              real_size;
   size_t              real_peak;
   size_t              limit;
   size_t              size;
   size_t              peak;
   size_t              reserve_size;
   void               *reserve;
   int                 overflow;
   int                 internal;
#if ZEND_MM_CACHE
   unsigned int        cached;
   zend_mm_free_block *cache[ZEND_MM_NUM_BUCKETS];
#endif
   zend_mm_free_block *free_buckets[ZEND_MM_NUM_BUCKETS*2];
   zend_mm_free_block *large_free_buckets[ZEND_MM_NUM_BUCKETS];
   zend_mm_free_block *rest_buckets[2];
   int                 rest_count;
#if ZEND_MM_CACHE_STAT
   struct {
      int count;
      int max_count;
      int hit;
      int miss;
   } cache_stat[ZEND_MM_NUM_BUCKETS+1];
#endif
};
```

#### ZMM初始化

```flow
st=>start:   Start   |past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request
startzmm=>operation: start_memory_manager ()
at php-src/sapi/cli/php_cli.c:1362 |past
alloc_globals_ctor=>operation: alloc_globals_ctor () 
at /home/vagrant/php-src/Zend/zend_alloc.c:2742 |past
zend_mm_startup=>operation: zend_mm_startup () 
at /home/vagrant/php-src/Zend/zend_alloc.c:1221
为初始化管理器做一些准备，如获取设置seg_size(块大小)
zend_mm_startup_ex=>operation: zend_mm_startup_ex () 
at /home/vagrant/php-src/Zend/zend_alloc.c:1110
初始化storge和heap结构体中各个字段
zend_mm_init=>operation: zend_mm_init()
at /home/vagrant/php-src/Zend/zend_alloc.c:911
初始化heap中的四种内存块：缓存、小内存、大内存、剩余内存

st->startzmm->alloc_globals_ctor->zend_mm_startup->zend_mm_startup_ex->zend_mm_init->e
```
#### zend_mm_init小内存初始化流程详解

这部分鸟哥曾经写过相关文章，但我第一次看的时候基本看蒙，后来通过看源码和gdb调试才搞明白。其在heap中定义如下。

```c
struct _zend_mm_heap {
   ...
   zend_mm_free_block *free_buckets[ZEND_MM_NUM_BUCKETS*2];
   ...
};
```

free_buckets初始化代码如下，其中ZEND_MM_SMALL_FREE_BUCKET取出对应index的元素**p**，随后直接访问**next_free_block**,我在这里的主要疑问是，heap->free_buckets是一个指针数组，且没有用malloc初始化过，为什么可以直接访问其next_free_block属性呢？

```c
...
p = ZEND_MM_SMALL_FREE_BUCKET(heap, 0);
for (i = 0; i < ZEND_MM_NUM_BUCKETS; i++) {
   p->next_free_block = p;
   p->prev_free_block = p;
   p = (zend_mm_free_block*)((char*)p + sizeof(zend_mm_free_block*) * 2);
}
...
```

ZEND_MM_SMALL_FREE_BUCKET定义如下:

```c
#define ZEND_MM_SMALL_FREE_BUCKET(heap, index) \
   (zend_mm_free_block*) ((char*)&heap->free_buckets[index * 2] + \
      sizeof(zend_mm_free_block*) * 2 - \
      sizeof(zend_mm_small_free_block))
```

鸟哥的解释如下: [转自](http://www.laruence.com/2011/11/09/2277.html)

> 这是因为, PHP在这处使用了一个技巧, 用一个定长的数组来存储ZEND_MM_NUMBER_BUCKET个zend_mm_free_block, 如上图中红色框所示. 对于一个没有被使用的free_buckets的元素, 唯一有用的数据结构就是next_free_block和prev_free_block, 所以, 为了节省内存, PHP并没有分配ZEND_MM_NUMBER_BUCKET * sizeof(zend_mm_free_block)大小的内存, 而只是用了ZEND_MM_NUMBER_BUCKET * (sizeof(*next_free_block) + sizeof(*prev_free_block))大小的内存.

**其中对于heap->free_buckets这个定长数组的长度描述是不对的,不是ZEND_MM_NUMBER_BUCKET而是ZEND_MM_NUM_BUCKETS＊2。**

下面解释下ZEND_MM_SMALL_FREE_BUCKET，在64为系统中，假设index＝3，代入到ZEND_MM_SMALL_FREE_BUCKET中展开为

```c
//宏定义
ZEND_MM_SMALL_FREE_BUCKET(heap, index) \
   (zend_mm_free_block*) ((char*)&heap->free_buckets[index * 2] + \
      sizeof(zend_mm_free_block*) * 2 - \  // 8*2
      sizeof(zend_mm_small_free_block))	//32
//代入index并计算几个sizeof的值
  ZEND_MM_SMALL_FREE_BUCKET(heap, 3)\
	   (zend_mm_free_block*) ((char*)&heap->free_buckets[3 * 2] + \
     	8 * 2 - \
	    32)
```

计算后的最终值为 `(zend_mm_free_block*) ((char*)&heap->free_buckets[3 * 2] -16)`，这里假设`&heap->free_buckets[3 * 2]`的值为`0xe89028`,那么`(char*)&heap->free_buckets[3 * 2] -16`的值为`0xe89018`，所以宏ZEND_MM_SMALL_FREE_BUCKET的返回值就为 `(zend_mm_free_block*)0xe89018`，这个值也是`heap->free_buckets[2 * 2]`的地址，注意，这并不是巧合。宏ZEND_MM_SMALL_FREE_BUCKET返回后的代码逻辑如下：

```
...
p = ZEND_MM_SMALL_FREE_BUCKET(heap, 0);
for (i = 0; i < ZEND_MM_NUM_BUCKETS; i++) {
   p->next_free_block = p;
   p->prev_free_block = p;
   p = (zend_mm_free_block*)((char*)p + sizeof(zend_mm_free_block*) * 2);
}
...
```

可以看到这里只用到了`p`的`next_free_block`和`prev_free_block`，并没有对其他变量的访问。这里再回忆下`zend_mm_free_block`的定义。

```c
typedef struct _zend_mm_free_block {
	zend_mm_block_info info;
#if ZEND_DEBUG
	unsigned int magic;
# ifdef ZTS
	THREAD_T thread_id;
# endif
#endif
	struct _zend_mm_free_block *prev_free_block;
	struct _zend_mm_free_block *next_free_block;

	struct _zend_mm_free_block **parent;
	struct _zend_mm_free_block *child[2];
} zend_mm_free_block;

```

这里可以看到结构体`zend_mm_free_block`中`zend_mm_block_info`占据16个字节，和`heap->free_buckets[3 * 2] - heap->free_buckets[2 * 2] `相等。所以，访问`p->prev_free_block`和访问`heap->free_buckets[3*2]`是一致的，访问`p->next_free_block`和访问`heap->free_buckets[3*2+1]`是一致的。在访问`index＝3`的元素时，`heap->free_buckets[3 * 2] `到 `heap->free_buckets[2 * 2]`的16个字节可以看成是存储` index＝3`元素`zend_mm_block_info info;`但这个info字段zend memory manager 并不会也不应该被用到。对于小内存的处理就是利用这个技巧，节约了内存。

看明白了这些，再回头看小内存初始化的代码，就很简单了

#### 总结

本文大致介绍了ZMM较基本的偏概念的一些东西，对于小内存的初始化做了较为深入的讲解。这块开始理解起来会比较困难，建议配合gdb调试PHP查看heap->free_buckets相关内存地址配合理解。

#### 参考

- http://flykobe.com/index.php/2015/03/04/php%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/
- http://www.cnblogs.com/taek/p/4229869.html
- http://www.laruence.com/2011/11/09/2277.html