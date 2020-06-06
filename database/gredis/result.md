# 结果处理

可以看到通过客户端方法`Do/Receive`获取的数据都是二进制形式`[]byte`的，需要开发者手动进行数据转换。

当然，`gredis`模块也提供了`DoVar/ReceiveVar`方法，用以获取可供方便转换的`gvar.Var`通用变量结果。

通过`gvar.Var`的强大转换功能可以转换为任意的数据类型，如基本数据类型`Int`,`String`,`Strings`，或者结构体`Struct`等等。

## 示例1，`DoVar`

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
    v, _ := conn.DoVar("GET", "k")
    fmt.Println(v.String())
}
```
执行后，输出结果为：
```html
v
```

## 示例2，`ReceiveVar`

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/frame/g"
)

func main() {
	conn := g.Redis().Conn()
	defer conn.Close()
	_, err := conn.Do("SUBSCRIBE", "channel")
	if err != nil {
		panic(err)
	}
	for {
		reply, err := conn.ReceiveVar()
		if err != nil {
			panic(err)
		}
		fmt.Println(reply.Strings())
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