[TOC]

# gqueue

动态大小的并发安全队列。同时，`gqueue`也支持固定队列大小，固定队列大小时队列效率和标准库的`channel`无异。

**使用场景**：

该队列是并发安全的，常用于多`goroutine`数据通信且支持动态队列大小的场景。

**使用方式**：
```go
import "github.com/gogf/gf/container/gqueue"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/gqueue


## 示例1，基本使用

```go
package main

import (
    "fmt"
    "time"
    "github.com/gogf/gf/os/gtimer"
    "github.com/gogf/gf/container/gqueue"
)

func main() {
    q := gqueue.New()
    // 数据生产者，每隔1秒往队列写数据
    gtimer.SetInterval(time.Second, func() {
        v := gtime.Now().String()
        q.Push(v)
        fmt.Println("Push:", v)
    })

    // 3秒后关闭队列
    gtimer.SetTimeout(3*time.Second, func() {
        q.Close()
    })

    // 消费者，不停读取队列数据并输出到终端
    for {
        if v := q.Pop(); v != nil {
            fmt.Println(" Pop:", v)
        } else {
            break
        }
    }
}
```
在该示例中，第3秒时关闭队列，这时程序立即退出，因此结果中只会打印2秒的数据。
执行后，输出结果为：
```html
Push: 2018-09-07 14:03:00
 Pop: 2018-09-07 14:03:00
Push: 2018-09-07 14:03:01
 Pop: 2018-09-07 14:03:01
```

## 示例2，结合`select`语法使用

使用队列对象公开的`Queue.C`属性，结合`select`IO复用语法实现对队列的读取。

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/container/gqueue"
    "github.com/gogf/gf/os/gtime"
    "github.com/gogf/gf/os/gtimer"
    "time"
)

func main() {
    queue := gqueue.New()
    // 数据生产者，每隔1秒往队列写数据
    gtimer.SetInterval(time.Second, func() {
        queue.Push(gtime.Now().String())
    })

    // 消费者，不停读取队列数据并输出到终端
    for {
        select {
            case v := <-queue.C:
                if v != nil {
                    fmt.Println(v)
                } else {
                    return
                }
        }
    }
}
```
执行后，输出结果为：
```
2019-01-23 20:42:01
2019-01-23 20:42:02
2019-01-23 20:42:03
2019-01-23 20:42:04
2019-01-23 20:42:05
2019-01-23 20:42:06
...
```

## `gqueue`与`glist`

`gqueue`的底层基于`glist`链表实现动态大小特性，在队列满或者在队列空时读取数据会产生阻塞。

`glist`是一个并发安全的链表，并可以允许在关闭并发安全特性的时和一个普通的`list`链表无异，在存储和读取数据时不会发生阻塞。


## `gqueue`与`channel`
`gqueue`与标准库`channel`的性能基准测试，其中每一次基准测试的`b.N`值均为`20000000`，以保证动态队列存取一致防止`deadlock`:
```html
goos: linux
goarch: amd64
pkg: github.com/gogf/gf/container/gqueue
Benchmark_Gqueue_StaticPushAndPop-4       20000000            84.2 ns/op
Benchmark_Gqueue_DynamicPush-4            20000000             164 ns/op
Benchmark_Gqueue_DynamicPop-4             20000000             121 ns/op
Benchmark_Channel_PushAndPop-4            20000000            70.0 ns/op
PASS
```
可以看到标准库的`channel`的读写性能是非常高的，但是创建的时候由于需要初始化内存，因此创建`channel`的时候效率非常非常低（初始化即分配内存），并且受到队列大小的限制，写入的数据不能超过指定的队列大小。`gqueue`使用起来比`channel`更加灵活，不仅创建效率高（动态分配内存），不受队列大小限制(也可限定大小)。
