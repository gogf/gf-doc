
# gredis

Redis客户端由`gredis`模块实现，底层采用了链接池设计。

> 目前`gredis`暂不支持官方`redis`集群功能，但支持阿里云`redis`集群功能。

**使用方式**：
```go
import "github.com/gogf/gf/database/gredis"
```

**接口文档**：
https://godoc.org/github.com/gogf/gf/database/gredis


`gredis`使用了连接池来进行`Redis`连接管理，通过`Config`配置对象或者`Set*`方法可以对连接池的属性进行管理，通过`Stats`方法可以获取连接池的统计信息。

我们最常用的是`Do/DoVar`方法，执行同步指令，通过向`Redis Server`发送对应的`Redis API`命令，来使用`Redis Server`的服务。`Do/Var`方法最大的特点是内部自行从连接池中获取连接对象，使用完毕后自动丢回连接池中，开发者无需手动调用`Close`方法，方便使用。

- Redis中文手册请参考：http://redisdoc.com/ 
- Redis官方命令请参考：https://redis.io/commands

> `gredis.Redis`客户端对象提供了一个`Close`方法，该方法用于关闭Redis客户端（同时关闭客户端的连接池），而不是连接对象，开发者基本不会用到，非高级玩家请不要使用。






