[TOC]

# gvar

`通用动态变量`，支持各种内置的数据类型转换，可以作为`interface{}`类型的替代数据类型，并且该类型支持并发安全开关。

> Tips: 框架同时提供了`g.Var`的数据类型，其实也是`gvar.Var`数据类型的别名。

**使用场景**：

使用`interface{}`的场景，各种不固定数据类型格式，或者需要对变量进行频繁的数据类型转换的场景。

**使用方式**：
```go
import "github.com/gogf/gf/container/gvar"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/container/gvar

 

## 基本使用

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "fmt"
)

func main() {
    var v g.Var

    v.Set("123")

    fmt.Println(v.Val())

    // 基本类型转换
    fmt.Println(v.Int())
    fmt.Println(v.Uint())
    fmt.Println(v.Float64())

    // slice转换
    fmt.Println(v.Ints())
    fmt.Println(v.Floats())
    fmt.Println(v.Strings())

    // struct转换
    type Score struct {
        Value int
    }
    s := new(Score)
    v.Struct(s)
    fmt.Println(s)
}
```

执行后，输出结果为：

```html
123
123
123
123
[123]
[123]
[123]
&{123}
```

## `JSON`序列化/反序列
`gvar.Var`容器实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/frame/g"
    )

    func main() {
        type Student struct {
            Id     *g.Var
            Name   *g.Var
            Scores *g.Var
        }
        s := Student{
            Id:     g.NewVar(1),
            Name:   g.NewVar("john"),
            Scores: g.NewVar([]int{100, 99, 98}),
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
        "github.com/gogf/gf/frame/g"
    )

    func main() {
        b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
        type Student struct {
            Id     *g.Var
            Name   *g.Var
            Scores *g.Var
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
