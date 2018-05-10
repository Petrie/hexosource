---
title: PHP设置连接mysql超时时间
date: 2017-06-04 20:55:12
category: php
tags: [php ,mysql]
---

本文将分别介绍PHP的mysql扩展, mysqli扩展, mysql_pdo扩展,mysqlnd扩展和libmysql 这些名词的含义。以及他们之间的关系。最后再介绍如何配置mysql的超时时间。

# 一、mysql,mysqli,mysql_pdo,mysqlnd扩展

当考虑连接到MySQL数据库服务器的时候，有三种主要的API可供选择：

- PHP的MySQL扩展
- PHP的mysqli扩展
- PHP数据对象(PDO)

三者都有各自的优缺点。下面的讨论就是为了对每种API的关键方面给出一个简短的介绍。

<!--more-->
## *什么是PHP的MySQL扩展?*(废弃)

这是设计开发允许PHP应用与MySQL数据库交互的早期扩展。*mysql*扩展提供了一个面向过程 的接口，并且是针对MySQL4.1.3或更早版本设计的。因此，这个扩展虽然可以与MySQL4.1.3或更新的数据库服务端 进行交互，但并不支持后期MySQL服务端提供的一些特性。

## *什么是PHP的mysql_pdo扩展*

*PHP 数据对象* （PDO） 扩展为PHP访问数据库定义了一个轻量级的一致接口。实现 PDO 接口的每个数据库驱动可以公开具体数据库的特性作为标准扩展功能。 注意利用 PDO 扩展自身并不能实现任何数据库功能；必须使用一个 具体数据库的 PDO 驱动 来访问数据库服务。

PDO 提供了一个 *数据访问* 抽象层，这意味着，不管使用哪种数据库，都可以用相同的函数（方法）来查询和获取数据。 PDO *不*提供 *数据库* 抽象层；它不会重写 SQL，也不会模拟缺失的特性。如果需要的话，应该使用一个成熟的抽象层。

从 PHP 5.1 开始附带了 PDO，在 PHP 5.0 中是作为一个 PECL 扩展使用。 PDO 需要PHP 5 核心的新 OO 特性，因此不能在较早版本的 PHP 上运行。

## 什么是PHP的mysqli扩展

*mysqli*扩展，我们有时称之为MySQL*增强*扩展，可以用于使用 MySQL4.1.3或更新版本中新的高级特性。*mysqli*扩展在PHP 5及以后版本中包含。

*mysqli*扩展有一系列的优势，相对于*mysql*扩展的提升主要有：

- 面向对象接口
- prepared语句支持（译注：关于prepare请参阅mysql相关文档）
- 多语句执行支持
- 事务支持
- 增强的调试能力
- 嵌入式服务支持

## *什么是PHP的mysqlnd扩展*

*为了与MySQL数据库服务端进行交互，*mysql*扩展，*mysqli*扩展， PDO MySQL驱动都使用了实现了必要的协议的底层库。以前，可用的库只有MySQL客户端库和*libmysql*。*

然而，*libmysql*包含的接口没有针对与PHP的应用交互进行优化，*libmysql* 是早期为C应用程序设计的。基于这个原因，MySQL Native驱动*mysqlnd*，作为*libmysql*的一个 针对PHP应用的修改版本被开发。

*mysql*，*mysqli*以及PDO Mysql驱动都可以各自配置使用 *libmysql*或者*mysqlnd*。*mysqlnd*作为一个专门设计 用于PHP系统的库，它在内存和速度上都比*libmysql*有很大提升。非常希望你去尝试这些提升。

# 二、如何设置php连接mysql的超时时间

php连接mysql的超时可以分为三种：

1. 连接超时
2. 读超时
3. 写超时

因为php连接mysql当前有两种连接方式，libmysql和mysqlnd（推荐）。

## *首先libmysql*

在libmysql这个底层库中，提供了MYSQL_OPT_CONNECT_TIMEOUT、MYSQL_OPT_READ_TIMEOUT、MYSQL_OPT_WRITE_TIME设置项的，并且提供了相关的API。

源码位置

`mysql-VERSION/sql-common/client.c 3020行` 

MYSQL_OPT_CONNECT_TIMEOUT可以直接设置。但MYSQL_OPT_READ_TIMEOUT、MYSQL_OPT_WRITE_TIME因为mysqli没有导入这两个常量。所以需要查看MySQL的源码，得到MYSQL_OPT_READ_TIMEOUT、MYSQL_OPT_WRITE_TIME的实际值，然后直接调用mysql_option。但是mysql_pdo是写死的，所以没有任何办法设置MYSQL_OPT_READ_TIMEOUT、MYSQL_OPT_WRITE_TIME。

源码位置：

`mysql-VERSION/include/mysql.h 160行`

但是MYSQL_OPT_READ_TIMEOUT在mysql5.6版本以及之前（我们线上用的5.6），这个参数有以下三个限制。

1. 只能在TCP／IP协议下工作
2. MySQL server版本必须大于5.1.2
3. 只能在windows下生效。

## *其次是mysqlnd*

对于libmysql和mysqlnd来讲，他们都是mysql、mysqli、mysqlpdo的底层库，这三个扩展才是对外暴露api的，所以说，他们对于这三种超时的支持，从代码层面是一样的。

但是对于**MYSQL_OPT_READ_TIMEOUT**，mysqlnd在php.ini层面多处配置

`mysqlnd.net_read_timeout`

这个配置会在执行耗时较长的sql时生效，并且报错“2006-MySQL Server has gone away”。

这个参数和libmysql的MYSQL_OPT_READ_TIMEOUT是极为相似的。但是MYSQL_OPT_READ_TIMEOUT这个参数是有使用场景的，上一小结已经阐述。

MYSQL_OPT_CONNECT_TIMEOUT和MYSQL_OPT_WRITE_TIME和libmysql是相同的。

# *总而言之*

推荐方案

- 使用mysqlnd
- 配置 mysqlnd.net_read_time 



# 引用

- [mysql支持的四种协议](http://fendou.org/post/2011/05/06/mysql-communication-protocols/)
- [MYSQL_OPT_READ_TIMEOUT设置](blog.csdn.net/heiyeshuwu/article/details/5869813)
- [mysqlnd.net-read-time官方解释](http://php.net/manual/zh/mysqlnd.config.php#ini.mysqlnd.net-read-timeout)
- [MYSQL_OPT_CONNECT/READ/WRITE_TIMEOUT官方解释](https://dev.mysql.com/doc/refman/5.6/en/mysql-options.html)
