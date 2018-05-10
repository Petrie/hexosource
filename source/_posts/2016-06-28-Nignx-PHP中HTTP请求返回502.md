---
title: Nignx+PHP中HTTP请求返回502
date: 2016-06-28 19:38:18
description:
category:
- troubleshooting 
tags:
- nginx
- php
- php-fpm
- http
- 502
---

Nginx + PHP-FPM 报 502 错误，我想大部分 RD 都遇到过吧。根据报错的频率，可以分为两种情况，间歇性的502和连续性的502。间歇性502，是后端 PHP-FPM 不可用造成的，间歇性的502一般认为是由于 PHP-FPM 进程重启造成的。连续性502很可能是由于cgi处理时间过长导致超时所致。下面讨论超时相关的参数：

nginx 相关配置:nginx.conf
``` bash
fastcgi_connect_timeout 300;
fastcgi_send_timeout 300;
fastcgi_read_timeout 300;
```

php-fpm相关配置:php-fpm.conf
``` bash
request_terminate_timeout = 10s
```

php 相关配置:php.ini
``` bash
max_execution_time
```
