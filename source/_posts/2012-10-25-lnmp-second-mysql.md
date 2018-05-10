---
layout: post  
title: "LNMP环境搭建（二） 编译安装MySQL"  
description: "LNMP环境搭建（二） 编译安装MySQL"  
category: LNMP
tags: [LNMP, MySQL]  
---
**编译安装MySQL**  

**1. 创建组和用户**
>groupadd Mysql  
>useradd -g Mysql Mysql  

**2. 解压源码包并进入**
>tar zxvf Mysql-5.5.3-m3.tar.gz  
>cd Mysql-5.5.3-m3  

**3. 配置cmake编译参数**
>cmake -DCMAKE_INSTALL_PREFIX=/usr/local/Mysql \  
>-DSYSCONFDIR=/usr/local/Mysql/etc   \  
>-DMYSQL_DATADIR=/usr/local/Mysql/data \   
>-DMYSQL_TCP_PORT=3306 \   
>-DMYSQL_UNIX_ADDR=/tmp/Mysqld.sock \   
>-DMYSQL_USER=Mysql \   
>-DEXTRA_CHARSETS=all \   
>-DWITH_READLINE=1 \  
>-DWITH_SSL=system \  
>-DWITH_EMBEDDED_SERVER=1 \  
>-DENABLED_LOCAL_INFILE=1 \  
>-DWITH_INNOBASE_STORAGE_ENGINE=1 \  
>-DWITHOUT_PARTITION_STORAGE_ENGINE=2  

**4. 编译安装**
>make && make install  

**5. 将安装目录极其子目录的所属组和拥有者设置为Mysql:Mysql**
>chown -R Mysql:Mysql /usr/local/Mysql   
>cd .. 

**6. 以Mysql用户帐号的身份建立数据表**
>cd /usr/local/Mysql 
>./scripts/Mysql_install_db --basedir=/usr/local/Mysql --datadir=/usr/local/Mysql/data --user=Mysql  

 如果执行成功会出来一堆提示信息。  

 现在你已经成功安装了MySQL。你可以用如下命令启动MySQL：   
>./support-files/Mysql.server start   

 用下面的命令访问MySQL：   
>./bin/Mysql  

 你应该已经发现我们成功登录了Mysql，但是没有提供用户名密码。这样的话任何人都可以登录了，这肯定不是你想要的。 

**7. 可以通过一下命令来为root设置密码**
>/usr/local/Mysql/bin/Mysqladmin -u root password '123123'

 我们的密码是123123。  
 这样再执行`./bin/Mysql`的时候就会提示你需要密码了。 

**8. 好了，这就完事了.**
