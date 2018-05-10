---
layout: post  
title: "Nginx配置PATHINFO"  
description: "Nginx默认的配置是不支持PATHINFO的，今天遇到了这个问题，记录下解决方法"  
category: Nginx
tags: [Nginx, ThinkPHP, PATHINFO]
---

最近在学习php，《PHP和MySQL
Web开发》零零碎碎的算是看完了，讲了一下基础的东西，对于入门是足够了。前几天开始些项目，是在先前别人的基础上写的，同时有事两个人一起写。完成之后看了看，代码太乱了，包括代码结构和命名规范。遂决定找点着方便的资料。  
在网上搜来搜去，找到了篇挺不错的PHP框架（ThinkPHP）教程：http://www.thinkphp.cn/info/155.html
着实还不错，推荐学习PHP的同学看看。  
这篇教程的默认环境是wamp的，由于我用的是Nginx，在学习第二篇的时候就遇到了问题。ThinkPHP支持的URL模式：PATHINFO
Nginx默认配置是不支持的。本以为是自己ThinkPHP环境配置的问题，后来删了所有的相关文件（我的一贯作风--屡试不爽），重新配置ThinkPHP环境，可是还是同样的报404错误。这样的话只能是Nginx和php配置的问题了。  
URL首先是通过Nginx解析的，先从Nginx下手。猜想Nginx是不是不支持ThinkPHP的PATHINFO呢。于是百度一下，便一目了然了。  
试了几篇博文的方法，最后终于弄好了。  
**废话结束**  
打开Nginx的配置文件nginx.conf  
在server中加入一下配置：
>location ~ \.php {  
>            root  d:/ThinkPHP/;  
>            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  
>            fastcgi_pass   127.0.0.1:9000;  
>            fastcgi_index  index.php;  
>            include        fastcgi_params;   
>            #pathinfo support   
>            set $real_script_name $fastcgi_script_name;  
>            set $path_info "";  
>            if ( $fastcgi_script_name ~ "^(.+?\.php)(/.+)$"){  
>                set $real_script_name $1;  
>                set $path_info $2;  
>            }
>            fastcgi_param SCRIPT_NAME $real_script_name;  
>            fastcgi_param PATH_INFO $path_info;   
> }  

**需要注意的是那个if判断语句，在(的前后都必须有空格，否则Nginx会报配置语法错误。**





