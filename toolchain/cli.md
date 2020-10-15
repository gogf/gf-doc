[TOC]


# `gf tool chain`

从`GF v1.9`版本开始提供了`gf`命令行开发辅助工具，将会随着框架发展不断完善，作为未来框架发展的一个重要组成部分，工具开源项目地址：https://github.com/gogf/gf-cli

我们推荐通过下载安装预编译的二进制使用。工具安装成功后，可以通过`gf`或者`gf -h`查看所有支持的命令。复杂的命令可以通过`gf help COMMAND`或者`gf COMMAND -h`查看更详细的使用帮助信息，例如：`gf help gen`、`gf gen -h`。

> 工具运行初始化时将会自动打开`Go Modules`特性并设置反向代理地址为 `https://goproxy.cn`。

> 当前帮助文档以`gf cli v0.7.5`版本为例进行简单的介绍，详细的介绍信息请查看命令行帮助信息。本章内容信息可能会有滞后，最新的具体详细介绍请查看工具帮助信息。

```
$ gf
USAGE
    gf COMMAND [ARGUMENT] [OPTION]

COMMAND
    env        show current Golang environment variables
    get        install or update GF to system in default...
    gen        automatically generate go files for ORM models...
    mod        extra features for go modules...
    run        running go codes with hot-compiled-like feature...
    init       initialize an empty GF project at current working directory...
    help       show more information about a specified command
    pack       packing any file/directory to a resource file, or a go file...
    build      cross-building go project for lots of platforms...
    docker     create a docker image for current GF project...
    swagger    swagger feature for current project...
    update     update current gf binary to latest one (might need root/admin permission)
    install    install gf binary to system (might need root/admin permission)
    version    show current binary version info

OPTION
    -y         all yes for all command without prompt ask
    -?,-h      show this help or detail for specified command
    -v,-i      show version information

ADDITIONAL
    Use 'gf help COMMAND' or 'gf COMMAND -h' for detail about a command, which has '...'
    in the tail of their comments.
```


## `install` 工具安装

使用方式：`./gf install`

该命令往往是在`gf`命令行工具下载到本地后执行（注意执行权限），用于将`gf`命令安装到系统环境变量默认支持的目录路径中，以便于在系统任何的地方直接可以使用`gf`工具。

> 部分系统需要管理员权限支持。

> 如果是`MacOS`下使用`zsh`的小伙伴可能会遇到别名冲突问题，可以通过`alias gf=gf`来解决，运行一次之后`gf`工具会自动修改`profile`中的别名设置，用户重新登录就好了。

## `version` 工具版本查看

使用方式：
- `gf -v`
- `gf -i`
- `gf version`

用以查看当前`gf`命令行工具编译时的版本信息。例如：
```
$ gf -v
GoFrame CLI Tool v0.7.5, https://goframe.org
Built Detail:
  Go Version:  go1.13.5
  GF Version:  v1.13.0
  Git Commit:  0ab2a4a9e753f4f5fae1e96ecf40cae94895d494
  Built Time:  2020-06-05 23:30:39
```

## `init` 项目初始化

使用方式：`gf init [NAME]`

我们可以使用`init`命令在当前目录生成一个示例的`GF`空框架项目，并可给定可选的项目名称参数。生成的项目目录结构仅供参考，根据业务项目具体情况可自行调整。

> `GF`框架开发推荐统一使用官方的`go module`特性进行依赖包管理，因此空项目根目录下也有一个`go.mod`文件。

## `build` 交叉编译

使用方式：`gf build FILE [OPTION]`

仅限于交叉编译使用到`GF`框架的项目，支持绝大部分常见系统的直接交叉编译。并且支持配置文件管理编译选项、嵌入编译时变量。使用`gf build`的项目将会默认嵌入以下变量（参考`gf -v`）：
1. 当前`Go`版本。
1. 当前`GF`版本。
1. 当前编译时间。
1. 当前`Git Commit`（如果存在）。

编译配置文件选项示例（默认读取`config.toml`）：
```toml
[compiler]
    name     = "my-app"
    version  = "1.0.0"
    arch     = "386,amd64"
    system   = "linux,windows,darwin"
    output   = ""
    path     = "./bin"
	extra    = ""
    # 自定义编译时内置变量
    [compiler.VarMap]
        author = "john"
        email  = "john@goframe.org"
```
配置选项的释义同命令行同名选项。

> 编译时的内置变量可以在运行时通过`gbuild`包获取。

## `gen` 代码生成命令

使用方式：`gf gen model [PATH] [OPTION]`

`gen`命令用以自动化从数据库直接生成模型文件。可以参考`gf-demos`示例项目中的模型文件即是通过该命令生成的：https://github.com/gogf/gf-demos/tree/master/app/model

模型生成采用了`Active Record`设计模式。该命令将会根据数据表名生成对应的目录，该目录名称即数据表包名。目录下自动生成3个文件：
1. `数据表名.go` 自定义文件，开发者可以自由定义填充的代码文件，仅会生成一次，每一次模型生成不会覆盖。
1. `数据表名_entity.go` 表结构文件，根据数据表结构生成的结构体定义文件，包含字段注释。数据表在外部变更后，可使用`gen`命令重复生成更新该文件。
1. `数据表名_model.go` 表模型文件，为数据表提供了许多便捷的`CURD`操作方法，并可直接查询返回该表的结构体对象。数据表在外部变更后，可使用`gen`命令重复生成更新该文件。

> 数据表模型生成支持的数据库类型为：`MySQL/MariaDB`、`PostgreSQL`、`SQLite`、`SQLServer`。目前暂不支持`Oracle`，若有需求请联系作者。

## `run` 热编译（自动编译）
使用方式：`gf run FILE`

由于`Go`是不支持热编译特性的，每一次代码变更后都要重新手动停止、编译、运行代码文件。`run`命令也不是实现热编译功能，而是提供了自动编译功能，当开发者修改了项目中的`go`文件时，该命令将会自动编译当前程序，并停止原有程序，运行新版的程序。

> `run`命令会递归监控**当前运行目录**的所有`go`文件变化来实现自动编译。

## `get` 依赖包下载

使用方式：`gf get PACKAGE`

`gf get`命令和`go get`命令类似，内部自动提供了代理设置功能，并智能识别并设置最快的下载代理地址。


## `docker` 镜像编译

使用方式：`gf docker [FILE] [OPTIONS]`

自动编译并生成`docker`镜像。非必需`FILE`参数为编译文件路径，默认为`main.go`。非必需参数`OPTIONS`为`docker build`命令相同参数及选项。

## `update` 工具更新

使用方式：`gf update`

该命令用以检测`gf`命令行工具的版本，并自动执行版本更新。

> 部分系统需要管理员权限支持。如果更新失败，请手动重新下载更新。


## `pack` 二进制打包

使用方式：`gf pack SRC DST`

该命令用以将任意的文件打包为二进制文件，或者`Go`代码文件，可将任意文件打包后随着可执行文件一同发布。此外，在`build`命令中支持打包+编译一步进行，具体请查看`build`命令帮助信息。

## `mod` 包管理扩展功能

使用方式：`gf mod path`

该命令用于将当前`go modules`包拷贝到`GOPATH`中，以便使用原始的`GOPATH`方式开发，往往为了方便调试本地开发的依赖包。


## `help` 命令行帮助

使用方式：
- `gf -h`
- `gf -?`
- `gf help [COMMAND]`  
- `gf [COMMAND] -h`

任何不懂的，就用`help`看看吧。















