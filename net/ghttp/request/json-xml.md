[TOC]

# `JSON/XML`解析

从`GF v1.11`版本开始，`Request`对象提供了对客户端提交的`JSON/XML`数据格式的原生支持，为开发者提供了更便捷的数据获取特性，以进一步提高开发效率。

## 示例1，简单示例

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.Writef("name: %v, pass: %v", r.Get("name"), r.Get("pass"))
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们通过`curl`工具提交数据来测试一下：

1. `Query`数据格式
    ```
    $ curl "http://127.0.0.1:8199/?name=john&pass=123"
    name: john, pass: 123
    ```

1. `Form`表单提交
    ```
    $ curl -d "name=john&pass=123" "http://127.0.0.1:8199/"
    name: john, pass: 123
    ```

1. `JSON`数据格式
    ```
    $ curl -d '{"name":"john","pass":"123"}' "http://127.0.0.1:8199/"
    name: john, pass: 123
    ```
1. `XML`数据格式
    ```
    $ curl -d '<?xml version="1.0" encoding="UTF-8"?><doc><name>john</name><pass>123</pass></doc>' "http://127.0.0.1:8199/"
    name: john, pass: 123

    $ curl -d '<doc><name>john</name><pass>123</pass></doc>' "http://127.0.0.1:8199/"
    name: john, pass: 123
    ```

## 示例2，对象转换及校验
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/util/gvalid"
)

type RegisterReq struct {
	Name  string `p:"username"  v:"required|length:6,30#请输入账号|账号长度为:min到:max位"`
	Pass  string `p:"password1" v:"required|length:6,30#请输入密码|密码长度不够"`
	Pass2 string `p:"password2" v:"required|length:6,30|same:password1#请确认密码|密码长度不够|两次密码不一致"`
}

type RegisterRes struct {
	Code  int         `json:"code"`
	Error string      `json:"error"`
	Data  interface{} `json:"data"`
}

func main() {
	s := g.Server()
	s.BindHandler("/register", func(r *ghttp.Request) {
		var req *RegisterReq
		if err := r.Parse(&req); err != nil {
			// Validation error.
			if v, ok := err.(*gvalid.Error); ok {
				r.Response.WriteJsonExit(RegisterRes{
					Code:  1,
					Error: v.FirstString(),
				})
			}
			// Other error.
			r.Response.WriteJsonExit(RegisterRes{
				Code:  1,
				Error: err.Error(),
			})
		}
		// ...
		r.Response.WriteJsonExit(RegisterRes{
			Data: req,
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们通过`curl`工具提交数据来测试一下：

1. `JSON`数据格式
    ```
    $ curl -d '{"username":"johngcn","password1":"123456","password2":"123456"}' "http://127.0.0.1:8199/register"
    {"code":0,"error":"","data":{"Name":"johngcn","Pass":"123456","Pass2":"123456"}}

    $ curl -d '{"username":"johngcn","password1":"123456","password2":"123"}' "http://127.0.0.1:8199/register"
    {"code":1,"error":"两次密码不一致","data":null}
    ```

    可以看到，我们提交的`JSON`内容也被`Parse`方法智能地转换为了结构体对象。

1. `XML`数据格式
    ```
    $ curl -d '<?xml version="1.0" encoding="UTF-8"?><doc><username>johngcn</username><password1>123456</password1><password2>123456</password2></doc>' "http://127.0.0.1:8199/register"
    {"code":0,"error":"","data":{"Name":"johngcn","Pass":"123456","Pass2":"123456"}}

    $ curl -d '<?xml version="1.0" encoding="UTF-8"?><doc><username>johngcn</username><password1>123456</password1><password2>123</password2></doc>' "http://127.0.0.1:8199/register"
    {"code":1,"error":"两次密码不一致","data":null}
    ```
    可以看到，我们提交的`XML`内容也被`Parse`方法智能地转换为了结构体对象。

