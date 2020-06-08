[TOC]

# `Context`上下文

`glog`支持标准库`context.Context`接口对象中上下文变量的自动读取打印。

## 键名配置

我们推荐使用配置文件来对上下文中的键名进行配置，例如：

```toml
# 日志组件配置
[logger]
    Path    = "/var/log/my-app"
    Level   = "all"
    Stdout  = false
    CtxKeys = ["Trace-Id", "Span-Id"]
```
其中`CtxKeys`用于配置需要从`context.Context`接口对象中读取并输出的键名。

## 日志输出

在输出日志的时候，需要通过`Ctx`链式操作方法指定输出的`context.Context`接口对象，例如：
```go
ctx := context.WithValue(context.Background(), "Trace-Id", "123456789")
g.Log().Ctx(ctx).Error("runtime error")

// May Output:
// 2020-06-08 20:17:03.630 [ERRO] {Trace-Id: 123456789} runtime error
// Stack:
// ...
```
我们推荐使用`g.Log()`方法获取单例对象，该方法内部会自动读取配置文件并初始化单例对象，该初始化操作仅会执行一次。



