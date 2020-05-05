# Nginx

## Nginx

### cgi\#\#\#

CGI scripts is a way how to run a server side script when a HTTP request comes;CGI脚本是一种如何在服务端运行HTTP请求的方式

### fastcgi\#\#\#

比cgi更好的一种方式，GCI很慢，fastcgi更快

### php-fpm\#\#\#

fastcgi的php实现

### 限流

#### 限流算法令牌桶和漏桶对比：

* 令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；
* 漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝；
* 令牌桶限制的是平均流入速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个令牌），并允许一定程度突发流量；
* 漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不能一次是1，下次又是2），从而平滑突发流入速率；
* 令牌桶允许一定程度的突发，而漏桶主要目的是平滑流入速率；

### 配置优化性能篇

#### TCP优化

#### 开启Gzip

#### 开启缓存

## 配置选项\#\#

#### 虚拟主机与请求的转发

**server\_name匹配顺序**

1. 首先选择所有字符串完全匹配的server\_name，如www.testweb.com
2. 其次选择通配符在前面的server\_name，如\*.testweb.com
3. 再次选择通配符在后面的server\_name，如www.testweb.\*
4. 最后选择使用正则表达式才匹配的server\_name，如~^\.testweb\.com

**location 匹配顺序**

1. = 表示把URI作为字符串，以便与参数中的uri做完全匹配。例如

   ```text
   location =/ {
     #只有用户请求是/时，才会使用location下的配置
   }
   ```

2. ~表示匹配URI时是字母大小写敏感的。
3. ~\*表示匹配URI是忽略字母大小写问题。
4. ^~表示匹配URI时只需要其前半部分与uri参数匹配即可。例如：

   ```text
   location ^~ /images/{
     #以/images/开始的请求都会匹配上
   }
   ```

5. @表示仅用于Nginx服务内部请求之间的重定向，带有@的location不直接处理用户请求。

#### 文件路径定义

```text
alias
root
```

**根据HTTP返回码重定向**

```text
error_page    404             /404.html
error_page     502 503 504    /50x.html
error_page    403                http://example.com/forbidden.html;
error_page    404                =@fetch;
```

* 重定义错误码

  ```text
  error_page    404    =200    /empty.gif
  error_page    404    =403    /forbidden.gif
  ```

* 跳转后决定错误码

  ```text
  error_page 404    = /empty.gif
  ```

* 如果不想修改URI，只是想让这样的请求重定向到另一个location中进行处理，那么可以这样设置“

  ```text
  location /{
    error_page     404
  }
  ```

**try\_files**

尝试按照顺序访问每一个path，如果可以有效的读取，就直接向用户返回这个path对应的文件请求结束，否则继续向访问。\*\*必须保证一个uri参数存在。

```text
try_files /system/maintenace.html $uri $uri/index.html $uri.html @other;
location @other{
  proxy_pass    http://backend;
}
```

#### 内存及磁盘资源的分配

#### 网络连接的设置

**发送响应的超时时间**

```text
send_timeout 60
```

#### reset\_timeout\_connection

连接超时后将通过向客户端发送RST包来直接重置连接。不同过四次握手，相比于正常的方式，FIN\_WAIT\_1、FIN\_WAIT\_2、TIME\_WAIT状态的TCP连接。

注意：使用RST重置包关闭连接会带来一些问题，默认情况下不会开启。

**lingering\_time**

lingering\_close起用后，这个配置对于上传大文件很有用，上文讲过，当用户请求的Content-Length大于max\_client\_body\_size时，Nginx服务会立刻向用户发送413返回值，仍然持续不断的上传HTTP body，这时，经过了lingering\_time设置后，Nginx将不管用户是否仍在上传，都会把连接关闭掉。

**lingering\_timeout**

**keeplive\_disale**

针对浏览器限制keepalive

```text
keepalive_disable [msie6|safari|none]
```

**keepalive\_timeout超时时间**

一个连接闲置超过一定时间后默认75s关闭。

**keepalive\_requests**

一个keepalive长连接上允许承载的请求最大数.

一个keepalive连接上默认最多只能发送100个请求。

**tcp\_nodelay**

确定对keepalive连接是否使用TCP\_NODELAY选项

**tcp\_nopush**

打开tcp\_nopush后，将会在发送响应时把整个响应包头放到一个tcp包中发送。

#### MIME类型的设置

**mime映射**

```text
types {
  text/html        html;
  text/html        conf;
  image/gif        gif;
  image/jpeg    jpg;
}
```

**默认MIME type**

```text
default_type text/plain;
```

找不到相应的MIME type与文件扩展名之间的映射时，使用默认的MIME type作为HTTP header中的Content-Type。

#### 对客户端请求的限制

**按照HTTP方法名限制用户请求**

```text
limit_except method {...}
```

**HTTP请求包体的最大值**

```text
client_max_body_size size
```

**对请求的限速limit\_rate**

```text
limit_rate speed;
```

请求限制每秒传输的字节数。

**limit\_rate\_after**

向客户端发送的响应长度超过limit\_rate\_after后才开始限速。

#### 文件操作的优化

**sendfile 系统调用**

启用linux上的sendfile系统调用来发送文件，减少内核态禹用户态直接的内存复制，这样就会从磁盘读取文件后直接在内核态发送到网卡设备，提高了发送文件的效率。

**AIO系统调用**

是否在FreeBSD或Linux系统上启用内核级别的移步文件I/O功能。

**directio**

对大文件的读取速度有优化作用

**directio\_alignment**

**打开文件缓存open\_file\_cache**

存储信息

1. 文件句柄、文件大小和上次修改时间
2. 已经打开过的目录结构
3. 没有找到或者没有权限操作的文件信息。

**是否缓存文件错误的信息open\_file\_cache\_errors**

**不被淘汰的最小访问次数**

**检验缓存中元素有效性的频率**

#### 对客户端请求的特殊处理

**忽略不合法的HTTP头部**

**HTTP头部是否允许下划线**

**对If-Modified-Since头部的处理策略**

服务器对比request中的if-modified-since时间，如果过期则返回内容，状态吗200，否则返回304 Not modified

**文件未找到时是否记录到error日志**

**DNS解析地址**

**DNS解析超时时间**

**返回错误页面时，是否在Server中注明Nginx版本**

### 开发一个简单的HTTP模块

#### 准备工作

**整形的封装**

**ngx\_str\_t字符串**

**ngx\_list\_t 链表**

**ngx\_table\_elt\_t HTTP头部**

**ngx\_buf\_t 处理大数据**

**ngx\_chain\_t**

