[TOC]

# 基本使用

## `Do/DoVar`方法

我们最常用的是`Do/DoVar`方法，执行同步指令，通过向`Redis Server`发送对应的`Redis API`命令，来使用`Redis Server`的服务。`Do/Var`方法最大的特点是内部自行从连接池中获取连接对象，使用完毕后自动丢回连接池中，开发者无需手动调用`Close`方法，方便使用。

`Do`和`DoVar`的区别在于返回的结果类型不一样，`DoVar`返回的是`gvar.Var`类型，该类型对象可以执行非常方便的类型转换。

## `Conn`连接对象

使用`Do/DoVar`方法已经能够满足绝大部分的场景需要，如果需要更复杂的Redis操作，那么我们可以使用`Conn`方法从连接池中获取一个连接对象，随后使用该连接对象进行操作。但需要注意的是，该连接对象不再使用时，应当显式调用`Close`方法进行关闭（丢回连接池）。

> 由于该`Conn`是个连接对象，注意该对象存在连接超时的限制，超时和服务端配置有关。

### 基本使用
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gconv"
)

func main() {
    conn := g.Redis().Conn()
    defer conn.Close()
    conn.Do("SET", "k", "v")
    v, _ := conn.Do("GET", "k")
    fmt.Println(gconv.String(v))
}
```
执行后，输出结果为：
```html
v
```
### `Send`方法

`Send`可以执行批量指令，并通过`Receive`方法一一获取返回结果。

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gconv"
)

func main() {
    conn := g.Redis().Conn()
    defer conn.Close()
    conn.Send("SET", "foo", "bar")
    conn.Send("GET", "foo")
    conn.Flush()
    // reply from SET
    conn.Receive()
    // reply from GET
    v, _ := conn.Receive()
    fmt.Println(gconv.String(v))
}
```
执行后，输出结果为：
```html
bar
```
### 订阅/发布

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gconv"
)

func main() {
    conn := g.Redis().Conn()
    defer conn.Close()
    _, err := conn.Do("SUBSCRIBE", "channel")
    if err != nil {
        panic(err)
    }
    for {
        reply, err := conn.Receive()
        if err != nil {
            panic(err)
        }
        fmt.Println(gconv.Strings(reply))
    }
}
```
执行后，程序将阻塞等待获取数据。

另外打开一个终端通过`redis-cli`命令进入Redis Server，发布一条消息：
```shell
$ redis-cli
127.0.0.1:6379> publish channel test
(integer) 1
127.0.0.1:6379>
```
随后程序终端立即打印出从Redis Server获取的数据：
```html
[message channel test]
```


## 自动序列化/反序列化

当给定的参数为`map`, `slice`, `struct`时，`gredis`内部支持自动对其使用`json`序列化，并且读取数据时可使用`gvar.Var`的转换功能实现反序列化。

### `map`存取
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gvar"
	"github.com/gogf/gf/frame/g"
)

func main() {
	var (
		err    error
		result *gvar.Var
		key    = "user"
		data   = g.Map{
			"id":   10000,
			"name": "john",
		}
	)
	_, err = g.Redis().Do("SET", key, data)
	if err != nil {
		panic(err)
	}
	result, err = g.Redis().DoVar("GET", key)
	if err != nil {
		panic(err)
	}
	fmt.Println(result.Map())
}
```

### `struct`存取
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gvar"
	"github.com/gogf/gf/frame/g"
)

func main() {
	type User struct {
		Id   int
		Name string
	}
	var (
		err    error
		result *gvar.Var
		key    = "user"
		user   = &User{
			Id:   10000,
			Name: "john",
		}
	)

	_, err = g.Redis().Do("SET", key, user)
	if err != nil {
		panic(err)
	}
	result, err = g.Redis().DoVar("GET", key)
	if err != nil {
		panic(err)
	}

	var user2 *User
	if err = result.Struct(&user2); err != nil {
		panic(err)
	}
	fmt.Println(user2.Id, user2.Name)
}
```

