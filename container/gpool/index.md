[TOC]

# gpool

对象复用池（并发安全）。将对象进行缓存复用，支持`过期时间`、`创建方法`及`销毁方法`定义。


**使用场景**：

任何需要支持定时过期的对象复用场景。

**使用方式**：
```go
import "github.com/gogf/gf/container/gpool"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/gpool

需要注意两点：
1. `New`方法的过期时间类型为`time.Duration`；
1. 对象`创建方法`(`newFunc NewFunc`)返回值包含一个`error`返回，当对象创建失败时可由该返回值反馈原因；
1. 对象`销毁方法`(`expireFunc...ExpireFunc`)为可选参数，用以当对象超时/池关闭时，自动调用自定义的方法销毁对象；

## gpool与sync.Pool

`gpool`与`sync.Pool`都可以达到对象复用的作用，但是两者的设计初衷和使用场景不太一样。

`sync.Pool`的对象生命周期不支持自定义过期时间，究其原因，`sync.Pool`并不是一个Cache；`sync.Pool`设计初衷是为了缓解GC压力，`sync.Pool`中的对象会在GC开始前全部清除；并且`sync.Pool`不支持对象创建方法及销毁方法。

## 使用示例1，基本使用

```go
package main

import (
    "github.com/gogf/gf/container/gpool"
    "fmt"
    "time"
)

func main () {
    // 创建一个对象池，过期时间为1秒
    p := gpool.New(time.Second, nil)

    // 从池中取一个对象，返回nil及错误信息
    fmt.Println(p.Get())

    // 丢一个对象到池中
    p.Put(1)

    // 重新从池中取一个对象，返回1
    fmt.Println(p.Get())

    // 等待2秒后重试，发现对象已过期，返回nil及错误信息
    time.Sleep(2*time.Second)
    fmt.Println(p.Get())
}
```

## 使用示例2，创建及销毁方法

我们可以给定动态创建及销毁方法。

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gpool"
	"github.com/gogf/gf/net/gtcp"
	"github.com/gogf/gf/os/glog"
	"time"
)

func main() {
	// 创建对象复用池，对象过期时间为3秒，并给定创建及销毁方法
	p := gpool.New(3*time.Second, func() (interface{}, error) {
		return gtcp.NewConn("www.baidu.com:80")
	}, func(i interface{}) {
		glog.Println("expired")
		i.(*gtcp.Conn).Close()
	})
	conn, err := p.Get()
	if err != nil {
		panic(err)
	}
	result, err := conn.(*gtcp.Conn).SendRecv([]byte("HEAD / HTTP/1.1\n\n"), -1)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(result))
	// 丢回池中以便重复使用
	p.Put(conn)
	// 等待一定时间观察过期方法调用
	time.Sleep(4*time.Second)
}
```
执行后，终端输出结果：
```html
HTTP/1.1 302 Found
Connection: Keep-Alive
Content-Length: 17931
Content-Type: text/html
Date: Wed, 29 May 2019 11:23:20 GMT
Etag: "54d9749e-460b"
Server: bfe/1.0.8.18


2019-05-29 19:23:24.732 expired
```




