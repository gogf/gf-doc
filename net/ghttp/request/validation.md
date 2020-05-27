
# 请求校验

`Request`对象支持非常完美的请求校验能力，通过给结构体属性绑定`v`标签即可。由于底层校验功能通过`gvalid`模块实现，更详细的校验规则和介绍请参考【[数据校验 - 结构体校验](util/gvalid/index.md)】章节。

## 示例1，基本使用

我们将之前的示例做下调整，增加`v`校验标签。


```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
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
在该示例中，我们定义了两个结构体：`RegisterReq`用于参数接收，`RegisterRes`用于数据返回。由于该接口返回的是`JSON`数据结构，可以看到，只有返回的结构体中存在`json`标签，而接收的结构体中只有`p`标签。因为`RegisterReq`仅用于参数接收，无需设置返回的`json`标签。

为了演示测试效果，这里在正常的返回结果`Data`属性中将`RegisterReq`对象返回，由于该对象没有绑定`json`标签，因此返回的`JSON`字段将会为其属性名称。

执行后，我们通过`curl`工具来测试一下：
```
$ curl "http://127.0.0.1:8199/register?name=john&password1=123456&password2=123456"
{"code":0,"error":"","data":{"Name":"john","Pass":"123456","Pass2":"123456"}}

$ curl "http://127.0.0.1:8199/register?name=john&password1=123456&password2=12345"
{"code":1,"error":"密码长度不够; 两次密码不一致","data":null}

$ curl "http://127.0.0.1:8199/register"
{"code":1,"error":"请输入账号; 账号长度为6到30位; 请输入密码; 密码长度不够; 请确认密码; 两次密码不一致","data":null}
```

## 示例2，校验错误处理

可以看到在以上示例中，当请求校验错误时，所有校验失败的错误都返回了，这样对于用户体验不是特别友好。当产生错误时，我们可以将校验错误转换为`*gvalid.Error`对象，随后可以通过灵活的方法控制错误的返回。

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
可以看到，当错误产生后，我们可以通过`err.(*gvalid.Error)`断言的方式判断错误是否为校验错误，如果是的话则返回第一条校验错误，而不是所有都返回。更详细的错误控制方法，请参考【[数据校验 - 校验结果](util/gvalid/error.md)】章节。

执行后，我们通过`curl`工具来测试一下：
```
$ curl "http://127.0.0.1:8199/register"
{"code":1,"error":"请输入账号","data":null}

$ curl "http://127.0.0.1:8199/register?name=john&password1=123456&password2=12345"
{"code":1,"error":"两次密码不一致","data":null}
```







