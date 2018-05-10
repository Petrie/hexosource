---
layout: post  
title: "LNMP环境搭建（一）安装前准备"  
description: "LNMP环境搭建（一） 安装前准备"  
category: Linux
tags: [Linux]  
---

博客后续将退出LNMP环境搭建系列博文：

1. 安装依赖库：  
	>yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers

	
 以上命令在连网下才能进行，如果不能联网需要配置本地yum源，配置方法请点击我以前的文章[点击](http://petrie.Github.com/Linux/2012/09/18/Linux-yum-iso/)

2. 下载安装所需的源码包：  
	- **PHP**相关程序包  
	1. php源码包：[点击](http://www.php.net/get/php-5.4.8.tar.gz/from/hk2.php.net/mirror)    
	2. php依赖的libmcrypt加密库：[点击](http://downloads.sourceforge.net/mcrypt/libmcrypt-2.5.8.tar.gz?modtime=1171868460&big_mirror=0)   
	2. php依赖的mcrypt加密库：[点击](http://downloads.sourceforge.net/mcrypt/mcrypt-2.6.8.tar.gz?modtime=1194463373&big_mirror=0)  
	3. php依赖的编码转换库：[点击](http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.13.1.tar.gz)    
	4. php依赖的mhash库：[点击](http://downloads.sourceforge.net/mhash/mhash-0.9.9.9.tar.gz?modtime=1175740843&big_mirror=0)  
    4. memcache源码包：[点击](http://pecl.php.net/get/memcache-2.2.5.tgz)  
	4. eaccelerator源码包：[点击](http://bart.eaccelerator.net/source/0.9.6.1/eaccelerator-0.9.6.1.tar.bz2)  
	4. PDO_MYSQL源码包：[点击](http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz)  
	4. ImageMagick源码包：[点击](http://blog.s135.com/soft/Linux/nginx_php/imagick/ImageMagick.tar.gz)  
	4. imagick源码包：[点击](http://pecl.php.net/get/imagick-2.3.0.tgz)  
    - **Nginx** 相关程序包   
	1. Nginx源码包：[点击](http://nginx.org/download/nginx-1.2.4.tar.gz)    
    - **MySQL**相关程序包   
	1. MySQL源码包：[点击](http://www.Mysql.com/get/Downloads/MySQL-5.5/Mysql-5.5.28.tar.gz/from/http://cdn.Mysql.com/)  

本文旨在在张宴博客的基础上，使得安装过程更加清晰易懂，更加适合新手。

