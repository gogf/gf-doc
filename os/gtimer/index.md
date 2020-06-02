[TOC]

# gtimer

`gtimer`是一个并发安全的高性能任务定时器，类似于`Java`的`Timer`。`gtimer`内部实现采用灵活高效的分层时间轮（`Hierarchical Timing Wheel`）设计，被设计为可单进程管理维护百万级别以上数量的定时任务。

`gtimer`任务定时器与`gcron`定时任务模块区别:
- `gtimer`属于高性能模块，是框架核心模块，构建任何定时任务的基础，任何方法操作耗时均在`纳秒`级别。
- `gtimer`可适用于任何的定时任务场景中，例如: TCP通信、游戏开发等场景。
- `gcron`支持经典的`crontab`形式的定时任务语法，最小时间设定间隔为`秒`。
- `gcron`底层实现基于`gtimer`。

**使用场景**：

任何定时任务场景，大批量定时任务/延迟任务的场景，超时控制/频率控制的业务场景，对于定时时间准确度要求不高的业务场景。

**注意事项**：

1. 任何的定时任务都是有误差的，在时间轮刻度比较大，或者并发量大，负载较高的系统中尤其明显，具体请参考：https://github.com/golang/go/issues/14410
1. 定时间隔不会考虑任务的执行时间。例如，如果一项工作需要3分钟才能执行完成，并且计划每隔5分钟运行一次，那么每次任务之间只有2分钟的空闲时间。
1. 需要注意的是**单例模式**运行的定时任务，任务的执行时间会影响该任务下一次执行的**开始时间**。例如：一个每秒执行的任务，运行耗时为1秒，那么在**第1秒**开始运行后，下一次任务将会在**第3秒**开始执行。

**使用方式**：
```go
import "github.com/gogf/gf/os/gtimer"
```

**接口文档**： 

https://godoc.org/github.com/gogf/gf/os/gtimer

简要说明：
1. `New`方法用于创建自定义的任务定时器对象:
    - `slot` 参数用于指定每个时间轮的槽数；
    - `interval` 参数用于指定定时器的最小tick时间间隔；
    - `level` 为非必需参数，用于自定义分层时间轮的层数，默认为`6`；
1. `Add`方法用于添加定时任务，其中：
    - `interval` 参数用于指定方法的执行的时间间隔；
    - `job` 参数为需要执行的任务方法(方法地址)；
1. `AddEntry`方法添加定时任务，支持更多参数的控制；
1. `AddSingleton`方法用于添加单例定时任务，即同时只能有一个该任务正在运行；
1. `AddOnce`方法用于添加只运行一次的定时任务，当运行一次数后该定时任务自动销毁；
1. `AddTimes`方法用于添加运行指定次数的定时任务，当运行`times`次数后该定时任务自动销毁；
1. `Search`方法用于根据名称进行定时任务搜索(返回定时任务`*Entry`对象指针)；
1. `Start`方法用于启动定时器(使用`New`创建定时器时会自动启动)；
1. `Stop`方法用于停止定时器；
1. `Close`方法用于关闭定时器；

## 时间轮设计
<div align=center>
<img src="images/hierarchical-timing-wheel.png" />
</div>

`gtimer`采用的是 分层时间轮(`Hierarchical Timing Wheel`) 设计，使用的数据结构为`array + list`，相比较于单层时间轮设计，可以更高效地管理更多的定时任务。分层时间轮的设计类似于时钟的转动，在分层时间轮的内部，仅使用了一个`goroutine`来“滚动”刻度，有且只有最底层的时间轮会执行刻度的“滚动”。相当于底层的时间轮转一圈，上层的时间轮才会移动一个刻度，如此循环。时间间隔比较大的定时任务将会被添加到高层的时间轮，当刻度指向该任务的槽时，会将该任务移动到低层的时间轮继续运行。

在时间轮的每一个刻度上绑定了一个并发安全的链表，达到刻度时自动遍历该链表，满足条件的任务将会创建独立的`goroutine`进行异步执行。不满足条件的任务，或者需要继续执行的任务将会计算剩余的间隔时间，被重新调度到新的时间轮刻度上。


关于时间轮更详细的介绍，可以查看以下几个链接：
* http://www.embeddedlinux.org.cn/RTConforEmbSys/5107final/LiB0071.html
* https://www.slideshare.net/supperniu/timing-wheels



## 默认定时器

> 大部分的场景下使用默认的定时器即可。如果确实有创建自定义时间轮对象的必要，务必提前理解时间轮设计概念，并且防止设计的时间刻度大小溢出。

使用`gtimer`的默认定时器时，默认的时间轮槽数为`10`，间隔时间为`50ms`，分层大小为`6`，每一层的时间轮信息如下：

| 层级 | 槽数 | 间隔 | 一周大小
|---|---|---| ---
|0 | 10 | 50ms      | 500ms
|1 | 10 | 500ms     | 5000ms
|2 | 10 | 5000ms    | 50000ms
|3 | 10 | 50000ms   | 500000ms
|4 | 10 | 500000ms  | 5000000ms
|5 | 10 | 5000000ms | 50000000ms

可以使用以下两种方式修改默认的定时器参数：
1. 使用启动参数
 - `gf.gtimer.slots=100`: 修改默认的槽数为`100`；
 - `gf.gtimer.level=7`: 修改默认的分层数为`7`；
 - `gf.gtimer.interval=10`: 修改默认的时间刻度为`10毫秒`；
1. 使用环境变量
 - `GF_GTIMER_SLOTS=100`
 - `GF_GTIMER_LEVEL=7`
 - `GF_GTIMER_INTERVAL=10`



## 性能基准测试

```
goos: linux
goarch: amd64
pkg: github.com/gogf/gf/os/gtimer
Benchmark_Add-4               3000000             484 ns/op         152 B/op           5 allocs/op
Benchmark_StartStop-4       100000000            10.1 ns/op           0 B/op           0 allocs/op
PASS
ok      command-line-arguments    6.602s
```

## 示例1, 间隔任务

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gtimer"
    "time"
)

func main() {
    now      := time.Now()
    interval := 1400*time.Millisecond
    gtimer.Add(interval, func() {
        fmt.Println(time.Now(), time.Duration(time.Now().UnixNano() - now.UnixNano()))
        now = time.Now()
    })

    select { }
}
```
执行后，输出结果为:
```
2019-01-17 18:17:37.022442 +0800 CST m=+1.354132542 1.353568s
2019-01-17 18:17:38.422467 +0800 CST m=+2.754148119 1.399624s
2019-01-17 18:17:39.82318 +0800 CST m=+4.154851847 1.40066s
2019-01-17 18:17:41.222422 +0800 CST m=+5.554084408 1.399094s
2019-01-17 18:17:42.622461 +0800 CST m=+6.954112968 1.399962s
2019-01-17 18:17:44.022769 +0800 CST m=+8.354411089 1.400251s
...
```

## 示例2, 单例任务

```go
package main

import (
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/os/gtimer"
    "time"
)

func main() {
    interval := time.Second
    gtimer.AddSingleton(interval, func() {
        glog.Println("doing")
        time.Sleep(5*time.Second)
    })

    select { }
}
```
执行后，输出结果为:
```
2019-01-23 17:04:18.892 doing
2019-01-23 17:04:24.890 doing
2019-01-23 17:04:29.892 doing
2019-01-23 17:04:35.891 doing
...
```

















