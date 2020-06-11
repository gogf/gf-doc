
# 版本选择算法

当项目中存在同一个第三方包依赖，并且依赖版本不一致时，`Go Modules`使用的“最小版本选择算法”(`The minimal version selection algorithm`: https://github.com/golang/go/wiki/Modules#version-selection )。

例如，如果您的模块依赖于具有`require D v1.0.0`的模块A，并且您的模块还依赖于具有`require D v1.1.1`的模块B，则最小版本选择将会选择D的`v1.1.1`版本用以构建（使用最高版本）。

> 请不要问我为什么这个算法名字叫“最小版本选择算法”，然而内容却是“最高版本选择算法”，若有纠结于此的同学欢迎向官方提issue：https://github.com/golang/go/issues

# 私有依赖管理

如果你可以通过`go.mod`完美地管理当前的项目包依赖，那么可以忽略该章节。如果你在处理项目的包依赖管理中遇到了问题，那么建议你继续阅读该章节，可以找到解决问题的灵感。

前面章节我们非常详细、图文并茂地介绍了基本的开发环境安装/配置、`Go Module`安装使用，在实际的项目开发中，你会发现更多的问题，常见的：
1. 虽然`GF`足够强大，但多数时候依赖的包不仅仅是`GF`，还包括一些额外的第三方包，特别是`golang.org`的包，需要自带梯子翻墙下载，即使本地方便处理，但是在自动部署系统上可能会稍麻烦；
1. 一些自研开发的第三方包，特别是一些业务依赖包，是不允许公开下载的（私有库），并且版本库也可能不支持`HTTPS`协议，因此无法使用`go get`或者`go.mod`进行下载和管理；
1. 等等；

如果你遇到了上面所提到的问题，我们建议的解决方案：通过`GOPRIVATE`的方式设置私有包有效域名。

例如以下命令行方式：
```shell
export GOPROXY=https://goproxy.cn
export GOPRIVATE=git.xxx.com
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main main.go
```

> 该特性需要`Go v1.13`以上版本支持。


在`Goland`中这么设置：


![](/images/WX20200611-185757@2x.png) 


