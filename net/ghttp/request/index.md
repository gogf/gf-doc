[TOC]


请求输入依靠 `ghttp.Request` 对象实现，`ghttp.Request`继承了底层的`http.Request`对象。`ghttp.Request`包含一个与当前请求对应的返回输出对象`Response`，用于数据的返回处理。

相关方法：
https://pkg.go.dev/github.com/gogf/gf/net/ghttp#Request



# 简要说明

可以看到`Request`对象的参数获取方法非常丰富，可以分为以下几类：
1. `Get*`: 常用方法，简化参数获取，`GetRequest*`的别名。
1. `GetQuery*`: 获取`GET`方式传递过来的参数，包括`Query String`及`Body`参数解析。
2. `GetForm*`: 获取表单方式传递过来的参数，表单方式提交的参数`Content-Type`往往为`application/x-www-form-urlencoded`, `application/form-data`, `multipart/form-data`, `multipart/mixed`等等。
3. `GetRequest*`: 获取客户端提交的参数，不区分提交方式。
1. `Get*Struct`: 将指定类型的请求参数绑定到指定的`struct`对象上，注意给定的参数为对象指针。绝大部分场景中往往使用`Parse`方法将请求数据转换为请求对象，具体详见后续章节。
4. `GetBody/GetBodyString`: 获取客户端提交的原始数据，该数据是客户端写入到`body`中的原始数据，与`HTTP Method`无关，例如客户端提交`JSON/XML`数据格式时可以通过该方法获取原始的提交数据。
5. `GetJson`: 自动将原始请求信息解析为`gjson.Json`对象指针返回，`gjson.Json`对象具体在【[gjson模块](encoding/gjson/index.md)】章节中介绍。
1. `Exit*`: 用于请求流程退出控制，详见本章后续说明；

# 提交方式

`GF`框架的参数获取不是通过`HTTP Method`来做区分，而是通过参数提交类型来区分。例如，分别通过`HTTP Method: POST、INPUT、DELETE`来提交表单参数，在服务端获取参数不是通过`GetPost`/`GetInput`/`GetDelete`的方式来获取，而是统一通过`GetForm`方法来获取表单参数，针对其他的`HTTP Method`也是如此。

在`GF`框架下，有以下几种提交类型：
1. `Router`: 路由参数，来源于路由规则匹配。
1. `Query`: `URL`中的`Query String`参数解析，如：`http://127.0.0.1/index?id=1&name=john` 中的`id=1&name=john`。
1. `Form`: 表单提交参数，最常见的提交方式，提交的`Content-Type`往往为：`application/x-www-form-urlencoded`、`multipart/form-data`、`multipart/mixed`。
1. `Body`: 原始提交内容，从`Body`中获取并解析得到的参数，`JSON`/`XML`请求往往使用这种方式提交。
1. `Custom`: 自定义参数，往往在服务端的中间件、服务函数中通过`SetParam/GetParam`方法管理。

# 参数类型

获取的参数方法可以对指定键名的数据进行自动类型转换，例如：`http://127.0.0.1:8199/?amount=19.66`，通过`GetQueryString`将会返回`19.66`的字符串类型，`GetQueryFloat32`/`GetQueryFloat64`将会分别返回`float32`和`float64`类型的数值`19.66`。但是，`GetQueryInt`/`GetQueryUint`将会返回`19`（如果参数为`float`类型的字符串，将会按照**向下取整**进行整型转换）。

变量类型的获取方法仅提供了常用类型的直接获取方法，如果有更多参数类型转换的需求，可以使用`Get*Var`参数获取方法，获得`*gvar.Var`变量再进行相应类型转换。例如，假如我们要获取一个`int8`类型的参数，我们可以这样`GetVar("id").Int8()`。


使用示例：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.Writeln(r.Get("amount"))
		r.Response.Writeln(r.GetInt("amount"))
		r.Response.Writeln(r.GetFloat32("amount"))
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后我们访问地址 http://127.0.0.1:8199/?amount=19.66 页面输出
```
19.66
19
19.66
```

# 参数优先级

我们考虑一种场景，当不同的提交方式中存在同名的参数名称会怎么样？在`GF`框架下，我们根据不同的获取方法，将会按照不同的优先级进行获取，优先级高的方式提交的参数将会优先覆盖其他方式的同名参数。优先级规则如下：
1. `Get*`及`GetRequset*`方法：`Router < Query < Body < Form < Custom`，也就是说自定义参数的优先级最高，其次是`Form`表单参数，再次是`Body`提交参数，以此类推。例如，`Query`和`Form`中都提交了同样名称的参数`id`，参数值分别为`1`和`2`，那么`Get("id")`/`GetForm("id")`将会返回`2`，而`GetQuery("id")`将会返回`1`。
1. `GetQuery*`方法：`Query > Body`，也就是说`query string`的参数将会覆盖`Body`中提交的同名参数。例如，`Query`和`Body`中都提交了同样名称的参数`id`，参数值分别为`1`和`2`，那么`Get("id")`将会返回`2`，而`GetQuery("id")`将会返回`1`。
1. `GetForm*`方法：由于该类型的方法仅用于获取`Form`表单参数，因此没什么优先级的差别。

使用示例：
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/input", func(r *ghttp.Request) {
		r.Response.Writeln(r.Get("amount"))
	})
	s.BindHandler("/query", func(r *ghttp.Request) {
		r.Response.Writeln(r.GetQuery("amount"))
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们通过`curl`工具进行测试：
```
$ curl -d "amount=1" -X POST "http://127.0.0.1:8199/input?amount=100"
1
$ curl -d "amount=1" -X POST "http://127.0.0.1:8199/query?amount=100"
100
```
可以看到，当我们访问`/input`路由时，该路由方法中采用了`Get*`方法获取`amount`参数，按照同名优先级的规则返回了`1`，即`body`中传递的参数。而当我们通过`/query`路由访问时，该路由方法中使用了`GetQuery*`方法获取`amount`参数，因此获取到的是`query string`参数中的`amount`值，返回了`100`。
