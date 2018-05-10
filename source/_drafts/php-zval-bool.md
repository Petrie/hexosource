---
title: php-zval-bool
tags:
---
### PHP5.6 源码中bool的声明和初始化

#### 初始化bool

```c
zval *test_bool;
MAKE_STD_ZVAL(test_bool);
ZVAL_BOOL(test_bool, 1);
```

MAKE_STD_ZVAL在堆中分配内存并初始化zval和zval_gc_info的部分字段。初始化zval_gc_info的u.buffered字段为null、zval中的refcount\_\_gc＝1和is_ref\_\_gc=0字段。

ZVAL_BOOL为字段赋值和类型

#### 宏定义源码

```c
#define MAKE_STD_ZVAL(zv)				 \
	ALLOC_ZVAL(zv); \
	INIT_PZVAL(zv);
```
```c
#define ALLOC_ZVAL(z) 									\
	do {												\
		(z) = (zval*)emalloc(sizeof(zval_gc_info));		\
		GC_ZVAL_INIT(z);								\
	} while (0)
```
```c
#define GC_ZVAL_INIT(z) \
	((zval_gc_info*)(z))->u.buffered = NULL
```
```c
#define INIT_PZVAL(z)		\
	(z)->refcount__gc = 1;	\
	(z)->is_ref__gc = 0;
```
```c
#define ZVAL_BOOL(z, b) do {		\
		zval *__z = (z);			\
		Z_LVAL_P(__z) = ((b) != 0);	\
		Z_TYPE_P(__z) = IS_BOOL;	\
	} while (0)
```
```c
#define Z_LVAL_P(zval_p)		Z_LVAL(*zval_p)
#define Z_LVAL(zval)			(zval).value.lval
```
```c
#define Z_TYPE_P(zval_p)	Z_TYPE(*zval_p)
#define Z_TYPE(zval)		(zval).type
```

### PHP7源码中bool的声明和初始化

#### 初始化bool

```c
   zval result;
   ZVAL_BOOL(result, 0);
```

可以看出在PHP7初始化bool过程中，没有用到MAKE_STD_ZVAL初始化宏，为什么PHP5.6却有呢。这是因为PHP7对zval结构进行了重构。bool类型不再需要在堆中分配内存。

```c
...	
#define IS_FALSE					2
#define IS_TRUE						3
...
#define ZVAL_BOOL(z, b) do {			\
		Z_TYPE_INFO_P(z) =				\
			(b) ? IS_TRUE : IS_FALSE;	\
	} while (0)
```

```
#define Z_TYPE_INFO_P(zval_p)      Z_TYPE_INFO(*(zval_p))
#define Z_TYPE_INFO(zval)			(zval).u1.type_info
```

PHP5中bool类型的实现是通过zvalue_value.lval实现的，而PHP7中却没有类似的代码。这是因为PHP7中把bool类型拆分为两种不同的类型表示。分别为IS_FALSE，IS_TRUE。