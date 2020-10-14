
[TOC]

# 基本介绍

`gf`框架的`ORM`功能由`gdb`模块实现，用于常用关系型数据库的`ORM`操作。其最大的特色在于同时支持`map`和`struct`两种方式操作数据库。`gdb`默认情况下使用的是`map`数据类型作为基础的数据表记录载体，开发者无需预先定义数据表记录`struct`便可直接对数据表记录执行各种操作。这样的设计赋予了开发者更高的灵活度和简便性。

> `gdb`数据库引擎底层采用了**链接池设计**，当链接不再使用时会自动关闭，因此链接对象不用的时候不需要显式使用`Close`方法关闭数据库连接。这也是`gdb`数据库模块人性化设计的地方，方便开发者使用数据库而无需手动维护大量的数据库链接对象。

**注意：为提高数据库操作安全性，在`ORM`操作中不建议直接将参数拼接成`SQL`字符串执行，建议尽量使用预处理的方式（充分使用`?`占位符）来传递SQL参数。`gdb`的底层实现中均采用的是预处理的方式处理开发者传递的参数，以充分保证数据库操作安全性。**

接口文档：
https://godoc.org/github.com/gogf/gf/database/gdb

# 数据结构

为便于数据表记录的操作，ORM定义了5种基本的数据类型：

```go
type Map         map[string]interface{} // 数据记录
type List        []Map                  // 数据记录列表

type Value       *gvar.Var              // 返回数据表记录值
type Record      map[string]Value       // 返回数据表记录键值对
type Result      []Record               // 返回数据表记录列表
```

1. `Map`与```List```用于ORM操作过程中的输入参数类型（与全局类型`g.Map`和`g.List`一致，在项目开发中常用`g.Map`和`g.List`替换）；
2. `Value/Record/Result`用于ORM操作的结果数据类型，具体说明请查看【[ORM结果处理](database/gdb/result.md)】章节；



# `g.DB`与`gdb.New`、`gdb.Instance`

获取数据库操作对象有三种方式，一种是使用`g.DB`方法（推荐），一种是使用原生`gdb.New`方法，还有一种是使用包原生单例方法`gdb.Instance`，而第一种是推荐的使用方式。这三种方式的区别如下：
1. `g.DB`对象管理方式获取的是单例对象，整合了配置文件的管理功能，支持配置文件热更新；
1. `gdb.New`是创建一个新的数据库对象(非单例)，无法使用配置文件，需要使用包配置管理方法进行配置；
1. `gdb.Instance`是包原生单例管理方法，需要结合配置方法一起使用，通过分组名称(非必需)获取对应配置的数据库单例对象；
1. 其他使用无差别；

> 有这么多对象获取方式原因在于`GF`是一个模块化设计的框架，每个模块皆可单独使用。为了方便开发者使用常用的一些模块，因此框架也提供了一个`g`模块，这是一个高度耦合的模块，封装了一些常用对象的单例获取方式，详见【[对象管理](frame/g/index.md)】章节。


获取ORM对象示例：
```go
// 获取默认配置的数据库对象(配置名称为"default")
db := g.DB()

// 获取配置分组名称为"user-center"的数据库对象
db := g.DB("user-center")

// 使用原生New方法创建数据库对象
db, err := gdb.New()
db, err := gdb.New("user-center")

// 使用原生单例管理方法获取数据库对象单例
db, err := gdb.Instance()
db, err := gdb.Instance("user-center")

// 注意不用的时候不需要使用Close方法关闭数据库连接(并且gdb也没有提供Close方法)，
// 数据库引擎底层采用了链接池设计，当链接不再使用时会自动关闭
```


# 支持的数据库类型

由于go标准库的数据库操作对象采用接口实现，因此提供了非常好的可扩展性和兼容性。

## MySQL

内置支持，无需额外扩展或第三方包接入，直接可用。
依赖的第三方包：https://github.com/go-sql-driver/mysql

## SQLite

在使用时需要引入第三方包 ([ go-sqlite3](https://github.com/mattn/go-sqlite3) )：
```go
_ "github.com/mattn/go-sqlite3"
```
**限制**
1. 不支持`Save/Replace`方法

## PostgreSQL

在使用时需要引入第三方包 ([pq](https://github.com/lib/pq) )：
```go
_ "github.com/lib/pq"
```
**限制**
1. 不支持`LastInsertId`方法
1. 不支持`Save/Replace`方法

## SQL Server

使用时需导入第三方包 ([go-mssqldb](https://github.com/denisenkom/go-mssqldb) )：
```go
_ "github.com/denisenkom/go-mssqldb"
```

**限制**
1. 不支持`LastInsertId`方法
2. 不支持`Save/Replace`方法
3. 仅支持`SQL Server 2005`及其后的版本

## Oracle

使用时需导入第三方包 ([go-oci8](https://github.com/mattn/go-oci8) )：
```go
_ "github.com/mattn/go-oci8"
```
**限制**
1. 不支持`LastInsertId`方法
2. 不支持`Save/Replace`方法





## 其他数据库类型

额外接入新的数据库相当方便，可参考源码中关于`PostgreSQL`、`SQLite`、`Oracle`、`SQL Server`的接入方式。
