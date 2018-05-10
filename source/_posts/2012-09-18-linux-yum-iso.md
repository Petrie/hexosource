---
layout: post  
title: "利用CentOS.ISO配置本地yum源"  
description: "将CentOS的dvdiso挂载到系统中，利用其中的rpm包配置yum源"  
category: Linux
tags: [Linux, centos, yum, iso, 本地yum]  
---  

#扯淡#
公司的网络要用代理，许多非常cool的东西都玩不了，真垃圾。  
组长让搞一个LNMP的教程。  
上次在开发机上乱搞，把LNMP搞乱了，程序不能运行，调了好久，最后只好请来别的组的一位大神才解决。谁让自己是LNMP菜鸟呢!组里也都是做.Net的  
从此我就不敢再在开发机上乱弄了。 （这事才几天，缓缓再说）。  

开发联网上网不用通过代理，用起来爽啊，装点啥直接yum。

这几天还得把LNMP教程弄出来。  

用不了开发机，只能在本机折腾了，先装Mysql，源码安装要用到cmake，不能用yum（不能上网），自己找依赖吧。

本想依赖最多有1，2层吧，可是事实不是这样，各种深度，各种广度啊。果断改变策略。想起以前在uplooking学Linux的时候老师曾教过本地yum配置。所幸试一试。

废话了这么多终于要进入正题了...

#正题#
__1__.将下好的dvdiso复制到虚拟机的 /home/petrie/ 下并重命名为centos.iso。此时iso的目录为/home/petrie/centos.iso  
__2__.将dvdiso文件绑定到目录/mnt/centos_iso  
>mkdir /mnt/centos_iso  
>mount -o loop /home/petrie/centos.iso /mnt/centos_iso

*好了，现在光盘中的rpm都可以用来安装了，但是这能安装那些没有依赖的，有依赖的要自行处理，接下来要安装的东西后面会用到，十分幸运，她没有依赖*  
__3__.安装配置软件   
>cd /mnt/centos_iso/CentOS  
>rpm -ivh createrepo-0.4.11-3.el5.noarch.rpm  

__4__.运行createrepo
>cd /mnt  
>createrepo __./__
*这个过程挺长的2，3分钟吧*  

__5__.修改配置
>cd /etc/yum.repos.d/  
>vim dvdiso.repo  

在dvdiso.repo中写入以下内容：
>\[DVDISO\]  
>name=DVD ISO  
>baseurl=file:///mnt/   
>enabled=1  
>gpgcheck=0  

注意：__步骤5 需要将/etc/yum.repos.d/下的文件清空，也就是说/etc/yum.repos.d/下只能有dvdiso.repo文件__  
这就好了，执行 yum clean all;yum list,执行成功则说明配置成功了。
赶紧装个软件试试吧~
pretty cool！ huh~ 






