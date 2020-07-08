
[TOC]


# 全局对象

`GF`框架封装了一些常用的数据类型以及对象，通过`g.*`方法获取。

> `g`是一个强耦合的模块，目的是为开发者在对频繁使用的类型/对象调用时提供便利。

**使用方式**：
```go
import "github.com/gogf/gf/frame/g"
```

## 数据类型

常用数据类型别名。

```go
// 泛型
type Var = gvar.Var

// 常用Map类型
type Map = map[string]interface{}
type MapAnyAny = map[interface{}]interface{}
type MapAnyStr = map[interface{}]string
type MapAnyInt = map[interface{}]int
type MapStrAny = map[string]interface{}
type MapStrStr = map[string]string
type MapStrInt = map[string]int
type MapIntAny = map[int]interface{}
type MapIntStr = map[int]string
type MapIntInt = map[int]int

// 常用Slice Map类型
type List = []Map
type ListAnyStr = []map[interface{}]string
type ListAnyInt = []map[interface{}]int
type ListStrAny = []map[string]interface{}
type ListStrStr = []map[string]string
type ListStrInt = []map[string]int
type ListIntAny = []map[int]interface{}
type ListIntStr = []map[int]string
type ListIntInt = []map[int]int

// 常用Slice类型
type Slice = []interface{}
type SliceAny = []interface{}
type SliceStr = []string
type SliceInt = []int

// 常用Slice类型(别名)
type Array = []interface{}
type ArrayAny = []interface{}
type ArrayStr = []string
type ArrayInt = []int
```

## 常用对象

常用对象往往通过`单例模式`进行管理，可以根据不同的单例名称获取对应的对象实例。并在对象初始化时会自动检索获取配置文件中的对应配置项，具体配置项请查看对应对象的章节介绍。

### `HTTP`客户端对象
```go
func Client() *ghttp.Client
```
创建一个新的`HTTP`客户端对象。

### (单例) 配置管理对象
```go
func Config(name...string) *gcfg.Config
```
别名：
```go
func Cfg(name ...string) *gcfg.Config
```
该单例对象将会自动读取默认配置文件`config.toml`并缓存，配置文件在外部被修改时将会自动刷新缓存。

### (单例) 日志管理对象
```go
func Log(name ...string) *glog.Logger
```
该单例对象将会自动读取默认配置文件中的`logger`配置项，并只会初始化一次日志对象。

### (单例) 模板引擎对象
```go
func View(name ...string) *gview.View
```
该单例对象将会自动读取默认配置文件中的`viewer`配置项，并只会初始化一次模板引擎对象。

### (单例) `WEB Server`
```go
func Server(name ...interface{}) *ghttp.Server
```
该单例对象将会自动读取默认配置文件中的`server`配置项，并只会初始化一次模板引擎对象。

### (单例) `TCP Server`
```go
func TcpServer(name ...interface{}) *gtcp.Server
```

### (单例) `UDP Server`
```go
func UdpServer(name ...interface{}) *gudp.Server
```

### (单例) 数据库`ORM`对象
```go
func DB(name ...string) *gdb.Db
```
该单例对象将会自动读取默认配置文件中的`database`配置项，并只会初始化一次模板引擎对象。

### (单例) `Redis`客户端对象
```go
func Redis(name ...string) *gredis.Redis
```
该单例对象将会自动读取默认配置文件中的`redis`配置项，并只会初始化一次模板引擎对象。

### (单例) 资源管理对象
```go
func Resource(name ...string) *gres.Resource
```
别名：
```go
func Res(name ...string) *gres.Resource
```

### (单例) 国际化管理对象
```go
func I18n(name ...string) *gi18n.Manager
```

