
[TOC]


# 接口文档

https://godoc.org/github.com/gogf/gf/os/gsession


任何时候都可以通过`ghttp.Request`获取`Session`对象，因为`Cookie`和`Session`都是和请求会话相关，因此都属于`Request`的成员对象，并对外公开。`gf`框架的`Session`处理是存放在内存中的，因此处理效率非常高，默认过期时间是`24小时`。

此外，需要说明的是，`Session`的操作是支持`并发安全`的，这也是框架在对`Session`的设计上不采用直接以`map`的形式操作数据的原因。

在任何时候，我们都可以通过`ghttp.Request`对象来修改和获取`Session`的全局相关属性。

# `gsession`模块

从`GF v1.9`开始，`Session`的管理功能单独解耦出来作为一个单独的模块，由`gsession`实现，并已完美整合到了`ghttp.Server`中。由于该模块是解耦独立的，因此可以使用到更多不同的场景中，例如：`TCP`通信、`gRPC`接口服务等等。

`gsession`模块中有三个对象/接口：
1. `gsession.Manager`：管理`Session`对象、`Storage`持久化存储对象、以及过期时间控制。
1. `gsession.Session`：单个`Session`会话管理对象，用于`Session`参数的增删查改等数据管理操作。
1. `gsession.Storage`：这是一个接口定义，用于`Session`对象的持久化存储、读取、存活更新操作，开发者可基于该接口实现自定义的持久化存储特性。该接口定义如下：
    ```go
    type Storage interface {
        // Get retrieves session value with given key.
        // It returns nil if the key does not exist in the session.
        Get(key string) interface{}
        // GetMap retrieves all key-value pairs as map from storage.
        GetMap() map[string]interface{}
        // GetSize retrieves the size of key-value pairs from storage.
        GetSize(id string) int

        // Set sets key-value session pair to the storage.
        Set(key string, value interface{}) error
        // SetMap batch sets key-value session pairs with map to the storage.
        SetMap(data map[string]interface{}) error

        // Remove deletes key with its value from storage.
        Remove(key string) error
        // RemoveAll deletes all key-value pairs from storage.
        RemoveAll() error

        // GetSession returns the session data bytes for given session id.
        GetSession(id string) map[string]interface{}
        // SetSession updates the content for session id.
        // Note that the parameter <content> is the serialized bytes for session map.
        SetSession(id string, data map[string]interface{}) error

        // UpdateTTL updates the TTL for specified session id.
        UpdateTTL(id string) error
    }
    ```
    
## 默认存储方式
默认的`Session`存储使用了`内存+文件`的方式。具体原理为：
1. `Session`的数据操作完全基于内存；
1. 使用`gcache`进程缓存模块控制数据过期；
1. 使用文件存储持久化存储管理`Session`数据；
1. 当且仅有当`Session`被标记为`dirty`时才会执行`Session`序列化并执行文件持久化存储；
1. 当且仅当内存中的`Session`不存在时，才会从文件存储中反序列化恢复`Session`数据到内存中；
1. 序列化/反序列化使用的是`json.Marshal/UnMarshal`；

从原理可知，当`Session`为读多写少的场景中，`Session`的数据操作非常高效。

> 有个注意的细节，由于文件存储涉及到文件操作，为便于降低`IO`开销并提高`Session`操作性能，并不是每一次`Session`请求结束后都即时地去更新存储的文件`TTL`时间。只有当涉及到写入操作时（被标记为`dirty`），这种情况下，每一次`Session`请求结束后会即时地更新对应`Session`存储文件的TTL时间；而针对于读取请求，将会每隔`一分钟`更新前一分钟内读取操作对应的`Session`文件`TTL`时间，以便于`Session`自动续活。


## 使用示例1，基本使用

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/session", func(r *ghttp.Request) {
        id := r.Session.GetInt("id")
        r.Session.Set("id", id + 1)
        r.Response.Write("id:", id)
    })
    s.SetPort(8199)
    s.Run()
}
```
启动`main.go`，访问`http://127.0.0.1:8199/session`，刷新几次页面，可以看到页面输出的id值在不断递增。


## 使用示例2，持久化示例

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/gtime"
	"time"
)

func main() {
	s := g.Server()
	s.SetSessionMaxAge(10)
	s.BindHandler("/set", func(r *ghttp.Request) {
		r.Session.Set("time", gtime.Second())
		r.Response.Write("ok")
	})
	s.BindHandler("/get", func(r *ghttp.Request) {
		r.Response.WriteJson(r.Session.Map())
	})
	s.BindHandler("/clear", func(r *ghttp.Request) {
		r.Session.Clear()
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，
1. 首先，访问 `http://127.0.0.1:8199/set` 设置一个Session变量；
1. 随后，访问 `http://127.0.0.1:8199/get` 可以看到该Session变量已经设置并成功获取；
1. 接着，我们停止程序，并重新启动，再次访问 `http://127.0.0.1:8199/get` ，可以看到Session变量已经从文件存储中恢复并成功获取；
1. 等待10秒后，再次访问 `http://127.0.0.1:8199/get` 可以看到已经无法获取该Session，因为该Session已经过期；


