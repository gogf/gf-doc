# 缓冲控制

接口文档：https://godoc.org/github.com/gogf/gf/net/ghttp#Response

`Response`输出采用了缓冲控制，输出的内容预先写入到一块缓冲区，等待服务方法执行完毕后才真正地输出到客户端。该特性在提高执行效率同时为输出内容的控制提供了更高的灵活性。

相关方法：

```go
func (r *Response) Buffer() []byte
func (r *Response) BufferString() string
func (r *Response) BufferLength() int
func (r *Response) SetBuffer(data []byte)
func (r *Response) ClearBuffer()
func (r *Response) Output()
```
其中`Output`作用类似于`Flush`将缓冲区的数据输出到客户端并清空缓冲区。

举个例子：

我们通过后置中间件统一对返回的数据做处理，如果服务方法产生了异常，那么不能将敏感错误信息推送到客户端，而统一设置错误提示信息。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"net/http"
)

func MiddlewareErrorHandler(r *ghttp.Request) {
	r.Middleware.Next()
	if r.Response.Status >= http.StatusInternalServerError {
		r.Response.ClearBuffer()
		r.Response.Writeln("服务器居然开小差了，请稍后再试吧！")
	}
}

func main() {
	s := g.Server()
	s.Group("/api.v2", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareErrorHandler)
		group.ALL("/user/list", func(r *ghttp.Request) {
			panic("db error: sql is xxxxxxx")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
访问 http://127.0.0.1:8199/api.v2/user/list 可以看到，页面输出了：
```
服务器居然开小差了，请稍后再试吧！
```

