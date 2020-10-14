
# 分组路由

`GF`框架支持分组路由的注册方式，可以给分组路由指定一个`prefix`前缀（也可以直接给定`/`前缀，表示注册在根路由下），在该分组下的所有路由注册都将注册在该路由前缀下。分组路由注册方式也是推荐的路由注册方式。

**接口文档**：

https://godoc.org/github.com/gogf/gf/net/ghttp#RouterGroup

```go
// 创建分组路由
func (s *Server) Group(prefix string, groups ...func(g *RouterGroup)) *RouterGroup 
func (d *Domain) Group(prefix string, groups ...func(g *RouterGroup)) *RouterGroup 

// 注册Method路由
func (g *RouterGroup) ALL(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) GET(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) PUT(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) POST(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) DELETE(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) PATCH(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) HEAD(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) CONNECT(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) OPTIONS(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) TRACE(pattern string, object interface{}, params...interface{})

// 中间件绑定
func (g *RouterGroup) Middleware(handlers ...HandlerFunc) *RouterGroup

// REST路由
func (g *RouterGroup) REST(pattern string, object interface{})

// 批量注册
func (g *RouterGroup) Bind(items []GroupItem)
```

其中：
1. `Group`方法用户创建一个分组路由对象，并且支持在指定域名对象上创建。
1. 以`HTTP Method`命名的方法用以绑定指定的`HTTP Method`路由；其中`ALL`方法用于注册所有的`HTTP Method`到指定的函数/对象/控制器上；`REST`方法用户注册`RESTful`风格的路由，需给定一个执行对象或者控制器对象。
1. `Middleware`方法用于绑定一个或多个中间件到当前分组的路由上，具体详见中间件章节。
1. `Bind`方法用于批量路由注册，每一个路由注册项为`Slice`类型的参数，且参数数量应该`>=3`个，具体使用请见后续示例。

我们来看一个简单的示例：
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	group := s.Group("/api")
	group.ALL("/all", func(r *ghttp.Request) {
		r.Response.Write("all")
	})
	group.GET("/get", func(r *ghttp.Request) {
		r.Response.Write("get")
	})
	group.POST("/post", func(r *ghttp.Request) {
		r.Response.Write("post")
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，终端打印出路由表如下：
```
  SERVER  | DOMAIN  | ADDRESS | METHOD |   ROUTE   |     HANDLER     | MIDDLEWARE  
|---------|---------|---------|--------|-----------|-----------------|------------|
  default | default | :8199   | ALL    | /api/all  | main.main.func1 |             
|---------|---------|---------|--------|-----------|-----------------|------------|
  default | default | :8199   | GET    | /api/get  | main.main.func2 |             
|---------|---------|---------|--------|-----------|-----------------|------------|
  default | default | :8199   | POST   | /api/post | main.main.func3 |             
|---------|---------|---------|--------|-----------|-----------------|------------|
```
其中，`/api/get`仅允许`GET`方式访问，`/api/post`仅允许`POST`方式访问，`/api/all`允许所有的方式访问。

我们使用`curl`工具来测试一下：
1. `/api/get`
    ```shell
    $ curl http://127.0.0.1:8199/api/get
    get
    $ curl -X POST http://127.0.0.1:8199/api/get
    Not Found
    ```
1. `/api/post`
    ```shell
    $ curl http://127.0.0.1:8199/api/post
    Not Found
    $ curl -X POST http://127.0.0.1:8199/api/post
    post
    ```
1. `/api/all`
    ```shell
    $ curl http://127.0.0.1:8199/api/all
    all
    $ curl -X POST http://127.0.0.1:8199/api/all
    all
    $ curl -X DELETE http://127.0.0.1:8199/api/all
    all
    $ curl -X OPTIONS http://127.0.0.1:8199/api/all
    all
    ```