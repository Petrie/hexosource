---
title: MySQL datetime vs timestamp
date: 2016-07-23 13:03:01
description:
category:
- Mysql
tags:
- Mysql
- 数据库
---

## MySQL 时间类型最佳实践

通常建立数据库表的时，最常出现的两个字段是，数据的创建时间和更新时间。这篇文章为大家整理出关于这两个字段数据类型选择（timestamp vs datetime）的推荐实践。

首先介绍下两种数据类型的试用场景

### timestamp

timestamp通常用来追踪数据记录的变化时间，通常它被设置成跟着字段的更新而更新 `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` 。如果你只是想存储某个具体的时间值，datetime字段更合适。

它具有以下特点

1. 它以UTC时间格式存储
2. 可以由Mysql自动初始化和更新
3. 可存储范围 `1970-01-01 00:00:01 UTC` to `2038-01-19 03:14:07 UTC`

### datetime

datetime字段可以很方便的通过`SELECT UNIX_TIMESTAMP(my_datetime)` 转换为Linux 时间戳。

它具有以下特点

1. 存储内容，所存即所得
2. 可存储范围`1000-01-01 00:00:00` to `9999-12-31 23:59:59`
3. 可以指定日和月字段为0值
4. 某些情况下可以设置默认值now()。但是此种方式并不推荐

#### 以上

实践一

```sql
CREATE TABLE ts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    created_at DATETIME ,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

实践一的优点

1.数据库相关操作不需要关心changed_at字段的更新，MySQL会自动在数据变动的时候将此字段自动更新为当前时间



(the end)

