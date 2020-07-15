[TOC]

# Struct转换

项目中我们经常会遇到大量`struct`的使用，以及各种数据类型到`struct`的转换/赋值（特别是`json`/`xml`/各种协议编码转换）。为提高编码及项目维护效率，`gconv`模块为各位开发者带来了极大的福利，为数据解析提供了更高的灵活度。

`gconv`模块执行`struct`转换的方法仅有两个，定义如下：
```go
func Struct(params interface{}, pointer interface{}, mapping ...map[string]string) error
func StructDeep(params interface{}, pointer interface{}, mapping ...map[string]string) error
```

其中：
1. `params`为需要转换到`struct`的变量参数，可以为任意数据类型，常见的数据类型为`map`；
1. `pointer`为需要执行转的目标`struct`对象，这个参数必须为该`struct`的对象指针，转换成功后该对象的属性将会更新；
1. `mapping`为自定义的`map键名`到`strcut属性`之间的映射关系，此时`params`参数必须为map类型，否则该参数无意义；
1. `StructDeep`相比较于`Struct`方法，区别是支持递归转换，即会同时递归转换其属性中的结构体对象，特别用于转换带有继承结构的自定义`struct`，详见后续示例；

> 更多的`struct`相关转换方法请参考接口文档：https://godoc.org/github.com/gogf/gf/util/gconv

## 转换规则
`gconv`模块的`struct`转换特性非常强大，支持任意数据类型到`struct`属性的映射转换。在没有提供自定义`mapping`转换规则的情况下，默认的转换规则如下：
1. `struct`中需要匹配的属性必须为**`公开属性`**(首字母大写)；
2. 根据`params`类型的不同，逻辑会有不同：
    - `params`参数为`map`: 键名会自动按照 **`不区分大小写`** 且 **忽略`-/_/空格`符号** 的形式与`struct`属性进行匹配；
    - `params`参数为其他类型: 将会把该变量值与`struct`的第一个属性进行匹配；
    - 此外，如果`struct`的属性为复杂数据类型如`slice`,`map`,`strcut`那么会进行递归匹配赋值； 
3. 如果匹配成功，那么将键值赋值给属性，如果无法匹配，那么忽略该键值；

以下是几个`map`键名与`struct`属性名称的示例：
```html
map键名    struct属性     是否匹配
name       Name           match
Email      Email          match
nickname   NickName       match
NICKNAME   NickName       match
Nick-Name  NickName       match
nick_name  NickName       match
nick name  NickName       match
NickName   Nick_Name      match
Nick-name  Nick_Name      match
nick_name  Nick_Name      match
nick name  Nick_Name      match
```

## 自动创建对象

当给定的`pointer`参数类型为`**struct`时，`Struct`方法内部将会自动创建该`struct`对象，并修改传递变量指向的指针地址。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type User struct {
		Uid  int
		Name string
	}
	params := g.Map{
		"uid":  1,
		"name": "john",
	}
	var user *User
	if err := gconv.Struct(params, &user); err != nil {
		panic(err)
	}
	g.Dump(user)
}
```
执行后，输出结果为：
```json
{
	"Name": "john",
	"Uid": 1
}
```

## `Struct`递归转换

递归转换是指当`struct`对象包含子对象时，可以将`params`参数数据（第一个参数）同时递归地映射到其子对象上，常用于带有继承对象的`struct`上。

可以使用`StructDeep`方法实现递归转换。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type Ids struct {
		Id         int    `json:"id"`
		Uid        int    `json:"uid"`
	}
	type Base struct {
		Ids
		CreateTime string `json:"create_time"`
	}
	type User struct {
		Base
		Passport   string `json:"passport"`
		Password   string `json:"password"`
		Nickname   string `json:"nickname"`
	}
	data := g.Map{
		"id"          : 1,
		"uid"         : 100,
		"passport"    : "john",
		"password"    : "123456",
		"nickname"    : "John",
		"create_time" : "2019",
	}
	user := new(User)
	gconv.StructDeep(data, user)
	g.Dump(user)
}
```

执行后，终端输出结果为：
```json
{
	"Base": {
		"id": 1,
		"uid": 100,
		"create_time": "2019"
	},
	"nickname": "John",
	"passport": "john",
	"password": "123456"
}
```


## 示例1，基本使用
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gconv"
)

type User struct {
    Uid      int
    Name     string
    Site_Url string
    NickName string
    Pass1    string `gconv:"password1"`
    Pass2    string `gconv:"password2"`
}

func main() {
    user    := (*User)(nil)

    // 使用默认映射规则绑定属性值到对象
    user     = new(User)
    params1 := g.Map{
        "uid"       : 1,
        "Name"      : "john",
        "siteurl"   : "https://goframe.org",
        "nick_name" : "johng",
        "PASS1"     : "123",
        "PASS2"     : "456",
    }
    if err := gconv.Struct(params1, user); err == nil {
        g.Dump(user)
    }

    // 使用struct tag映射绑定属性值到对象
    user     = new(User)
    params2 := g.Map {
        "uid"       : 2,
        "name"      : "smith",
        "site-url"  : "https://goframe.org",
        "nick name" : "johng",
        "password1" : "111",
        "password2" : "222",
    }
    if err := gconv.Struct(params2, user); err == nil {
        g.Dump(user)
    }
}
```
可以看到，我们可以直接通过```Struct```方法将map按照默认规则绑定到struct上，也可以使用```struct tag```的方式进行灵活的设置。此外，```Struct```方法有第三个map参数，用于指定自定义的参数名称到属性名称的映射关系。

执行后，输出结果为：
```json
{
	"Uid": 1,
	"Name": "john",
	"Site_Url": "https://goframe.org",
	"NickName": "johng",
	"Pass1": "123",
	"Pass2": "456"
}
{
	"Uid": 2,
	"Name": "smith",
	"Site_Url": "https://goframe.org",
	"NickName": "johng",
	"Pass1": "111",
	"Pass2": "222"
}
```

## 示例2，复杂属性类型

### 1. 属性为`struct`/`*struct`

属性支持struct对象或者struct对象指针(目标为指针时，转换时会自动初始化)转换。

```go
package main

import (
    "github.com/gogf/gf/util/gconv"
    "github.com/gogf/gf/frame/g"
    "fmt"
)

func main() {
    type Score struct {
        Name   string
        Result int
    }
    type User1 struct {
        Scores Score
    }
    type User2 struct {
        Scores *Score
    }

    user1  := new(User1)
    user2  := new(User2)
	scores := g.Map{
		"Scores": g.Map{
			"Name":   "john",
			"Result": 100,
		},
	}

    if err := gconv.Struct(scores, user1); err != nil {
        fmt.Println(err)
    } else {
        g.Dump(user1)
    }
    if err := gconv.Struct(scores, user2); err != nil {
        fmt.Println(err)
    } else {
        g.Dump(user2)
    }
}
```

执行后，输出结果为：

```json
{
	"Scores": {
		"Name": "john",
		"Result": 100
	}
}
{
	"Scores": {
		"Name": "john",
		"Result": 100
	}
}
```



### 2. 属性为`slice`，数值为`slice`

```go
package main

import (
    "github.com/gogf/gf/util/gconv"
    "github.com/gogf/gf/frame/g"
    "fmt"
)

func main() {
    type Score struct {
        Name   string
        Result int
    }
    type User struct {
        Scores []Score
    }

    user   := new(User)
	scores := g.Map{
		"Scores": g.Slice{
			g.Map{
				"Name":   "john",
				"Result": 100,
			},
			g.Map{
				"Name":   "smith",
				"Result": 60,
			},
		},
	}

    // 嵌套struct转换，属性为slice类型，数值为slice map类型
    if err := gconv.Struct(scores, user); err != nil {
        fmt.Println(err)
    } else {
        g.Dump(user)
    }
}
```

执行后，输出结果为：

```json
{
	"Scores": [
		{
			"Name": "john",
			"Result": 100
		},
		{
			"Name": "smith",
			"Result": 60
		}
	]
}
```

### 3. 属性为`slice`，数值为`非slice`

```go
package main

import (
    "github.com/gogf/gf/util/gconv"
    "github.com/gogf/gf/frame/g"
    "fmt"
)

func main() {
    type Score struct {
        Name   string
        Result int
    }
    type User struct {
        Scores []Score
    }

    user   := new(User)
	scores := g.Map{
		"Scores": g.Map{
			"Name":   "john",
			"Result": 100,
		},
	}

    // 嵌套struct转换，属性为slice类型，数值为map类型
    if err := gconv.Struct(scores, user); err != nil {
        fmt.Println(err)
    } else {
        g.Dump(user)
    }
}
```
执行后，输出结果为：

```json
{
	"Scores": [
		{
			"Name": "john",
			"Result": 100
		}
	]
}
```

### 4. 属性为`[]*struct`

```go
package main

import (
    "github.com/gogf/gf/util/gconv"
    "github.com/gogf/gf/frame/g"
    "fmt"
)

func main() {
    type Score struct {
        Name   string
        Result int
    }
    type User struct {
        Scores []*Score
    }

    user   := new(User)
	scores := g.Map{
		"Scores": g.Slice{
			g.Map{
				"Name":   "john",
				"Result": 100,
			},
			g.Map{
				"Name":   "smith",
				"Result": 60,
			},
		},
	}

    // 嵌套struct转换，属性为slice类型，数值为slice map类型
    if err := gconv.Struct(scores, user); err != nil {
        fmt.Println(err)
    } else {
        g.Dump(user)
    }
}
```
执行后，输出结果为：
```json
{
	"Scores": [
		{
			"Name": "john",
			"Result": 100
		},
		{
			"Name": "smith",
			"Result": 60
		}
	]
}
```

