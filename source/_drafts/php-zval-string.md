---
title: php-zval-string
tags:
---

### PHP5.6中字符串的定义

#### 初始化变量

```
zval	*zfuncname;
MAKE_STD_ZVAL(zfuncname);
ZVAL_STRING(zfuncname, "getTimestamp", 1);
```

#### 宏定义

```c
#define ZVAL_STRING(z, s, duplicate) do {	\
		const char *__s=(s);				\
		zval *__z = (z);					\
		Z_STRLEN_P(__z) = strlen(__s);		\
		if (UNEXPECTED(Z_STRLEN_P(__z) < 0)) { \
			zend_error(E_ERROR, "String size overflow"); \
		}									\
		Z_STRVAL_P(__z) = (duplicate?estrndup(__s, Z_STRLEN_P(__z)):(char*)__s);\
		Z_TYPE_P(__z) = IS_STRING;			\
	} while (0)
```

给zval.value.str.len赋值，给(zval).value.str.val赋值。给zval.type赋值。MAKE_STD_ZVAL中已经初始化了zval.refcount\_\_gc和zval.is_ref\_\_gc	

```
#define Z_STRLEN_P(zval_p)		Z_STRLEN(*zval_p)
#define Z_STRLEN(zval)			(zval).value.str.len
```

```c
#define Z_STRVAL_P(zval_p)		Z_STRVAL(*zval_p)
#define Z_STRVAL(zval)			(zval).value.str.val
```

```c
#define Z_TYPE_P(zval_p)	Z_TYPE(*zval_p)
#define Z_TYPE(zval)		(zval).type
```

### PHP7中字符串的定义

#### 初始化变量

```
	zval	zfuncname;
	ZVAL_STRING(&zfuncname, "getTimestamp");
```

#### 宏定义

```c
#define ZVAL_STRING(z, s) do {					\
		const char *_s = (s);					\
		ZVAL_STRINGL(z, _s, strlen(_s));		\
	} while (0)

#define ZVAL_STRINGL(z, s, l) do {				\
		ZVAL_NEW_STR(z, zend_string_init(s, l, 0));		\
	} while (0)

在函数zend_string_init中初始化 zend_string

#define ZVAL_NEW_STR(z, s) do {					\
		zval *__z = (z);						\
		zend_string *__s = (s);					\
		Z_STR_P(__z) = __s;						\
		Z_TYPE_INFO_P(__z) = IS_STRING_EX;		\
	} while (0)
```

将传入的zend_string复制给_zend_value.zend_string。更新zval.u1.type_info为IS_STRING_EX

```
#define Z_STR_P(zval_p)				Z_STR(*(zval_p))
#define Z_STR(zval)					(zval).value.str
```

```c
#define Z_TYPE_INFO_P(zval_p)		Z_TYPE_INFO(*(zval_p))
#define Z_TYPE_INFO(zval)			(zval).u1.type_info
```

```c

static zend_always_inline zend_string *zend_string_init(const char *str, size_t len, int persistent)
{
	zend_string *ret = zend_string_alloc(len, persistent);

	memcpy(ZSTR_VAL(ret), str, len);
	ZSTR_VAL(ret)[len] = '\0';
	return ret;
}
#define ZSTR_VAL(zstr)  (zstr)->val
```
初始化zend_string变量，包括内存分配，初始化gc相关字段，_zend_string.zend_refcounted.refcount，_zend_string.zend_refcounted.u.type_info。 为_zend_string.val和_zend_string.len赋值
```
static zend_always_inline zend_string *zend_string_alloc(size_t len, int persistent)
{
	zend_string *ret = (zend_string *)pemalloc(ZEND_MM_ALIGNED_SIZE(_ZSTR_STRUCT_SIZE(len)), persistent);

	GC_REFCOUNT(ret) = 1;
#if 1
	/* optimized single assignment */
	GC_TYPE_INFO(ret) = IS_STRING | ((persistent ? IS_STR_PERSISTENT : 0) << 8);
#else
	GC_TYPE(ret) = IS_STRING;
	GC_FLAGS(ret) = (persistent ? IS_STR_PERSISTENT : 0);
	GC_INFO(ret) = 0;
#endif
	zend_string_forget_hash_val(ret);
	ZSTR_LEN(ret) = len;
	return ret;
}

#define GC_REFCOUNT(p)				(p)->gc.refcount
#define GC_TYPE_INFO(p)				(p)->gc.u.type_info
#define ZSTR_LEN(zstr)  (zstr)->len
```

```c
#define pemalloc(size, persistent) ((persistent)?__zend_malloc(size):emalloc(size))
```
