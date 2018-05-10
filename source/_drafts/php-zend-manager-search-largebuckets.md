---
title: PHP内存管理ZMM（六）－大内存区large_free_bucket的查找
tags:
  - php
  - c
category: php
date: 2018-04-14 18:22:53
---

上一章介绍了 **大内存区large_free_bucket**的存储，本章将介绍如何在**大内存区large_free_bucket**中找到合适大小的内存。通过代码注释和图示举例的方式更好的理解large_free_bucket的查询逻辑
<!--more-->
#### 代码注释

代码如下：[引用 ](https://github.com/php/php-src/blob/9c1d686748cdb46e2f80a9bc800df0015fb709b1/Zend/zend_alloc.c#L1807)

```c
static zend_mm_free_block *zend_mm_search_large_block(zend_mm_heap *heap, size_t true_size)
{
	zend_mm_free_block *best_fit;
	size_t index = ZEND_MM_LARGE_BUCKET_INDEX(true_size);
	size_t bitmap = heap->large_free_bitmap >> index;
	zend_mm_free_block *p;

	if (bitmap == 0) {
		return NULL;
	}

	if (UNEXPECTED((bitmap & 1) != 0)) { //代码执行到这里bitmap肯定不为0,因为上面已经判断过bitmap的值,所有这个if条件始终为true.所以这里的判断是多余的
		/* Search for best "large" free block */
		zend_mm_free_block *rst = NULL;
		size_t m;
		size_t best_size = -1;

		best_fit = NULL;
		p = heap->large_free_buckets[index];//停止foreach的条件有两个是,第一个是所校验的bit位为0,且其左子数为空.第二个是所校验bit位为1,且其右子数为空
		for (m = true_size << (ZEND_MM_NUM_BUCKETS - index); ; m <<= 1) { //假设true_size=704(0010 1100 0000) 则index=9, m= true_size<<(64-9)=(0110 0000 ....)
			if (UNEXPECTED(ZEND_MM_FREE_BLOCK_SIZE(p) == true_size)) {
				return p->next_free_block;
			} else if (ZEND_MM_FREE_BLOCK_SIZE(p) >= true_size &&
			           ZEND_MM_FREE_BLOCK_SIZE(p) < best_size) {
				best_size = ZEND_MM_FREE_BLOCK_SIZE(p);
				best_fit = p;
			}//如果为true,则当前节点的左右子数都有可能比 true_size大,所以都要考虑,这里暂存右孩子,用左孩子继续循环
			if ((m & (ZEND_MM_LONG_CONST(1) << (ZEND_MM_NUM_BUCKETS-1))) == 0) { //m & (0000 ... 0001 << 64 -1) = 1000 0.... 0000
				if (p->child[1]) { //如果不为0
					rst = p->child[1];//到这里,rst所指向的 p->child[1],为了在p->child[0]大于true_size时在后续循环line1850进行继续校验p->child[1]
				}
				if (p->child[0]) {
					p = p->child[0];
				} else {
					break;
				}//如果为false,则当前节点只有其右子数会有可能大于 true_size,所以只需要考虑右子数
			} else if (p->child[1]) {
				p = p->child[1];
			} else {//此时左右子树都为空,说明已遍历到树的叶子节点
				break;
			}
		}
        //for条件解析,如果左子树不等于空,取右子数,如果左子树等于空,取左子树.简而言之.for循环下去的条件是 p的左右子树都不为空.并取右子树继续循环
		for (p = rst; p; p = p->child[p->child[0] != NULL]) {
			if (UNEXPECTED(ZEND_MM_FREE_BLOCK_SIZE(p) == true_size)) {
				return p->next_free_block;
			} else if (ZEND_MM_FREE_BLOCK_SIZE(p) > true_size &&
			           ZEND_MM_FREE_BLOCK_SIZE(p) < best_size) {
				best_size = ZEND_MM_FREE_BLOCK_SIZE(p);
				best_fit = p;
			}
		}

		if (best_fit) {
			return best_fit->next_free_block;
		}
		bitmap = bitmap >> 1;
		if (!bitmap) {
			return NULL;
		}
		index++;
	}

	/* Search for smallest "large" free block */ //函数执行到此,说明没有在heap->large_free_buckets[index]下找到合适的内存,这里的index + zend_mm_low_bit(bitmap)等于true_size中下一个bit不为0的index.假设large_bitmap为(1000100100) 那么之前的逻辑已经校验过倒数第三位的bit(为1),此部分逻辑校验倒数第六位的index. 因为这个index的内存都要比true_size大,所以只需要遍历其左子数,找出当前index中最小的
	best_fit = p = heap->large_free_buckets[index + zend_mm_low_bit(bitmap)];
	while ((p = p->child[p->child[0] != NULL])) {
		if (ZEND_MM_FREE_BLOCK_SIZE(p) < ZEND_MM_FREE_BLOCK_SIZE(best_fit)) {
			best_fit = p;
		}
	}
	return best_fit->next_free_block;
}
```

#### 示例构造

```shell
大小			二进制			alloc_size
769			1100000000  	262144-48-768=261328		
832			1101000000  	262144-48-832=261264		
992			1111100000  	262144-48-992=261104		
1008		1111110000  	262144-48-1008=261088		
968			1111001000  	262144-48-968=261128		
984			1111011000		262144-48-984＝261112
976			1111010000		262144-48-976=261120
2040		11111111000		262144-48-2040=260056
```

依次插入示例大小的内存到**大内存区**，具体方法和过程可以查看上一篇博客。最终在内存中结构如下：

![](http://on-img.com/chart_image/5ad0a999e4b04721d6171a06.png)

#### 查询大小为936的内存

```flow
st=>start:   Start   |past
e=>end: End
calc_index=>operation: 计算true_size内存所存储index，这里为9
calc_bitmap=>operation: 将large_free_bitmap
右移index位后判断是否为0。
为0则说明large_free_buckets
不存在true_size大小的内存。
这里true_size＝936。所以最终结果为1。
init_p=>operation: 初始话p指向
large_free_buckets[9]的第一个元素
init_m=>operation: 初始化init_m变量为 
true_size左移 (ZEND_MM_NUM_BUCKETS - index) 位
这里为936左移(64-9)位=1101 0100 0000 ... 0000b(共64位)。
compare_size=>operation: 判断p所指向元素的内存大小
是否满足true_size或best_size
first_m=>condition: 判断m中二进制最
高位的值。此值即true_size
二进制最高位1的后一位的值。
first_m0=>condition: 如果为0，则以p为父节点的
左右子树都可能，
优先考虑左孩子，再考虑右孩子
存在满足true_size内存块。
所以都要处理
first_m1=>operation: 如果为1，则只需要考虑p的右节点。
first_m01=>condition: 如果右孩子不为空则，
最将其暂存在rst指针中。
如果左孩子不为空。
则继续循环。如果左孩子
为空则停止循环
move_m=>operation: 将m左移一位。
rst=>operation: 处理rst,rst处理逻辑很简单，
节点的左右子树都不为空.
并取右子树继续循环。
best_fit=>operation: 以上如果存在best_fit则返回，
如果不存在继续
last_best_fit=>operation: 在最优的index(9)上未能找到best_fit。
只能在index更大的节点(index=10)上找。
通过判断bitmap，直接返回根元素即可。
st->calc_index->calc_bitmap->init_p->init_m->compare_size->first_m
first_m(yes,right)->first_m0
first_m(no)->first_m1
first_m0(yes)->first_m01(yes,left)->move_m(left)->fisrt_m
first_m1->move_m(left)->first_m
first_m01(no,down)->rst->best_fit->last_best_fit->e
```

