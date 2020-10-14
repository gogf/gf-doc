[TOC]

# Map转换

`gconv.Map`支持将任意的`map`或`struct`/`*struct`类型转换为常用的 `map[string]interface{}` 类型。当转换参数为`struct`/`*struct`类型时，支持自动识别`struct`的 `c/gconv/json` 标签，并且可以通过`Map`方法的第二个参数`tags`指定自定义的转换标签，以及多个标签解析的优先级。如果转换失败，返回`nil`。

> 属性标签：当转换`struct`/`*struct`类型时，支持 `c/gconv/json` 属性标签，也支持 `-`及`omitempty` 标签属性。当使用 `-` 标签属性时，表示该属性不执行转换；当使用 `omitempty` 标签属性时，表示当属性为空时（空指针`nil`, 数字`0`, 字符串`""`, 空数组`[]`等）不执行转换。具体请查看随后示例。

常用转换方法：
```go
func Map(value interface{}, tags ...string) map[string]interface{}
func MapDeep(value interface{}, tags ...string) map[string]interface{}
```
其中，`MapDeep`支持递归转换，即会递归转换属性中的`struct`/`*struct`对象。

> 更多的`map`相关转换方法请参考接口文档：https://pkg.go.dev/github.com/gogf/gf/util/gconv

### 示例1，基本示例
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type User struct {
		Uid  int    `c:"uid"`
		Name string `c:"name"`
	}
	// 对象
	g.Dump(gconv.Map(User{
		Uid:  1,
		Name: "john",
	}))
	// 对象指针
	g.Dump(gconv.Map(&User{
		Uid:  1,
		Name: "john",
	}))

	// 任意map类型
	g.Dump(gconv.Map(map[int]int{
		100: 10000,
	}))
}
```
执行后，终端输出：
```json
{
    "name": "john",
    "uid": 1
}

{
    "name": "john",
    "uid": 1
}

{
    "100": 10000
}
```

### 示例2，属性标签

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type User struct {
		Uid      int
		Name     string `c:"-"`
		NickName string `c:"nickname, omitempty"`
		Pass1    string `c:"password1"`
		Pass2    string `c:"password2"`
	}
	user := User{
		Uid:   100,
		Name:  "john",
		Pass1: "123",
		Pass2: "456",
	}
	g.Dump(gconv.Map(user))
}
```
执行后，终端输出：
```json
{
    "Uid": 100,
    "password1": "123",
    "password2": "456"
}
```
### 示例3，自定义标签

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type User struct {
		Id   int    `c:"uid"`
		Name string `my-tag:"nick-name" c:"name"`
	}
	user := &User{
		Id:   1,
		Name: "john",
	}
	g.Dump(gconv.Map(user, "my-tag"))
}
```
执行后，输出结果为：
```json
{
	"nick-name": "john",
	"uid": 1
}
```




### 示例4，递归转换

当参数为`map`/`struct`/`*struct`类型时，如果键值/属性为一个对象（或者对象指针）时，`Map`方法将会将对象转换为结果的一个键值。我们可以使用`MapDeep`方法递归转换参数的子对象。

使用示例：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type Base struct {
		Id   int    `c:"id"`
		Date string `c:"date"`
	}
	type User struct {
		Base     `c:"base"`
		Passport string `c:"passport"`
		Password string `c:"password"`
		Nickname string `c:"nickname"`
	}
	user := new(User)
	user.Id = 1
	user.Date = "2019"
	user.Nickname = "John"
	user.Passport = "johng"
	user.Password = "123456"
	g.Dump(gconv.MapDeep(user))
}
```
执行后，终端输出结果为：
```json
{
    "base": {
        "date": "2019",
        "id": 1
    },
    "nickname": "John",
    "passport": "johng",
    "password": "123456"
}
```