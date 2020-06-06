[TOC]

# Redis配置

## 配置文件

绝大部分情况下推荐使用`g.Redis`单例方式来操作redis。因此同样推荐使用配置文件来管理Redis配置，在`config.toml`中的配置示例如下：
```toml
# Redis数据库配置
[redis]
    default = "127.0.0.1:6379,0"
    cache   = "127.0.0.1:6379,1,123456?idleTimeout=600"
```
其中，Redis的配置格式为：`host:port[,db,pass?maxIdle=x&maxActive=x&idleTimeout=x&maxConnLifetime=x]`

各配置项说明如下：

|配置项名称|是否必须|默认值|说明
|---|---|---|---
| host            | 是 | -  | 地址
| port            | 是 | -  | 端口
| db              | 否 | 0  | 数据库
| pass            | 否 | -  | 授权密码
| maxIdle         | 否 | 0  | 允许限制的连接数(0表示不闲置)
| maxActive       | 否 | 0  | 最大连接数量限制(0表示不限制)
| idleTimeout     | 否 | 60 | 连接最大空闲时间(单位秒,不允许设置为0)
| maxConnLifetime | 否 | 60 | 连接最长存活时间(单位秒,不允许设置为0)

使用示例：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/util/gconv"
)

func main() {
    g.Redis().DoVar("SET", "k", "v")
    v, _ := g.Redis().DoVar("GET", "k")
    fmt.Println(v.String())
}
```
其中的`default`和`cache`分别表示配置分组名称，我们在程序中可以通过该名称获取对应配置的redis对象。不传递分组名称时，默认使用`redis.default`配置分组项)来获取对应配置的redis客户端单例对象。
执行后，输出结果为：
```html
v
```

## 配置方法

`gredis`提供了全局的分组配置功能，相关配置管理方法如下：
```go
func SetConfig(config Config, name ...string)
func GetConfig(name ...string) (config Config, ok bool)
func RemoveConfig(name ...string)
func ClearConfig()
```
其中`name`参数为分组名称，即为通过分组来对配置对象进行管理，我们可以为不同的配置对象设置不同的分组名称，随后我们可以通过`Instance`单例方法获取`redis`客户端操作对象单例。
```go
func Instance(name ...string) *Redis
```

使用示例：
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/database/gredis"
	"github.com/gogf/gf/util/gconv"
)

var (
	config = gredis.Config{
		Host : "127.0.0.1",
		Port : 6379,
		Db   : 1,
	}
)

func main() {
	group := "test"
	gredis.SetConfig(config, group)

	redis := gredis.Instance(group)
	defer redis.Close()

	_, err := redis.Do("SET", "k", "v")
	if err != nil {
		panic(err)
	}

	r, err := redis.Do("GET", "k")
	if err != nil {
		panic(err)
	}
	fmt.Println(gconv.String(r))
}
```