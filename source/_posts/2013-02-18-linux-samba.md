---
layout: post  
title: "Linux下搭建Samba服务器"  
description: "Linux下搭建Samba服务器"  
category: Linux
tags: [Linux, samba]
---
##Linux下搭建Samba服务器##

- **安装文件**  
三种方式获取安装文件  
*1. 网络下载*   
在http://rpmfind.net搜索"samba",下载对应rpm文件  
*2. 安装光盘*  
通过mount绑定到系统，然后拷贝使用[详见](http://http://petrie.Github.com/Linux/2012/09/18/Linux-mount-iso/)  
*3. iso文件*   
通过mount绑定到系统，然后拷贝使用[详见](http://http://petrie.Github.com/Linux/2012/09/18/Linux-mount-iso/)    

- **安装命令**  
采用RPM方式安装。  
>rpm -ivh samba_版本信息.rpm  


- **配置Samba**  
配置文件位置:
>/etc/samba/smb.conf

