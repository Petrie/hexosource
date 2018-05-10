---
layout: post  
title: "SQLServer 便捷管理多个服务器"  
description: "SQLServer 远程登录"  
category: SQL Server
tags: [SQL Server]  
published: flase
---
 
------

## 哇 你好out，还在用2005. ##
发现自己博客有个特点（这才谢了几篇，就总结出特点了），就是没篇要介绍的东西都很少，一个命令、一个技巧，但我通常都能罗出一大堆。这不，今天又是这个节奏。

------

## 从我的优点说起 ##
我的有点是什么呢？ 被某个东西折磨久了，就会主动去找一个便捷的代替方式。这次就是，SQL Server应该很多人都用过吧！好了不废话了，其实我要介绍两个东西。  
		1.“SQL Server 解决方案资源管理器”   
		2.SQL Server 下连接访问远程数据库  

------

## 1.SQL Server 解决方案资源管理器 ##
这个很简单，就是一个SQL Server的子窗口，只不过不常用SQL Server的人很难发现，工作原因我用的较多，就发现了。依次“视图”->“解决方案资源管理器”就出来了，就是这个窗口，你可以将相关的服务器和SQL文件保存在同一个解决方案里，这样找的时候十分的方便。

------

## 2.SQL Server 下连接访问远程数据库 ##
简单来说其实就几条SQL语句。  
`exec sp_addlinkedserver 'myserver','','SQLOLEDB ','远程服务器名或ip地址'`
`exec sp_addlinkedsrvlogin 'myserver','false',null,'username','password'`

有了上面两句就可以直接远程操作数据库了：  
`select * from myserver.database.dbo.table;`

就这样吧!  

------

## 拾遗 ##
[参考文章](http://www.cnblogs.com/OpenCoder/archive/2010/03/18/1689321.html "参考文章")  
