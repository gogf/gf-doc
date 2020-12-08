
[TOC]

`gf`框架自建了非常强大的路由功能，提供了比任何同类框架更加出色的路由特性，支持流行的命名匹配规则、模糊匹配规则及字段匹配规则，并提供了优秀的优先级管理机制。

# 一个简单示例

在真正开启本章的核心内容之前，我们先来看一个简单的动态路由使用示例：
```go
package main

import (
    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/frame/g"
)

func main() {
    s := g.Server()
    s.BindHandler("/:name", func(r *ghttp.Request){
       r.Response.Writeln(r.Router.Uri)
    })
    s.BindHandler("/:name/update", func(r *ghttp.Request){
        r.Response.Writeln(r.Router.Uri)
    })
    s.BindHandler("/:name/:action", func(r *ghttp.Request){
        r.Response.Writeln(r.Router.Uri)
    })
    s.BindHandler("/:name/*any", func(r *ghttp.Request){
       r.Response.Writeln(r.Router.Uri)
    })
    s.BindHandler("/user/list/{field}.html", func(r *ghttp.Request){
        r.Response.Writeln(r.Router.Uri)
    })
    s.SetPort(8199)
    s.Run()
}
```
以上示例中展示了`gf`框架支持的三种模糊匹配路由规则，`:name`、`*any`、`{field}`分别表示命名匹配规则、模糊匹配规则及字段匹配规则。不同的规则中使用`/`符号来划分层级，路由检索采用深度优先算法，层级越深的规则优先级也会越高。我们运行以上示例，通过访问几个URL来看看效果：
```html
                  URL                               结果
http://127.0.0.1:8199/user/list/2.html      /user/list/{field}.html
http://127.0.0.1:8199/user/update           /:name/update
http://127.0.0.1:8199/user/info             /:name/:action
http://127.0.0.1:8199/user                  /:name/*any
```
在这个示例中我们也可以看到，由于优先级的限制，路由规则`/:name`会被`/:name/*any`规则覆盖，将会无法被匹配到，所以在分配路由规则的时候，需要进行统一规划和管理，避免类似情况的产生。


#  路由注册规则

## 路由注册参数

我们来看一下之前一直使用的`BindHandler`的原型：
```go
func (s *Server) BindHandler(pattern string, handler HandlerFunc) error
```
该方法是路由注册的最基础方法，其中的`pattern`为路由注册规则字符串，在其他路由注册方法中也会使用到，参数格式如下：
```
[HTTPMethod:]路由规则[@域名]
```
其中`HTTPMethod`（支持的Method：`GET,PUT,POST,DELETE,PATCH,HEAD,CONNECT,OPTIONS,TRACE`）和`@域名`为非必需参数，一般来说直接给定路由规则参数即可，`BindHandler`会自动绑定**所有的**请求方式，如果给定`HTTPMethod`，那么路由规则仅会在该请求方式下有效。`@域名`可以指定生效的域名名称，那么该路由规则仅会在该域名下生效。

> `BindHandler`是最原生的路由注册方法，在大部分场景中，我们通常使用 **分组路由** 方式来管理理由，详见后续章节。

我们来看一个例子：
```go
package main

import (
    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/frame/g"
)

func main() {
    s := g.Server()
    // 该路由规则仅会在GET请求下有效
    s.BindHandler("GET:/{table}/list/{page}.html", func(r *ghttp.Request){
        r.Response.WriteJson(r.Router)
    })
    // 该路由规则仅会在GET请求及localhost域名下有效
    s.BindHandler("GET:/order/info/{order_id}@localhost", func(r *ghttp.Request){
        r.Response.WriteJson(r.Router)
    })
    // 该路由规则仅会在DELETE请求下有效
    s.BindHandler("DELETE:/comment/{id}", func(r *ghttp.Request){
        r.Response.WriteJson(r.Router)
    })
    s.SetPort(8199)
    s.Run()
}
```
其中返回的参数`r.Router`是当前匹配的路由规则信息，访问当该方法的时候，服务端会输出当前匹配的路由规则信息。执行后，我们在终端使用`curl`命令进行测试：
```shell
$ curl -XGET http://127.0.0.1:8199/order/list/1.html
{"Domain":"default","Method":"GET","Priority":3,"Uri":"/{table}/list/{page}.html"}

$ curl -XGET http://127.0.0.1:8199/order/info/1
Not Found

$ curl -XGET http://localhost:8199/order/info/1
{"Domain":"localhost","Method":"GET","Priority":3,"Uri":"/order/info/{order_id}"}

$ curl -XDELETE http://127.0.0.1:8199/comment/1000
{"Domain":"default","Method":"DELETE","Priority":2,"Uri":"/comment/{id}"}

$ curl -XGET http://127.0.0.1:8199/comment/1000
Not Found
```
值得说明的是，在大多数场景下，我们很少直接在路由规则中使用```@域名```这样的规则来限定路由注册的域名，而是使用```ghttp.Server.Domain(domains string)```方法来获得指定域名列表的管理对象，随后使用该域名对象进行路由注册，域名对象即可实现对指定域名的绑定操作。


## 动态路由规则

动态路由规则分为三种：**命名匹配规则**、**模糊匹配规则**和**字段匹配规则**。动态路由的底层数据结构是由层级`哈希表`和`双向链表`构建的`路由树`，层级哈希表便于高效率地层级匹配URI；数据链表用于优先级控制，同一层级的路由规则按照优先级进行排序，优先级高的规则排在链表头。底层的路由规则与请求URI的匹配计算采用的是正则表达式，并充分使用了缓存机制，执行效率十分高效。

所有匹配到的参数都将会以`Router`参数的形式传递给业务层，可以通过`ghttp.Request`对象的以下方法获取：
```go
func (r *Request) GetRouterValue(key string, def ...interface{}) interface{}
func (r *Request) GetRouterVar(key string, def ...interface{}) *gvar.Var
func (r *Request) GetRouterString(key string, def ...interface{}) string
```
也可以使用`ghttp.Request.Get*`方式进行获取。

### 精准匹配规则

精准匹配规则即**未使用任何动态规则**的规则，如：`user`、`order`、`info`等等这种**确定名称**的规则。在大多数场景下，精准匹配规则会和动态规则一起使用来进行路由注册(例如：`/:name/list`，其中层级1`:name`为命名匹配规则，层级2`list`是精准匹配规则)。

### 命名匹配规则

使用`:name`方式进行匹配(`name`为自定义的匹配名称)，对URI指定层级的参数进行命名匹配（类似正则`([^/]+)`，该URI层级必须有值），对应匹配参数会被解析为Router参数并传递给注册的服务接口使用。

匹配示例1：
```html
rule: /user/:user

/user/john                match
/user/you                 match
/user/john/profile        no match
/user/                    no match
```
匹配示例2：
```html
rule: /:name/action

/john/name                no match
/john/action              match
/smith/info               no match
/smith/info/age           no match
/smith/action             match
```
匹配示例3：
```html
rule: /:name/:action

/john/name                match
/john/info                match
/smith/info               match
/smith/info/age           no match
/smith/action/del         no match
```

### 模糊匹配规则

使用`*any`方式进行匹配(`any`为自定义的匹配名称)，对URI指定位置之后的参数进行模糊匹配（类似正则`(.*)`，该URI层级可以为空），并将匹配参数解析为Router参数并传递给注册的服务接口使用。

匹配示例1：
```html
rule: /src/*path

/src/                     match
/src/somefile.go          match
/src/subdir/somefile.go   match
/user/                    no match
/user/john                no match
```
匹配示例2：
```html
rule: /src/*path/:action

/src/                     no match
/src/somefile.go          match
/src/somefile.go/del      match
/src/subdir/file.go/del   match
```
匹配示例3：
```html
rule: /src/*path/show

/src/                     no match
/src/somefile.go          no match
/src/somefile.go/del      no match
/src/somefile.go/show     match
/src/subdir/file.go/show  match
/src/show                 match
```

### 字段匹配规则

使用`{field}`方式进行匹配(`field`为自定义的匹配名称)，可对URI任意位置的参数进行截取匹配（类似正则`([\w\.\-]+)`，该URI层级必须有值，并且可以在同一层级进行多个字段匹配），并将匹配参数解析为Router参数并传递给注册的服务接口使用。

匹配示例1：
```html
rule: /order/list/{page}.php

/order/list/1.php          match
/order/list/666.php        match
/order/list/2.php5         no match
/order/list/1              no match
/order/list                no match
```
匹配示例2：
```html
rule: /db-{table}/{id}

/db-user/1                     match
/db-user/2                     match
/db/user/1                     no match
/db-order/100                  match
/database-order/100            no match
```
匹配示例3：
```html
rule: /{obj}-{act}/*param

/user-delete/10                match
/order-update/20               match
/log-list                      match
/log/list/1                    no match
/comment/delete/10             no match
```

### 动态路由示例
```go
package main

import (
    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/frame/g"
)

func main() {
    s := g.Server()
    // 一个简单的分页路由示例
    s.BindHandler("/user/list/{page}.html", func(r *ghttp.Request){
        r.Response.Writeln(r.Get("page"))
    })
    // {xxx} 规则与 :xxx 规则混合使用
    s.BindHandler("/{object}/:attr/{act}.php", func(r *ghttp.Request){
        r.Response.Writeln(r.Get("object"))
        r.Response.Writeln(r.Get("attr"))
        r.Response.Writeln(r.Get("act"))
    })
    // 多种模糊匹配规则混合使用
    s.BindHandler("/{class}-{course}/:name/*act", func(r *ghttp.Request){
        r.Response.Writeln(r.Get("class"))
        r.Response.Writeln(r.Get("course"))
        r.Response.Writeln(r.Get("name"))
        r.Response.Writeln(r.Get("act"))
    })
    s.SetPort(8199)
    s.Run()
}
```
执行后，我们可以通过`curl`命令或者浏览器访问的方式进行测试，以下为测试结果：
```shell
$ curl -XGET http://127.0.0.1:8199/user/list/1.html
1

$ curl -XGET http://127.0.0.1:8199/user/info/save.php
user
info
save

$ curl -XGET http://127.0.0.1:8199/class3-math/john/score
class3
math
john
score
```


# 路由优先级控制

优先级控制按照深度优先策略，最主要的几点因素：

1. **层级越深的规则优先级越高**；
2. **同一层级下，精准匹配优先级高于模糊匹配**；
3. **同一层级下，模糊匹配优先级：字段匹配 > 命名匹配 > 模糊匹配**；

我们来看示例（左边的规则优先级比右边高）：
```html
/:name                   >            /*any
/user/name               >            /user/:action
/:name/info              >            /:name/:action
/:name/:action           >            /:name/*action
/:name/{action}          >            /:name/:action
/src/path/del            >            /src/path
/src/path/del            >            /src/path/:action
/src/path/*any           >            /src/path
```

<!--
# 路由检索算法介绍

1. 首先将URI按照`/`符号进行拆分为多个层级，按照层级进行检索；
2. 路由进行匹配检索的时候将会从根部开始通过哈希表一层一层往叶子节点进行匹配；
3. 如果该层级节点中如果有数据链表，那么保存该链表到一个链表数组中，以便后续回溯；
3. 如果该层级节点数据值能够和检索值精准匹配，那么继续遍历该匹配字段的下一层级；
4. 如果该层级无法精准匹配，并且存在匹配的模糊规则，那么继续遍历该模糊规则的下一层级；
5. 当层级检索到达叶子后，再将匹配到的数据链表按照从叶子节点往根部节点进行回溯检索；
6. 回溯检索流程：

	1). 从链表数组末尾开始遍历；

	2). 遍历数组对应位置的数据链表；

	3). 由于数据链表优先级是从前往后递减，因此遍历链表的时候是按照优先级遍历；

	4). 如果当前链表没有检索到匹配规则，那么重复2) - 3)，直到把链表数组遍历完毕；

	5). 匹配到规则立即返回，否则全部遍历完以后返回失败；

7. 检索完毕，如果存在结果则将结果与当前请求`Method`、`URI`、`Domain`进行绑定缓存，下一次直接读取缓存结果；
-->