---
title: php-zval-double
tags:
---

### PHP5.6中浮点型的定义

#### 初始化变量

```c
zval *http_version;
MAKE_STD_ZVAL(http_version);
ZVAL_DOUBLE(http_version, 1.1);
```

同整形的初始话过程类似，浮点型的初始化也调用了MAKE_STD_ZVAL。其为变量申请内存。同时初始化zval_gc_info的u.buffered字段为null、zval中的refcount\_\_gc和is_ref\_\_gc字段。

#### 宏定义

```
#define ZVAL_DOUBLE(z, d) {			\
		zval *__z = (z);			\
		Z_DVAL_P(__z) = d;			\
		Z_TYPE_P(__z) = IS_DOUBLE;	\
	}
```

```c
#define Z_DVAL_P(zval_p)		Z_DVAL(*zval_p)
#define Z_DVAL(zval)			(zval).value.dval
```

```
#define Z_TYPE_P(zval_p)	Z_TYPE(*zval_p)
#define Z_TYPE(zval)		(zval).type
```

ZVAL_DOUBLE给zval.zvalue_zvalue中的dval字段赋值，给zval中的type字段赋值IS_DOUBLE

### PHP7中浮点型的定义

#### 初始化变量

```
	zval tmp;
	ZVAL_DOUBLE(&tmp, 1.1);
```

与bool和整形在PHP7中初始化类似，浮点型的初始化也不需要调用MAKE_STD_ZVAL，因为其不再堆中分配。

#### 宏定义

```c
#define ZVAL_DOUBLE(z, d) {				\
		zval *__z = (z);				\
		Z_DVAL_P(__z) = d;				\
		Z_TYPE_INFO_P(__z) = IS_DOUBLE;	\
	}
```

```
#define Z_DVAL_P(zval_p)			Z_DVAL(*(zval_p))
#define Z_DVAL(zval)				(zval).value.dval
```

```c
#define Z_TYPE_INFO_P(zval_p)		Z_TYPE_INFO(*(zval_p))
#define Z_TYPE_INFO(zval)			(zval).u1.type_info
```

ZVAL_DOUBLE给zval.dval赋值，给(zval).u1.type_info赋值IS_DOUBLE