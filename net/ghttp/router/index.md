
[TOC]

`WebServer`提供服务需要回调函数/方法/对象/控制器的支持，`ghttp`包支持多种路由注册模式，为开发者提供非常强大和灵活的接口功能。路由注册是整个`WebServer`s最核心的部分，也是`gf`框架中最精心设计的一个模块。

接口文档：
https://pkg.go.dev/github.com/gogf/gf/net/ghttp

# g与ghttp包

在随后的章节示例代码中，我们将会看到频繁的`g.Server()`及`ghttp.GetServer()`混用，其实它们获取的都是同一个`WebServer`单例对象指针。其中`ghttp.GetServer()`是`ghttp`包**原生单例**`WebServer`对象指针获取方法。而`g.Server()`是框架通用**对象管理器**提供的方法，框架`g.*`对象管理器封装了常用的一些对象方法，具体请参看后续的【[对象管理](frame/g/index.md)】章节。虽然这种方式模块间耦合性比较高，但使用简便，也是推荐的使用方式。


# 路由注册介绍

本章开始之前，我们再来看一下本手册开头的`Hello World`程序：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/", func(r *ghttp.Request) {
        r.Response.Write("哈喽世界！")
    })
    s.Run()
}
```
其中，使用`BindHandler`方法进行路由注册的方式叫做`回调函数注册`，是最简单的一种路由注册方式。通过给指定的`WebServer`上对应的URI注册一个可执行的方法，当客户端访问该URI时，`WebServer`便自动调用对应注册的回调函数来执行处理。在回调函数注册中，每个注册函数都会有一个`ghttp.Request`对象参数指针，表示每个请求特定的独立的请求处理对象，回调函数可以通过该对象获取提交请求参数，也可以返回处理结果数据。

## 路由注册方式比较

在详细讲解每一种注册方式之前，先看看每种注册方式各自的优缺点，以便在不同的业务场景中选择更适合的注册方式。如果暂时不理解这个表格没有关系，可以在了解完每一种注册方式之后再回过头来看，也许会更清晰。

|  注册方式        |  使用难度  |  安全系数  |  执行性能  | 内存消耗  |
| ---                   |     ---       | --- | --- | ---|
|  回调函数注册  |  高  |  低  |  高  | 低 |
|  执行对象注册  |  中  |  中  |  中  | 中 |
|  ~~控制器方式注册~~ |  ~~低~~  |   ~~高~~ |  ~~低~~  |  ~~高~~ |

比较指标说明：
1. 使用难度：主要指对于执行流程以及数据管理维护的复杂度；
1. 安全系数：主要指在异步多协程下的内部数据安全管理维护；
1. 执行性能：执行性能，相对比较的结果；
1. 内存消耗：内存消耗，相对比较的结果；

**为何要设计三种注册方式？**


以上的三种方式对应的是三种`使用习惯`：

1. **回调函数注册**：这种方式的路由注册不限制给定的回调函数是一个对象方法还是包方法，它仅仅需要一个函数的内存地址指针即可，使用比较灵活。
1. **执行对象注册**：使用同一个实例化的`struct`对象进行路由注册，多个请求都将会由该对象进行管理，请求与请求之间也可能会通过该对象的属性共享变量。大多数的`go web`库也仅提供这种方式，大部分场景下也推荐使用这种方式进行注册。
1. **控制器方式注册**： 类似于`PHP`的执行机制，每一个请求都对应一个全新的控制器对象，变量的管理维护比较安全。由于内部在运行时使用了反射机制，因此对于性能没有过高要求的场景可以考虑这种方式。未来无特别需求不推荐继续使用该注册方式。

具体详细的介绍及使用请继续查看后续对应的章节。



## 路由注册接口文档

服务的注册仅有以下7个方法：

```go
func (s *Server) BindHandler(pattern string, handler HandlerFunc) error

func (s *Server) BindObject(pattern string, obj interface{}, methods ...string) error
func (s *Server) BindObjectMethod(pattern string, obj interface{}, method string) error
func (s *Server) BindObjectRest(pattern string, obj interface{}) error

func (s *Server) BindController(pattern string, c Controller, methods ...string) error
func (s *Server) BindControllerMethod(pattern string, c Controller, method string) error
func (s *Server) BindControllerRest(pattern string, c Controller) error
```

简要说明：
1. 路由注册使用的`pattern`参数格式请参考【[路由规则](net/ghttp/router/rules.md)】章节。
1. 其中，`BindHandler`方法用于特定的回调函数注册，`BindObject*`方法用于对象相关注册，`BindController*`方法用于控制器相关注册。
1. 需要注意的是，控制器注册`BindController*`系列方法第二个参数为控制器接口，给定的参数必须实现`ghttp.Controller`接口。简便的做法是用户自定义的控制器直接继承`gmvc.Controller`基类即可，`gmvc.Controller`已经实现了对应的接口方法。


## 域名路由注册方法

我们可以通过`Server`对象的以下方法获得`Domain`对象：
```go
func (s *Server) Domain(domains string) *Domain
```
其中`domains`参数支持多个域名绑定，使用`,`号分隔。

路由注册支持绑定域名，以下是对应的**接口文档**：
```go
func (d *Domain) BindHandler(pattern string, handler HandlerFunc) error

func (d *Domain) BindObject(pattern string, obj interface{}, methods ...string) error
func (d *Domain) BindObjectMethod(pattern string, obj interface{}, method string) error
func (d *Domain) BindObjectRest(pattern string, obj interface{}) error

func (d *Domain) BindController(pattern string, c Controller, methods ...string) error
func (d *Domain) BindControllerMethod(pattern string, c Controller, method string) error
func (d *Domain) BindControllerRest(pattern string, c Controller) error
```
各项参数和`Server`的路由注册对应方法一致，只不过在`Domain`对象的底层会自动将方法绑定到指定的域名列表中，只有对应的域名才能提供访问。

我们来看一个简单的例子，我们将前面的`Hello World`程序改成如下形式：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    d := g.Server().Domain("localhost")
    d.BindHandler("/", func(r *ghttp.Request) {
        r.Response.Write("Hello World!")
    })
    g.Server().Run()
}
```
我们再次使用 http://127.0.0.1/ 进行访问，发现WebServer返回`404`，为什么呢？因为该程序中的回调函数只注册到了`localhost`域名中，因此只能通过 http://localhost/ 访问，其他域名自然无法访问。

