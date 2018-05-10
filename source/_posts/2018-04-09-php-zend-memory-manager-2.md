---
title: PHP内存管理ZMM（三）－内存分配函数emalloc
date: 2018-04-11 10:22:53
tags:
  - php
  - c
category: php
---



#### 主流程

emalloc是ZMM中heap层实现的函数，其内部调用_zend_mm_alloc_int函数。在_zend_mm_alloc_int中会依次在heap层的缓存区、小内存区、大内存区、剩余内存区寻找合适的内存。如果在这四个区域中都为查找到合适的内存，则调用malloc向内核申请，在向内核申请内存时，申请的大小必须是segment_size(256k)的整数倍，最小为256k。以下用流程图展示
<!--more-->
```flow
st=>start:   Start   |past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request
cache=>operation: 查找缓存 heap- >cache 
找到返回，未找到则继续|past
free_buckets=>operation: 查找小块内存heap- >free_buckets 
找到返回，未找到则继续|past
large_free_buckets=>operation: 查找大块内存heap- >large_free_buckets 
找到返回，未找到则继续|past
rest_buckets=>operation: 查找剩余内存 heap- >rest_buckets 
找到返回，未找到则继续|past
core_malloc=>operation: 在heap四个区域中均为找到合适大小的内存，
则向内核申请内存
add_rest=>operation: 将内核申请的内存分割为两部分，一部分函数返回，即本次申请的内存。
剩下的调用zend_mm_add_to_free_list，根据大小添加到heap的四个区域中
st->cache->free_buckets->large_free_buckets->rest_buckets->core_malloc->add_rest->e
```

#### 代码注解

代码如下：[引自](https://github.com/php/php-src/blob/9c1d686748cdb46e2f80a9bc800df0015fb709b1/Zend/zend_alloc.c#L1880)

```c

  static void *_zend_mm_alloc_int(zend_mm_heap *heap, size_t size ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC)
{
   zend_mm_free_block *best_fit;
  //计算true_size,对于计算逻辑和取值可以参考上一篇文章
   size_t true_size = ZEND_MM_TRUE_SIZE(size);
   size_t block_size;
   size_t remaining_size;
   size_t segment_size;
   zend_mm_segment *segment;
   int keep_rest = 0;
#ifdef ZEND_SIGNALS
   TSRMLS_FETCH();
#endif

   HANDLE_BLOCK_INTERRUPTIONS();

   if (EXPECTED((true_size))) {
     //根据true_size 计算 index的值
      size_t index = ZEND_MM_BUCKET_INDEX(true_size);
      size_t bitmap;

      if (UNEXPECTED(true_size < size)) {
         goto out_of_memory;
      }
#if ZEND_MM_CACHE //在缓存块中中查找
      if (EXPECTED(heap->cache[index] != NULL)) {
         /* Get block from cache */
#if ZEND_MM_CACHE_STAT
         heap->cache_stat[index].count--;
         heap->cache_stat[index].hit++;
#endif
         best_fit = heap->cache[index];
         heap->cache[index] = best_fit->prev_free_block;
         heap->cached -= true_size;
         ZEND_MM_CHECK_MAGIC(best_fit, MEM_BLOCK_CACHED);
         ZEND_MM_SET_DEBUG_INFO(best_fit, size, 1, 0);
         HANDLE_UNBLOCK_INTERRUPTIONS();
         return ZEND_MM_DATA_OF(best_fit);
      }
#if ZEND_MM_CACHE_STAT
      heap->cache_stat[index].miss++;
#endif
#endif
	 //在小内存块中查找
      bitmap = heap->free_bitmap >> index;
      if (bitmap) { //如果bitmap不等于0，说明存在大于或等于true_size的内存
         /* Found some "small" free block that can be used */
         index += zend_mm_low_bit(bitmap);//定位到true_size所在的index。举例说明：假设heap->free_bitmap＝0b100010, index=3,则bitmap＝(0b100),zend_mm_low_bit(bitmap)=3,最终index＝6
         best_fit = heap->free_buckets[index*2];//从free_buckets取出内存
#if ZEND_MM_CACHE_STAT
         heap->cache_stat[ZEND_MM_NUM_BUCKETS].hit++;
#endif
         goto zend_mm_finished_searching_for_block; //内存查找完成
      }
   }

#if ZEND_MM_CACHE_STAT
   heap->cache_stat[ZEND_MM_NUM_BUCKETS].miss++;
#endif
   //从大内存中查找，这部分逻辑，后续文章会写
   best_fit = zend_mm_search_large_block(heap, true_size);
   //如果大内存中也没有找到，且real_size已经超过php内存限制，则到剩余内存中查找
   if (!best_fit && heap->real_size >= heap->limit - heap->block_size) {
      zend_mm_free_block *p = heap->rest_buckets[0];
      size_t best_size = -1;
   //取头节点并循环
      while (p != ZEND_MM_REST_BUCKET(heap)) {
         if (UNEXPECTED(ZEND_MM_FREE_BLOCK_SIZE(p) == true_size)) {
            best_fit = p;
            goto zend_mm_finished_searching_for_block;
         } else if (ZEND_MM_FREE_BLOCK_SIZE(p) > true_size &&
                    ZEND_MM_FREE_BLOCK_SIZE(p) < best_size) {
            best_size = ZEND_MM_FREE_BLOCK_SIZE(p);
            best_fit = p;
         }
         p = p->prev_free_block;
      }
   }
   //以上四个区域均未找到
   if (!best_fit) {
      if (true_size > heap->block_size - (ZEND_MM_ALIGNED_SEGMENT_SIZE + ZEND_MM_ALIGNED_HEADER_SIZE)) {//true_size是否大于segment_size(block_size)，小于，则申请segment_size，大于则申请小于true_size的segment_size的整数位
         /* Make sure we add a memory block which is big enough,
            segment must have header "size" and trailer "guard" block */
         segment_size = true_size + ZEND_MM_ALIGNED_SEGMENT_SIZE + ZEND_MM_ALIGNED_HEADER_SIZE;
         segment_size = (segment_size + (heap->block_size-1)) & ~(heap->block_size-1);
         keep_rest = 1;//如果keep_rest为1，则在后续处理剩余空闲内存时，将其放入剩余内存区域。这里的考虑应该是，当前剩余空闲内存会比较大。而且也会情况也会比较少，所以单独处理放入剩余内存区域(rest_buckets)
      } else {
         segment_size = heap->block_size;
      }
	  //校验内存是否超出限制
      if (segment_size < true_size ||
          heap->real_size + segment_size > heap->limit) {
         /* Memory limit overflow */
#if ZEND_MM_CACHE
         zend_mm_free_cache(heap);
#endif
         HANDLE_UNBLOCK_INTERRUPTIONS();
#if ZEND_DEBUG
         zend_mm_safe_error(heap, "Allowed memory size of %ld bytes exhausted at %s:%d (tried to allocate %lu bytes)", heap->limit, __zend_filename, __zend_lineno, size);
#else
         zend_mm_safe_error(heap, "Allowed memory size of %ld bytes exhausted (tried to allocate %lu bytes)", heap->limit, size);
#endif
      }
	 //向系统申请内存
      segment = (zend_mm_segment *) ZEND_MM_STORAGE_ALLOC(segment_size);

      if (!segment) {
         /* Storage manager cannot allocate memory */
#if ZEND_MM_CACHE
         zend_mm_free_cache(heap);
#endif
out_of_memory:
         HANDLE_UNBLOCK_INTERRUPTIONS();
#if ZEND_DEBUG
         zend_mm_safe_error(heap, "Out of memory (allocated %ld) at %s:%d (tried to allocate %lu bytes)", heap->real_size, __zend_filename, __zend_lineno, size);
#else
         zend_mm_safe_error(heap, "Out of memory (allocated %ld) (tried to allocate %lu bytes)", heap->real_size, size);
#endif
         return NULL;
      }
      //更新heap统计字段
      heap->real_size += segment_size;
      if (heap->real_size > heap->real_peak) {
         heap->real_peak = heap->real_size;
      }

      segment->size = segment_size;
      segment->next_segment = heap->segments_list;
      heap->segments_list = segment;
      //格式化申请的内存 为zend_mm_free_block
      best_fit = (zend_mm_free_block *) ((char *) segment + ZEND_MM_ALIGNED_SEGMENT_SIZE);
      ZEND_MM_MARK_FIRST_BLOCK(best_fit);//初始化block中prev字段
	  //这里最后的ZEND_MM_ALIGNED_HEADER_SIZE是预留在最后的remaining_size使用的
      block_size = segment_size - ZEND_MM_ALIGNED_SEGMENT_SIZE - ZEND_MM_ALIGNED_HEADER_SIZE;
	  //初始化新segment的能存末位的 zend_mm_block
      ZEND_MM_LAST_BLOCK(ZEND_MM_BLOCK_AT(best_fit, block_size));

   } else {
zend_mm_finished_searching_for_block:
      /* remove from free list */
      ZEND_MM_CHECK_MAGIC(best_fit, MEM_BLOCK_FREED);
      ZEND_MM_CHECK_COOKIE(best_fit);
      ZEND_MM_CHECK_BLOCK_LINKAGE(best_fit);
      zend_mm_remove_from_free_list(heap, best_fit);

      block_size = ZEND_MM_FREE_BLOCK_SIZE(best_fit);
   }
   //计算使用后的剩余内存
   remaining_size = block_size - true_size;
   //如果剩余内存小于block的头部大小，则将其合并到true_size中
   if (remaining_size < ZEND_MM_ALIGNED_MIN_HEADER_SIZE) {
      true_size = block_size;
      ZEND_MM_BLOCK(best_fit, ZEND_MM_USED_BLOCK, true_size);
   } else {//否则，将剩余内存放入到heap中
      zend_mm_free_block *new_free_block;

      /* prepare new free block */ 
      ZEND_MM_BLOCK(best_fit, ZEND_MM_USED_BLOCK, true_size);
      new_free_block = (zend_mm_free_block *) ZEND_MM_BLOCK_AT(best_fit, true_size);//定位并格式化剩下的空闲内存
      ZEND_MM_BLOCK(new_free_block, ZEND_MM_FREE_BLOCK, remaining_size);

      /* add the new free block to the free list */
      if (EXPECTED(!keep_rest)) {//这里请看上面代码中对keep_rest赋值时的解释
         zend_mm_add_to_free_list(heap, new_free_block);
      } else {
         zend_mm_add_to_rest_list(heap, new_free_block);
      }
   }

   ZEND_MM_SET_DEBUG_INFO(best_fit, size, 1, 1);

   heap->size += true_size;
   if (heap->peak < heap->size) {
      heap->peak = heap->size;
   }

   HANDLE_UNBLOCK_INTERRUPTIONS();

   return ZEND_MM_DATA_OF(best_fit);
}
```
以上就是emalloc的所有逻辑了。

#### 四个内存区域的区分

基于64位系统

- 缓存区 cache

  todo

- 小内存区 free_buckets

  当申请的内存小于**520b**时候，放入小内存区。这个数字是怎么来的呢。

  逆推：当申请520b内存是，其true_size内存为520b+16(ZEND_MM_ALIGNED_HEADER_SIZE)=536，那么其实是需要536b大小的内存的。这其中还包含占用32b的`zend_mm_small_free_block`结构体`zend_mm_small_free_block`大小是固定的，所以不计入index计算，所以小内存区最大存入504b的内存，将504b按照计算index的方式逆推即右移3为则为最大的index值63。

  正推：又因为小内存数组的长度为64*2。这里要看成64组，所以小内存数组的index的最大值63＊2。

  综上，因为小内存数组最大index为63（64位系统），所以其所能存储的最大值

- 大内存区 large_free_buckets

  不满足**小内存区**条件和**剩余内存**条件的放入large_free_buckets

- 剩余内存区

  当heap中没有合适大小的内存向内核申请时，如果申请的大小大于heap->block_size。则当有额外剩下的内存时，放入到**剩余内存区rest_bucket**。举个例子假设heap->block_size＝256k。如果emalloc申请的大小小于256k，则剩下的内存存入**大内存区large_free_bucket**，否则 emalloc 申请的大小大于256k时，则剩下的内存存入**剩余内存区rest_buckets**中