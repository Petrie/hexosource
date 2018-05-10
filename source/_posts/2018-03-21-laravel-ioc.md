---
title: 如何独立的使用Laravel的IOC功能
tags:
  - php
  - laravel
  - ioc
category: laravel
date: 2018-03-21 05:55:12
---

Laravel拥有一个强大的IOC/DI容器。且可以独立于Laravel使用。本文介绍如何单独使用Laravel的container组件

<!--more-->

### 准备工作###

在家目录下新建测试目录
```shell 
mkdir ~/test-laravel-ioc && cd "$_"
```
下载container源码
```shell
composer require illuminate/container

```
新建测试文件
```shell
touch container01.php && vim "$_"

```
加载container
```php
#file container01.php
<?php
require vendor/autoload.php
use Illuminate\Container\Container;

$container = Container::getInstance();
```
执行`container01.php`，无报错则说明加载成功

### 测试Container基本功能###

新建测试文件
```shell
touch container02.php && vim "$_"
```
用container的make方法新建MyClass对象并执行其中的echo方法
```php
#file container02.php
<?php
require vendor/autoload.php
use Illuminate\Container\Container;

class MyClass
{
    public function __construct()
    {
    }
    public function echo()
    {
      echo 'testecho';
    }
}

$container = Container::getInstance();
$instance = $container->make(MyClass::class);
$instance->echo();
```

运行结果
```shell
% php container02.php
testecho
```

显示如上结果则说明容器成功实例话了对象。

### 参考###

更多容器功能参见官方文档

[Laravel IOC 官方文档](https://laravel.com/docs/5.4/container)
https://gist.github.com/davejamesmiller/bd857d9b0ac895df7604dd2e63b23afe