# 网络编程

## 惊群

### accept惊群

网络编程中，在父进程中bind,listen，然后fork出子进程，所有的子进程都accept这个监听句柄。这样，当新连接过来时，大家会发现，所有子进程都会被唤醒，执行accept操作，但是只有一个进程会成功。

* accept系统调用已经不存在惊群了（至少我在2.6.18内核版本上已经不存在）

### epoll惊群

* 如果我们在子进程内不是阻塞调用accept，而是用epoll\_wait，就会发现，新连接过来时，多个子进程都会在epoll\_wait后被唤醒！
* nginx 方案加锁
* lighttp 未处理
* swoole未处理

### SO\_REUSEPORT

Linux内核的3.9版本带来了SO\_REUSEPORT特性，该特性支持多个进程或者线程绑定到同一端口，提高服务器程序的性能，允许多个套接字bind\(\)以及listen\(\)同一个TCP或UDP端口，并且在内核层面实现负载均衡。

