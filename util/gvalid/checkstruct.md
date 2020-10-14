[TOC]

# 多数据校验 - `CheckStruct`

`CheckStruct`的使用方式同`CheckMap`，除了第一个参数为`struct类型`的结构体对象（也可以是对象指针）。**但是需要注意的一个细节是，`struct`的属性会有`默认值`，在某些情况下会引起`required`规则的失效，因此根据实际情况配合多种规则一起校验会是一个比较严谨的做法。**

接口文档：
https://pkg.go.dev/github.com/gogf/gf/util/gvalid
```go
func CheckStruct(object interface{}, rules interface{}, msgs ...CustomMsg) *Error
```

### 示例1，使用`map`指定规则及提示信息
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gvalid"
)

func main() {
    type Object struct {
        Name string
        Age  int
    }
    rules := map[string]string {
        "Name" : "required|length:6,16",
        "Age"  : "between:18,30",
    }
    msgs  := map[string]interface{} {
        "Name" : map[string]string {
            "required" : "名称不能为空",
            "length"   : "名称长度为:min到:max个字符",
        },
        "Age"  : "年龄为18到30周岁",
    }
    obj := Object{Name : "john"}
    // 也可以是指针
    // obj := &Object{Name : "john"}
    if e := gvalid.CheckStruct(obj, rules,msgs); e != nil {
        g.Dump(e.Maps())
    }
}

/*
输出：
{
	"Age": {
		"between": "年龄为18到30周岁"
	},
	"Name": {
		"length": "名称长度为6到16个字符"
	}
}
*/
```

在以上示例中，`Age`属性由于默认值`0`的存在，因此会引起`required`规则的失效，因此这里没有使用`required`规则而是使用`between`规则来进行校验。

### 示例2，使用`gvalid tag`绑定规则及错误信息


使用`gvalid tag`设置的规则，其校验结果是顺序性的。

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gvalid"
)

type User struct {
    Uid   int    `v:"uid      @integer|min:1"`
    Name  string `v:"name     @required|length:6,30#请输入用户名称|用户名称长度非法"`
    Pass1 string `v:"password1@required|password3"`
    Pass2 string `v:"password2@required|password3|same:password1#||两次密码不一致，请重新输入"`
}

func main() {
    user := &User{
        Name : "john",
        Pass1: "Abc123!@#",
        Pass2: "123",
    }

    // 使用结构体定义的校验规则和错误提示进行校验
    if e := gvalid.CheckStruct(user, nil); e != nil {
        g.Dump(e.Maps())
    }

    // 自定义校验规则和错误提示，对定义的特定校验规则和错误提示进行覆盖
    rules := map[string]string {
        "uid" : "min:6",
    }
    msgs  := map[string]interface{} {
        "password2" : map[string]string {
            "password3" : "名称不能为空",
        },
    }
    if e := gvalid.CheckStruct(user, rules, msgs); e != nil {
        g.Dump(e.Maps())
    }
}
```
可以看到，我们可以对在`struct`定义时使用了`gvalid`的标签属性(`gvalid tag`)来绑定校验的规则及错误提示信息，规则如下：
```html
[属性别名@]校验规则[#错误提示]
```

可以看到，`CheckStruct`和`CheckMap`的`gvalid tag`规则是一样的，不过在字段的含义上有一点点小区别：
- `属性别名` 和 `错误提示` 为**非必需字段**，`校验规则` 是**必需字段**；
- `属性别名` 非必需字段，指定在校验中使用的对应`struct`属性的别名，同时校验成功后的`map`中的也将使用该别名返回，例如在处理请求表单时比较有用，因为表单的字段名称往往和`struct`的属性名称不一致；
- `错误提示` 非必需字段，表示自定义的错误提示信息，当规则校验时对默认的错误提示信息进行覆盖；

在此示例代码中，`same:password1`规则同使用`same:Pass2`规则是一样的效果。也就是说，在数据校验中，可以同时使用原有的`struct`属性名称，也可以使用别名。但是，返回的结果中只会使用别名返回，这也是别名最大的用途。

此外，在使用`CheckStruct`方法对`struct`对象进行校验时，也可以传递校验或者和错误提示参数，这个时候会覆盖`struct`在定义时绑定的对应参数。

以上示例执行后，输出结果为：
```json
{
	"name": {
		"length": "用户名称长度非法"
	},
	"password2": {
		"password3": "密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符",
		"same": "两次密码不一致，请重新输入"
	},
	"uid": {
		"min": "字段最小值为1"
	}
}
{
	"name": {
		"length": "用户名称长度非法"
	},
	"password2": {
		"password3": "名称不能为空",
		"same": "字段值不合法"
	},
	"uid": {
		"min": "字段最小值为6"
	}
}
```

### 示例3，属性递归校验

`gvalid.CheckStruct`支持递归校验，即如果属性也是结构体，并且结构体的属性带有`gvalid`标签，无论多深的递归层级，这些属性都将被根据设定的规则进行校验。

使用示例：
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gvalid"
)

func main() {
	type Pass struct {
		Pass1 string `valid:"password1@required|same:password2#请输入您的密码|您两次输入的密码不一致"`
		Pass2 string `valid:"password2@required|same:password1#请再次输入您的密码|您两次输入的密码不一致"`
	}
	type User struct {
		Id   int
		Name string `valid:"name@required#请输入您的姓名"`
		Pass Pass
	}
	user := &User{
		Name: "john",
		Pass: Pass{
			Pass1: "1",
			Pass2: "2",
		},
	}
	err := gvalid.CheckStruct(user, nil)
	g.Dump(err.Maps())
}
```
执行后，终端输出：
```json
{
	"password1": {
		"same": "您两次输入的密码不一致"
	},
	"password2": {
		"same": "您两次输入的密码不一致"
	}
}
```





# 可选校验规则

当给定的数据校验规则中不包含`required*`规则时，表示该规则不是一个必须规则，当属性值为`nil`或者`空字符串`时，将会忽略其校验。

## 示例1，空字符串
```go
type Params struct {
	Page      int    `v:"required|min:1         # page is required"`
	Size      int    `v:"required|between:1,100 # size is required"`
	ProjectId string `v:"between:1,10000        # project id must between :min, :max"`
}
obj := &Params{
	Page: 1,
	Size: 10,
}
err := gvalid.CheckStruct(obj, nil)
fmt.Println(err)
// Output:
// <nil>
```

## 示例2，空指针属性
```go
type Params struct {
	Page      int       `v:"required|min:1         # page is required"`
	Size      int       `v:"required|between:1,100 # size is required"`
	ProjectId *gvar.Var `v:"between:1,10000        # project id must between :min, :max"`
}
obj := &Params{
	Page: 1,
	Size: 10,
}
err := gvalid.CheckStruct(obj, nil)
fmt.Println(err)
// Output:
// <nil>
```

## 示例3，空整型属性
需要注意的是，如果键值为`0`或者`false`，将仍然会被校验。
```go
type Params struct {
	Page      int    `v:"required|min:1         # page is required"`
	Size      int    `v:"required|between:1,100 # size is required"`
	ProjectId int `v:"between:1,10000        # project id must between :min, :max"`
}
obj := &Params{
	Page: 1,
	Size: 10,
}
err := gvalid.CheckStruct(obj, nil)
fmt.Println(err)
// Output:
// project id must between 1, 10000
```




