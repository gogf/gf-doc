[TOC]

# gtype

并发安全基本类型。

**使用场景**：

`gtype`使用得非常频繁，任何需要并发安全的场景下都适用。

在普通的并发安全场景中，一个基本类型的变量，特别是一个`struct`含有若干的属性，往往使用互斥(读写)锁或者多把(读写)锁来进行安全管理。
但这样的使用中，`变量/struct/属性`的操作性能**十分低下**，且由于互斥锁机制的存在往往使得操作变得相当复杂，必须小心翼翼地维护好`变量/属性`的并发安全控制（特别是`(RW)Mutex`）。

`gtype`针对于最常用的基本数据类型，提供了对应的并发安全数据类型，便于在并发安全场景下更好地维护变量/属性，开发者无需在`struct`中再创建和维护繁琐的`(RW)Mutex`。由于`gtype`维护的是基本类型的并发安全，因此内部基本都使用了`atomic`原子操作来维护并发安全性，因此效率往往会比`(RW)Mutex`互斥锁高出数十倍。

**使用方式**：

```go
import "github.com/gogf/gf/container/gtype"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/gtype


## 性能测试

基准测试结果如下:
```shell
john@john-B85M:~/Workspace/Go/GOPATH/src/github.com/gogf/gf/container/gtype$ go test -bench=".*"  -benchmem
goos: linux
goarch: amd64
pkg: github.com/gogf/gf/container/gtype
BenchmarkInt_Set-4            300000000           5.87 ns/op        0 B/op        0 allocs/op
BenchmarkInt_Val-4            2000000000          0.46 ns/op        0 B/op        0 allocs/op
BenchmarkInt_Add-4            300000000           5.86 ns/op        0 B/op        0 allocs/op
BenchmarkInt32_Set-4          300000000           5.87 ns/op        0 B/op        0 allocs/op
BenchmarkInt32_Val-4          2000000000          0.47 ns/op        0 B/op        0 allocs/op
BenchmarkInt32_Add-4          300000000           5.85 ns/op        0 B/op        0 allocs/op
BenchmarkInt64_Set-4          300000000           5.88 ns/op        0 B/op        0 allocs/op
BenchmarkInt64_Val-4          2000000000          0.46 ns/op        0 B/op        0 allocs/op
BenchmarkInt64_Add-4          300000000           5.88 ns/op        0 B/op        0 allocs/op
BenchmarkUint_Set-4           300000000           5.88 ns/op        0 B/op        0 allocs/op
BenchmarkUint_Val-4           2000000000          0.46 ns/op        0 B/op        0 allocs/op
BenchmarkUint_Add-4           300000000           5.87 ns/op        0 B/op        0 allocs/op
BenchmarkUint32_Set-4         300000000           5.86 ns/op        0 B/op        0 allocs/op
BenchmarkUint32_Val-4         2000000000          0.50 ns/op        0 B/op        0 allocs/op
BenchmarkUint32_Add-4         200000000           5.86 ns/op        0 B/op        0 allocs/op
BenchmarkUint64_Set-4         300000000           5.86 ns/op        0 B/op        0 allocs/op
BenchmarkUint64_Val-4         2000000000          0.47 ns/op        0 B/op        0 allocs/op
BenchmarkUint64_Add-4         300000000           5.85 ns/op        0 B/op        0 allocs/op
BenchmarkBool_Set-4           300000000           5.85 ns/op        0 B/op        0 allocs/op
BenchmarkBool_Val-4           2000000000          0.46 ns/op        0 B/op        0 allocs/op
BenchmarkString_Set-4         20000000            90.1 ns/op       23 B/op        1 allocs/op
BenchmarkString_Val-4         2000000000          1.58 ns/op        0 B/op        0 allocs/op
BenchmarkBytes_Set-4          20000000            76.2 ns/op       35 B/op        2 allocs/op
BenchmarkBytes_Val-4          2000000000          1.58 ns/op        0 B/op        0 allocs/op
BenchmarkInterface_Set-4      50000000            30.7 ns/op        8 B/op        0 allocs/op
BenchmarkInterface_Val-4      2000000000          0.74 ns/op        0 B/op        0 allocs/op
BenchmarkAtomicValue_Store-4  50000000            27.3 ns/op        8 B/op        0 allocs/op
BenchmarkAtomicValue_Load-4   2000000000          0.73 ns/op        0 B/op        0 allocs/op
PASS
ok   github.com/gogf/gf/container/gtype 49.454s
```


## 使用示例

`gtype`并发安全基本类型的使用非常简单，往往就类似以下几个方法(以`gtype.Int`类型举例)：

```go
func NewInt(value ...int) *Int
func (v *Int) Add(delta int) (new int)
func (v *Int) Cas(old, new int) bool
func (v *Int) Clone() *Int
func (v *Int) Set(value int) (old int)
func (v *Int) String() string
func (v *Int) Val() int
```

### 示例1，基本使用

```go
package main

import (
    "github.com/gogf/gf/container/gtype"
    "fmt"
)

func main() {
    // 创建一个Int型的并发安全基本类型对象
    i := gtype.NewInt()

    // 设置值
    fmt.Println(i.Set(10))

    // 获取值
    fmt.Println(i.Val())

    // 数值-1，并返回修改之后的数值
    fmt.Println(i.Add(-1))
}
```

执行后，输出结果为：
```html
0
10
9
```
### 示例2，JSON序列化/反序列
`gtype`模块下的所有容器类型均实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/gtype"
    )

    func main() {
        type Student struct {
            Id     *gtype.Int
            Name   *gtype.String
            Scores *gtype.Interface
        }
        s := Student{
            Id:     gtype.NewInt(1),
            Name:   gtype.NewString("john"),
            Scores: gtype.NewInterface([]int{100, 99, 98}),
        }
        b, _ := json.Marshal(s)
        fmt.Println(string(b))
    }
    ```
    执行后，输出结果：
    ```
    {"Id":1,"Name":"john","Scores":[100,99,98]}
    ```
1. `Unmarshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/gtype"
    )

    func main() {
        b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
        type Student struct {
            Id     *gtype.Int
            Name   *gtype.String
            Scores *gtype.Interface
        }
        s := Student{}
        json.Unmarshal(b, &s)
        fmt.Println(s)
    }
    ```
    执行后，输出结果：
    ```
    {1 john [100,99,98]}
    ```
