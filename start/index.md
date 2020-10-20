[TOC]


为方便小伙伴们快速使用`GF`框架创建一个基本的项目，我们这里以简单的开发示例，使用`GF`框架来创建一个简单的`API`服务项目，该项目实现以下几个示例接口：
1. 用户注册
1. 用户登录
1. 用户注销
1. 登录状态判断
1. 账号/昵称唯一性校验

包含以下功能特性：
1. 允许跨域访问
1. 包含权限校验

# 源码仓库

该示例项目的源代码仓库位于: https://github.com/gogf/gf-demos 

由于文档的粘贴的代码可能会滞后于仓库代码，建议通过下载该仓库代码查看示例。后续章节主要对其中的主要代码做介绍。

各位可以通过[《开始运行》](start/buildrun.md)章节末尾示例的`curl`命令行方式进行测试，也可以通过`/document/postman`目录的`postman`配置进行测试。

# 项目结构

如果是`Package`源码包项目，开发者可随意定义目录结构。

如果是业务类型项目，`GoFrame`官方推荐的`Go`项目目录结构如下：
```
/
├── app
│   ├── api
│   ├── model
│   └── service
├── boot
├── config
├── docker
├── document
├── i18n
├── library
├── packed
├── public
├── router
├── template
├── vendor
├── Dockerfile
├── go.mod
└── main.go
```
|目录/文件名称   | 说明 | 描述
|---|---|---
|`app`           | 业务逻辑层 | 所有的业务逻辑存放目录。
| - `api`        | 业务接口   | 接收/解析用户输入参数的入口/接口层。
| - `model`      | 数据模型   | 数据管理层，仅用于操作管理数据，如数据库操作。
| - `service`    | 逻辑封装   | 业务逻辑封装层，实现特定的业务需求，可供不同的包调用。
|`boot`          | 初始化包   | 用于项目初始化参数设置，往往作为`main.go`中第一个被`import`的包。
|`config`        | 配置管理   | 所有的配置文件存放目录。
|`docker`        | 镜像文件   | Docker镜像相关依赖文件，脚本文件等等。
|`document`      | 项目文档   | Documentation项目文档，如: 设计文档、帮助文档等等。
|`i18n`          | I18N国际化 | I18N国际化配置文件目录。
|`library`       | 公共库包   | 公共的功能封装包，往往不包含业务需求实现。
|`packed`        | 打包目录   | 将资源文件打包的`Go`文件存放在这里，`boot`包初始化时会自动调用。
|`public`        | 静态目录   | 仅有该目录下的文件才能对外提供静态服务访问。
|`router`        | 路由注册   | 用于路由统一的注册管理。
|`template`      | 模板文件   | MVC模板文件存放的目录。
|`vendor`        | 第三方包   | 第三方依赖包存放目录（可选, 未来会被淘汰）。
|`Dockerfile`    | 镜像描述 | 云原生时代用于编译生成Docker镜像的描述文件。
|`go.mod`        | 依赖管理   | 使用`Go Module`包管理的依赖描述文件。
|`main.go`       | 入口文件   | 程序入口文件。

在实践中，小伙伴们可以根据实际情况增删目录。

注意：如果需要提供静态服务，那么所有静态文件都需要存放到`public`目录下，仅有该目录下的静态文件才能被外部直接访问。不推荐将程序当前运行目录加入到静态服务中。

> 项目创建推荐使用`GF`工具链`gf init`命令，具体请参考【[GF工具链](toolchain/cli.md)】章节。

# 设计模式

`GF`官方推荐的项目目录结构采用的是`MVC`设计模式，准确的说是"`MVCS`"模式。

> 开发框架有别于一般的类库，其中几个比较重要的区别在于，框架提供了基础的开发模块、完善的工具链、项目结构及开发规范约束，这也是项目工程化的基础前提。`GF`框架的使用是灵活的，所有的"最佳实践"都是推荐及引导性的。

## 控制器

控制器负责接收并响应客户端的输入与输出，包括对输入参数的过滤、转换、校验，对输出数据结构的维护，并调用`service`实现业务逻辑处理。

控制器代码位于`/app/api`。

## 逻辑封装

业务逻辑是需要封装的，特别是一些可复用的业务逻辑，并被控制器调用实现业务逻辑处理。

逻辑封装的代码位于`/app/service`。

## 数据模型

数据模型负责维护所有的数据操作，特别是对数据库的访问控制。数据模型往往是被`service`调用，不推荐通过控制器直接访问数据模型。

数据模型的代码位于`/app/model`。

> 推荐使用`GF`工具链强大的`gf gen model`命令自动生成数据库模型，具体请参考【[GF工具链](toolchain/cli.md)】章节。

## 模板解析

模板解析是可选的，在实践中往往可以采用`MVVM`的模式，例如使用`vue`/`react`等框架实现模板解析。如果使用经典的模板解析，可以通过`GF`框架强大的模板引擎实现模板解析。

模板文件的存放于`/template`。


# 数据库设计
我们创建一个简单的用户表。

https://github.com/gogf/gf-demos/blob/master/document/sql/create.sql
```sql
CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `passport` varchar(45) NOT NULL COMMENT '用户账号',
  `password` varchar(45) NOT NULL COMMENT '用户密码',
  `nickname` varchar(45) NOT NULL COMMENT '用户昵称',
  `create_at` datetime DEFAULT NULL COMMENT '创建时间',
  `update_at` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

为简化示例项目的接口实现复杂度，这里的`password`没有做任何加密处理，明文存放密码数据。











