[TOC]


日志组件是`GF`框架核心的组件之一，当然也支持非常方便的配置管理功能。

# 单例对象

日志组件支持单例模式，使用`g.Log(单例名称)`获取不同的单例日志管理对象。提供单例对象的目的在于针对不同业务场景可以使用不同配置的日志管理对象。我们推荐使用`g.Log()`方法获取单例对象来进行日志操作，该方法内部会自动读取配置文件并初始化单例对象，该初始化操作仅会执行一次。

# 配置文件（推荐）

日志组件支持配置文件，当使用`g.Log(单例名称)`获取`Logger`单例对象时，将会自动通过默认的配置管理对象获取对应的`Logger`配置。默认情况下会读取`logger.单例名称`配置项，当该配置项不存在时，将会读取默认的`logger`配置项。配置项请参考配置对象结构定义：https://godoc.org/github.com/gogf/gf/os/glog#Config

## 示例1，默认配置项
```toml
[logger]
    path   = "/var/log"
    level  = "all"
    stdout = false
```
随后可以使用`g.Log()`获取默认的单例对象时自动获取并设置该配置。

## 示例2，多个配置项
多个`Logger`的配置示例：
```toml
[logger]
    path   = "/var/log"
    level  = "all"
    stdout = false
    [logger.logger1]
        path   = "/var/log/logger1"
        level  = "dev"
        stdout = false
    [logger.logger2]
        path   = "/var/log/logger2"
        level  = "prod"
        stdout = true
```
我们可以通过单例对象名称获取对应配置的`Logger`单例对象：
```go
// 对应 logger.logger1 配置项
l1 := g.Log("logger1")
// 对应 logger.logger2 配置项
l2 := g.Log("logger2")
// 对应默认配置项 logger
l3 := g.Log("none")
// 对应默认配置项 logger
l4 := g.Log()
```


# 配置方法（高级）

配置方法用于模块化使用`glog`时由开发者自己进行配置管理。

方法列表： https://godoc.org/github.com/gogf/gf/os/glog

简要说明：
1. 可以通过`SetConfig`及`SetConfigWithMap`来设置。
1. 也可以使用`Logger`对象的`Set*`方法进行特定配置的设置。
1. 主要注意的是，配置项在`Logger`对象执行日志输出之前设置，避免并发安全问题。

我们可以使用`SetConfigWithMap`方法通过`Key-Value`键值对来设置/修改`Logger`的特定配置，其余的配置使用默认配置即可。其中`Key`的名称即是`Config`这个`struct`中的属性名称，并且不区分大小写，单词间也支持使用`-`/`_`/`空格`符号连接，具体可参考【[gconv.Struct转换规则](util/gconv/struct.md)】章节。

简单示例：
```go
logger := glog.New()
logger.SetConfigWithMap(g.Map{
    "path":     "/var/log",
    "level":    "all",
    "stdout":   false,
    "StStatus": 0,
})
logger.Print("test")
```
其中`StStatus`表示是否开启堆栈打印，设置为`0`表示关闭。键名也可以使用`stStatus`, `st-status`, `st_status`, `St Status`，其他配置属性以此类推。

其中，`level`配置项使用字符串配置，按照日志级别支持以下配置：`DEBU` < `INFO` < `NOTI` < `WARN` < `ERRO` < `CRIT`，也支持`ALL`, `DEV`, `PROD`常见部署模式配置名称。`level`配置项字符串不区分大小写。关于日志级别的详细介绍请查看【[日志级别](os/glog/level.md)】章节。








