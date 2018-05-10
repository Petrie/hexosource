---
title: PHP扩展开发－自动生成扩展骨架
tags:
  - php
  - php-extension
category: php
date: 2018-03-29 03:55:12
---
PHP扩展编译有两种方式

- 作为一个可装载模块或者DSO（动态共享对象）
- 静态编译到PHP

静态编译的方式直接和PHP编译到一起，步骤比较简单，但每次变更代码都要编译这个PHP代码，非常耗时，所以这里采用第二种方式

#### 开发环境
本文脚本命令仅在以下环境测试成功
```shell
操作系统：CentOS 6.9
PHP版本：PHP-7.1
```

#### 安装开发工具包

```shell
yum groupinstall -y 'Development Tools'
```
<!--more-->
#### 下载PHP源码

```shell
git clone http://git.php.net/repository/php-src.git
git checkout PHP-7.1
```

#### 安装依赖

```shell
sudo yum install  libxml2-devel
```

#### 编译安装php####
安装到~/php7.1目录下，以免覆盖系统已安装的PHP
```shell
./buildconf
./configure --prefix=$HOME/php7.1 --enable-debug --enable-maintainer-zts
make && make install
```

#### 生成扩展框架

```shell
cd $HOME/php-src/ext
$HOME/php-src/ext/ext_skel --extname=testex
```

#### 编辑`config.m4`

去掉`config.m4`文件中第16、18行前的dnl注释

```shell
 16 PHP_ARG_ENABLE(testex, whether to enable testex support,
 17 dnl Make sure that the comment is aligned:
 18 [  --enable-testex           Enable testex support])
```

#### 编译扩展

```shell
cd $HOME/php-src/ext/testex
$HOME/php7.1/bin/phpize
./configure --with-php-config=$HOME/php7.1/bin/php-config
make && make install
```

#### 测试扩展

ext_skel脚本生成的扩展已包含confirm_testex_compiled函数，所以我们用它坐测试

```shell
$HOME/php7.1/bin/php -dextension=$HOME/php7.1/lib/php/extensions/debug-zts-20160303/testex.so -r "echo confirm_testex_compiled('1');"
```

#### 总结

本文只是用php源码自带的ext_skel生成扩展框架，测试编译和安装。扩展不包含任何功能。如果想加入更多功能，可以参考http://www.phpinternalsbook.com/