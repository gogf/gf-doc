
[TOC]

`GF`框架提供了完善的`Server`日志管理功能，包括`access log`以及`error log`，推荐使用配置文件的方式统一配置管理。

# 日志配置

## 配置对象

请查看API文档：

https://godoc.org/github.com/gogf/gf/net/ghttp#ServerConfig

## 配置属性
日志相关配置属性如下：
```go
Logger            *glog.Logger      // Logger for server.
LogPath           string            // Directory for storing logging files.
LogStdout         bool              // Printing logging content to stdout.
ErrorStack        bool              // Logging stack information when error.
ErrorLogEnabled   bool              // Enable error logging files.
ErrorLogPattern   string            // Error log file pattern like: error-{Ymd}.log
AccessLogEnabled  bool              // Enable access logging files.
AccessLogPattern  string            // Error log file pattern like: access-{Ymd}.log
```
简要说明：
1. 默认情况下，日志不会输出到文件中，而是直接打印到终端。默认情况下的`access`日志终端输出是关闭的，仅有`error`日志默认开启。
1. 所有的选项均可通过`Server.Set*`方法设置，大部分选项可以通过`Server.Get*`方法获取。
1. `Logger`是一个自定义的日志管理对象，开发者也可以传递一个完整的日志管理对象，忽略其他日志选项配置。
1. `LogPath`属性用于设置日志目录，只有在设置了日志目录的情况下才会输出日志到日志文件中。
1. `ErrorLogPattern`及`AccessLogPattern`用于配置日志文件名称格式，默认为`error-{Ymd}.log`及`access-{Ymd}.log`，例如：`error-20191212.log`, `access-20191212.log`。
1. 其他配置选项说明请参考注释和API文档。

## 配置文件

官方推荐使用配置文件的方式来管理服务配置以及日志日志配置。
一个参考的日志配置内容示例（以`toml`格式为例）：
```toml
[server]
	LogPath           = "/var/log/gf-demos/server"
    LogStdout         = false            
    ErrorStack        = true              
    ErrorLogEnabled   = true              
    ErrorLogPattern   = "error.{Ymd}.log"            
    AccessLogEnabled  = true        
    AccessLogPattern  = "access.{Ymd}.log"    
```

当`Server`启动时将会自动去读取默认配置文件`config.toml`中的`server`节点配置。

## 配置方法

日志的配置也可以通过`Server`对象的`Set*`方法来进行配置，参考【[配置管理](net/ghttp/config.md)】章节。



# 日志格式

配置文件的方式比较简单，这里不再示例说明。以下示例通过配置方法的方式进行对`Server`进行配置。

## 请求日志


一个示例的请求日志：
```shell
2018-04-20 18:11:57.344 200 "GET http 127.0.0.1:8199 /log/access HTTP/1.1" 0.120, 127.0.0.1, "", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/53.0.2785.143 Chrome/53.0.2785.143 Safari/537.36"
```
日志格式: `访问时间(精确到毫秒)` `HTTP状态码` "`请求方式` `请求前缀` `请求地址` `请求协议`" `执行时间(毫秒)` `客户端IP` `来源URL` `UserAgent`。其中，`请求前缀`为`http`或者`https`，`请求协议`往往为`HTTP/1.0`或者`HTTP/1.1`。

> 注意，日志中记录的`执行时间`单位为`毫秒`，绝大多数情况下看到的时间几乎都是`0.xxx`毫秒时间，也就是说执行时间都是微秒级不到1毫秒。

## 错误日志

一个示例的错误日志：

```shell
2019-12-20 20:10:56.484 [ERRO] 500, "GET http 127.0.0.1:8199 /log/error HTTP/1.1" 0.210, 127.0.0.1, "", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
Stack:
1. OMG
   1).  main.main.func1
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/net/ghttp/server/log/log_error.go:10

```
错误信息会打印出对应错误产生的堆栈信息（堆栈信息中不包含框架内部调用信息），以便于错误定位以及开发者分析问题原因。

> `WebServer`产生的任何`panic`错误都将会被自动捕获到错误日志中，因此对于业务端程序来讲，无论是在控制器中、业务封装层、数据模型中，如果产生了错误想要直接退出业务请求处理，直接`panic`即可。特别是对于绝大多数`golang`风格的方法`error`返回值判断，处理将会变得异常的简单。如果是业务程序自身定义的方法`error`甚至可不用返回，可以根据业务场景选择直接`panic`。

# 自定义日志处理

开发者可以自定义处理`Server`的请求日志，方法有两种：
1. 可以通过日志配置项传递自定义的`*glog.Logger`对象。
1. 可以通过中间件来统一捕获处理，参考【[中间件设计](net/ghttp/router/middleware.md)】章节。

# `Server`日志与业务日志

这是一个`FAQ`。

我们需要注意的是，这里提到的日志都是`Server`的日志，类似于`nginx`, `apache`, `tomcat`等等一系列`Web Server`服务的日志，只允许`Server`输出内容，开发者无法往`Server`的日志文件中写入日志内容，并且日志类型和格式是完全固定的。

`GF`框架也提供了日志模块，由`glog`包实现，开发者通过`glog`包打印的日志属于业务日志，程序业务代码可以决定输出什么内容，输出到哪里，输出格式是什么样等。并且常用`g.Log()`方法来输出业务日志，该方法支持自动读取配置文件中的`[logger]`配置项。具体请参考【[日志管理](os/glog/index.md)】章节。