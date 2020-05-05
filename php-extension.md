# PHP扩展

## 基本语法

### 定义函数

```c
ZEND_FUNCTION(academy_hello)
{
    php_printf("Hello Laravel Academy!\n");
}

ZEND_FUNCTION(sample_long)
{
    RETVAL_LONG(42);
    return;
}
ZEND_FUNCTION(sample_getlong) {
    long foo;
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "l", &foo) == FAILURE)
    {
        RETURN_NULL();
    }
    php_printf("The integer value of the parameter is: %ld\n", foo);
    RETURN_TRUE;
}

/* {{{ calc_functions[]
 *
 * Every user visible function must have an entry in calc_functions[].
 */
const zend_function_entry calc_functions[] = {
    ZEND_FE(academy_hello, NULL)
    ZEND_FE(sample_long, NULL)
    ZEND_FE(sample_getlong, NULL)
    PHP_FE_END    /* Must be the last line in calc_functions[] */
};
/* }}} */

/* {{{ calc_module_entry
 */
zend_module_entry calc_module_entry = {
    STANDARD_MODULE_HEADER,
    "calc",
    calc_functions,
    PHP_MINIT(calc),
    PHP_MSHUTDOWN(calc),
    PHP_RINIT(calc),        /* Replace with NULL if there's nothing to do at request start */
    PHP_RSHUTDOWN(calc),    /* Replace with NULL if there's nothing to do at request end */
    PHP_MINFO(calc),
    PHP_CALC_VERSION,
    STANDARD_MODULE_PROPERTIES
};
/* }}} */
```

### 获取函数参数

* 获取字符串

  \`\`\`c

ZEND\_FUNCTION\(sample\_hello\_world\) { char _name; size\_t name\_len; zend\_string_ strg;

```text
  if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s", &name, &name_len) == FAILURE)
  {
          RETURN_NULL();
  }
  strg = strpprintf(0, "Hello %s ", name);
  RETURN_STR(strg);
```

}

```text
- 获取long

  ```c
  ZEND_FUNCTION(sample_getlong) {
      long foo;
      if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "l", &foo) == FAILURE)
      {
          RETURN_NULL();
      }
      php_printf("The integer value of the parameter is: %ld\n", foo);
      RETURN_TRUE;
  }
```

​

