[TOC]



# 错误处理

由于这个问题问得比较多，因此专门为大家撰写了这一章节。


## 基本介绍

目前大多数`Golang`的第三方`WebServer`库均没有默认对`HTTP`请求处理过程中产生的异常进行捕获，轻者错误产生后无法记录到日志造成排查错困难，重则异常造成进程直接崩溃，服务不可用。

> 吐槽：由于作者公司产品也有在用`gin`，某几次生产事故便因为开发者不严谨的代码`panic`造成进程崩溃退出。解决方案是所有相关项目必须手动创建`recover`中间件进行捕获。

当你选择`gf`，你很幸运。为保证严谨性，执行过程中产生的`panic`是有被`Server`自动捕获的，产生`panic`时当前执行流程会立即中止，但是绝对不会影响进程直接崩溃。



## 自定义错误处理


`HTTP`执行流程中产生`panic`异常时，默认处理是记录到`Server`的日志文件中。当然，开发者也可以通过注册中间件方式手动捕获，然后自定义相关的错误处理。这一操作其实在中间件章节的示例中也有介绍，我们这里来再仔细说明下。

### 相关方法

异常的捕获我们通过`Request`对象中的`GetError`方法获取：
```go
// GetError returns the error occurs in the procedure of the request.
// It returns nil if there's no error.
func (r *Request) GetError() error
```
该方法往往使用在流程控制组件中，如后置中间件或者`HOOK`钩子方法中。

### 使用示例

我们这里使用一个全局的后置中间件来捕获异常，当异常产生后，捕获并写入到指定的日志文件中，返回固定友好的提示信息，避免敏感的报错信息暴露给调用端。

> 需要注意的是，即使开发者有自己捕获记录异常错误的日志，但是`Server`依旧会打印到`Server`自己的错误日志文件中。由开发者调用日志接口方法输出的日志属于业务日志（与业务相关），而`Server`管理的日志属于服务日志（类似于`ngin`的`error.log`）。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func MiddlewareErrorHandler(r *ghttp.Request) {
	r.Middleware.Next()
	if err := r.GetError(); err != nil {
		// 记录到自定义错误日志文件
		g.Log("exception").Error(err)
		//返回固定的友好信息
		r.Response.ClearBuffer()
		r.Response.Writeln("服务器居然开小差了，请稍后再试吧！")
	}
}

func main() {
	s := g.Server()
	s.Use(MiddlewareErrorHandler)
	s.Group("/api.v2", func(group *ghttp.RouterGroup) {
		group.ALL("/user/list", func(r *ghttp.Request) {
			panic("db error: sql is xxxxxxx")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们通过`curl`工具来试试吧：
```shell
$ curl -v "http://127.0.0.1:8199/api.v2/user/list"
> GET /api.v2/user/list HTTP/1.1
> Host: 127.0.0.1:8199
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 500 Internal Server Error
< Server: GF HTTP Server
< Date: Sun, 19 Jul 2020 07:44:30 GMT
< Content-Length: 52
< Content-Type: text/plain; charset=utf-8
<
服务器居然开小差了，请稍后再试吧！
```




























