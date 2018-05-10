---
title: php-zeal-long
tags:
---
###PHP5中整形的定义

#### 初始化变量

```c
		zval *tmp; \
		MAKE_STD_ZVAL(tmp); \
		ZVAL_LONG(tmp, _long); \
```

####宏定义源码
```c
#define ZVAL_LONG(z, l) {			\
		zval *__z = (z);			\
		Z_LVAL_P(__z) = l;			\
		Z_TYPE_P(__z) = IS_LONG;	\
	}
```

```c
#define Z_LVAL_P(zval_p)		Z_LVAL(*zval_p)
#define Z_LVAL(zval)			(zval).value.lval
```

```c
#define Z_TYPE_P(zval_p)	Z_TYPE(*zval_p)
#define Z_TYPE(zval)		(zval).type
```
###PHP7中整形的定义

#### 初始化变量

```
zval tmp; 
ZVAL_LONG(&tmp, _long); 
```

#### 宏定义源码

```
#define ZVAL_LONG(z, l) {				\
		zval *__z = (z);				\
		Z_LVAL_P(__z) = l;				\
		Z_TYPE_INFO_P(__z) = IS_LONG;	\
	}
```

```
#define Z_LVAL_P(zval_p)			Z_LVAL(*(zval_p))
#define Z_LVAL(zval)				(zval).value.lval
```

```
#define Z_TYPE_INFO_P(zval_p)		Z_TYPE_INFO(*(zval_p))
#define Z_TYPE_INFO(zval)			(zval).u1.type_info
```

