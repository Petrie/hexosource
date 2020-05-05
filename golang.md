# Golang

## 单元测试

> [https://www.oschina.net/translate/building-a-testable-webapp](https://www.oschina.net/translate/building-a-testable-webapp)

## 内存管理

> [http://legendtkl.com/2017/04/02/golang-alloc/](http://legendtkl.com/2017/04/02/golang-alloc/)

## 垃圾回收

> [http://legendtkl.com/2017/04/28/golang-gc/](http://legendtkl.com/2017/04/28/golang-gc/)

## 资料

> [https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.2.md](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.2.md)

## Go的http包详解

> [https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/03.4.md](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/03.4.md)

## 基础数据类型

> [https://research.swtch.com/godata](https://research.swtch.com/godata)

## go语言设计初衷

* 有像C/C++那样的性能，可以做系统开发
* 没有繁琐的类型系统，有简单统一化的模块化依赖管理，编译速度飞快
* 像Java的垃圾回收
* 像python那样简单易学，拥有灵活的类型，支持函数式编程，异步IO
* 编译器进行类型检查
* 并发编程
* channelsao

```go
package main

import "fmt"

func sum(s []int, c chan int) {
    sum := 0
    for _, v := range s {
        sum += v
    }
    c <- sum // send sum to c
}

func main() {
    s := []int{7, 2, 8, -9, 4, 0}

    c := make(chan int)
    go s(s[:len(s)/2], c)
    go sum(s[len(s)/2:], c)
    x, y := <-c, <-c // receive from c

    fmt.Println(x, y, x+y)
}
```

