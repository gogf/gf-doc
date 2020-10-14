[TOC]

# guid

`guid`提供了更简便更高性能的唯一数生成功能。 

> `guid`模块的设计目的在于提供一种使用更简便、性能更高效且能满足绝大多数业务场景的唯一数生成。`guid`的设计比较简单，详情可参考实现源码。如果需要严格的全局唯一数生成，请参考`UUID`。

**使用方式**：
```go
import "github.com/gogf/gf/util/guid"
```

**接口文档**： 

https://pkg.go.dev/github.com/gogf/gf/util/guid


## 基本介绍

`guid`通过`S`方法生成`32`字节的唯一数，该方法定义如下：
```go
func S(data ...[]byte) string
```
1. 通过不带任何参数的方式使用，该方法生成的唯一数将会有以下方式构成：
    
    `MAC(7) + PID(4) + TimestampNano(12) + Sequence(3) + RandomString(6)`

    其中：
    - `MAC`表示当前机器的`MAC`地址哈希值，由`7`个字节构成；
    - `PID`表示当前机器的进程ID哈希值，由`4`个字节构成；
    - `TimestampNano`表示当前的纳秒时间戳哈希值，由`12`个字节构成；
    - `Sequence`表示当前进程并发安全的序列号，由`3`个字节构成；
    - `RandomString`表示随机数，由`6`个字节构成；
    
1. 通过自定义任何参数的方式使用，该方法生成的唯一数将会有以下方式构成：
    
    `Data(7/14) + TimestampNano(12) + Sequence(3) + RandomString(3/10)`

    主要说明：
    - `Data`表示自定义的参数，参数类型为`[]byte`，最多支持`2`个参数输入，由`7`或`14`个字节构成；
    - 需要注意的是，输入的自定义参数需要在业务上具有一定的唯一识别性，使得生成的唯一数更有价值；
    - 不管每一个`[]byte`参数长度为多少，最终都将通过哈希方式生成`7`个字节的哈希值。
    - `TimestampNano`表示当前的纳秒时间戳哈希值，由`12`个字节构成；
    - `Sequence`表示当前进程并发安全的序列号，由`3`个字节构成；
    - `RandomString`表示随机数，由`3`或者`10`个字节构成，即:
        - 如果给定`1`个自定义参数，那么剩余的字节将会使用随机数占位，长度为`10`个字节；
        - 如果给定`2`个自定义参数，那么剩余的字节将会使用随机数占位，长度为`3`个字节；

## 基准测试
```
goos: linux
goarch: amd64
pkg: github.com/gogf/gf/util/guid
Benchmark_S
Benchmark_S-4          	 1525240	       737 ns/op
Benchmark_S_Data_1
Benchmark_S_Data_1-4   	 1294222	       920 ns/op
Benchmark_S_Data_2
Benchmark_S_Data_2-4   	 1344630	       889 ns/op
PASS
```

## 示例1，基本使用
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/util/guid"
)

func main() {
	fmt.Printf("TraceId: %s", guid.S())
}
```
执行后，输出结果为：
```
TraceId: oa9sdw03dk0c35q9bdwcnz42p00trwfr
```

## 示例2，自定义参数

我们的`SessionId`生成需要具有比较好的唯一性，且需要防止轻易的碰撞，因此可以使用以下方式：
```go
func CreateSessionId(r *ghttp.Request) string {
    var (
        address = request.RemoteAddr
        header  = fmt.Sprintf("%v", request.Header)
    )
    return guid.S([]byte(address), []byte(header))
}
```
可以看到，`SessionId`需要依靠自定义的两个输入参数`RemoteAddr`, `Header`来生成，这两个参数在业务上具有一定的唯一识别性，且通过`guid.S`方法的设计构成，生成的唯一数将会非常随机且唯一，既满足了业务需要也保证了安全。






