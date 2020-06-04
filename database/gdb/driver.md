[TOC]


# 驱动开发

默认情况下，`gdb`模块已经提供了一些常用的驱动支持，并允许开发者对接自定义的数据库驱动。

适用场景：新增第三方数据库驱动、对已有驱动进行定制化、实现自定义回调处理等。

## 驱动接口


### `Driver`接口

接口文档：https://godoc.org/github.com/gogf/gf/database/gdb#Driver

开发者自定义的驱动需要实现以下接口：
```go
// Driver is the interface for integrating sql drivers into package gdb.
type Driver interface {
	// New creates and returns a database object for specified database server.
	New(core *Core, node *ConfigNode) (DB, error)
}
```
其中的`New`方法用于根据`Core`数据库基础对象以及`ConfigNode`配置对象创建驱动对应的数据库操作对象，需要注意的是，返回的数据库对象需要实现`DB`接口。而数据库基础对象`Core`已经实现了`DB`接口，因此开发者只需要"继承"`Core`对象，然后根据需要覆盖对应的接口实现方法即可。

### `DB`接口

接口文档：https://godoc.org/github.com/gogf/gf/database/gdb#DB

`DB`接口是数据库操作的核心接口，这里主要对接口的几个重要方法做说明：
1. `Open`方法用于创建特定的数据库连接对象，返回的是标准库的`*sql.DB`通用数据库对象。
1. `Do*`系列方法的第一个参数`link`为`Link`接口对象，该对象在`master-slave`模式下可能是一个主节点对象，也可能是从节点对象，因此如果在继承的驱动对象实现中使用该`link`参数时，注意当前的运行模式。`slave`节点在大部分的数据库主从模式中往往是不可写的。
1. `HandleSqlBeforeCommit`方法将会在每一条`SQL`提交给数据库服务端执行时被调用做一些提交前的回调处理。
1. 其他接口方法详见接口文档或者源码文件。

## 驱动注册

通过以下方法注册自定义驱动到`gdb`模块：
```go
// Register registers custom database driver to gdb.
func Register(name string, driver Driver) error 
```
其中的驱动名称`name`可以是已有的驱动名称，例如`mysql`, `mssql`, `pgsql`等等，当出现同名的驱动注册时，新的驱动将会覆盖老的驱动。

## 新增第三方驱动

新增一个第三方的驱动到`gdb`模块中非常简单，可以参考`gdb`模块源码中已对接的数据库类型代码示例：
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_mysql.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_mssql.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_pgsql.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_oracle.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_sqlite.go
1. 更多： https://github.com/gogf/gf/blob/master/database/gdb

## 自定义回调处理
我们来看一个自定义回调处理的示例，我们需要将所有执行的`SQL`语句记录到`monitor`表中，以方便于进行`SQL`审计。

为简化示例编写，我们这里实现了一个自定义的`MySQL`驱动，该驱动继承于`gdb`模块中已经实现的`DriverMysql`，并按照需要修改覆盖相应的接口方法。由于所有的`SQL`语句执行必定会通过`DoQuery`或者`DoExec`接口，因此我们在自定义的驱动中实现并覆盖这两个接口方法即可。

```go
package driver

import (
	"database/sql"
	"github.com/gogf/gf/database/gdb"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/os/gtime"
)

// MyDriver is a custom database driver, which is used for testing only.
// For simplifying the unit testing case purpose, MyDriver struct inherits the mysql driver
// gdb.DriverMysql and overwrites its functions DoQuery and DoExec.
// So if there's any sql execution, it goes through MyDriver.DoQuery/MyDriver.DoExec firstly
// and then gdb.DriverMysql.DoQuery/gdb.DriverMysql.DoExec.
// You can call it sql "HOOK" or "HiJack" as your will.
type MyDriver struct {
	*gdb.DriverMysql
}

func init() {
	// It here registers my custom driver in package initialization function "init".
	// You can later use this type in the database configuration.
	if err := gdb.Register("MyDriver", &MyDriver{}); err != nil {
		panic(err)
	}
}

// New creates and returns a database object for mysql.
// It implements the interface of gdb.Driver for extra database driver installation.
func (d *MyDriver) New(core *gdb.Core, node *gdb.ConfigNode) (gdb.DB, error) {
	return &MyDriver{
		&gdb.DriverMysql{
			Core: core,
		},
	}, nil
}

// DoQuery commits the sql string and its arguments to underlying driver
// through given link object and returns the execution result.
func (d *MyDriver) DoQuery(link gdb.Link, sql string, args ...interface{}) (rows *sql.Rows, err error) {
	tsMilli := gtime.TimestampMilli()
	rows, err = d.DriverMysql.DoQuery(link, sql, args...)
	if _, err := d.DriverMysql.InsertIgnore("monitor", g.Map{
		"sql":   gdb.FormatSqlWithArgs(sql, args),
		"cost":  gtime.TimestampMilli() - tsMilli,
		"time":  gtime.Now(),
		"error": err.Error(),
	}); err != nil {
		panic(err)
	}
	return
}

// DoExec commits the query string and its arguments to underlying driver
// through given link object and returns the execution result.
func (d *MyDriver) DoExec(link gdb.Link, sql string, args ...interface{}) (result sql.Result, err error) {
	tsMilli := gtime.TimestampMilli()
	result, err = d.DriverMysql.DoExec(link, sql, args...)
	if _, err := d.DriverMysql.InsertIgnore("monitor", g.Map{
		"sql":   gdb.FormatSqlWithArgs(sql, args),
		"cost":  gtime.TimestampMilli() - tsMilli,
		"time":  gtime.Now(),
		"error": err.Error(),
	}); err != nil {
		panic(err)
	}
	return
}
```
我们看到，这里在包初始化方法`init`中使用了`gdb.Register("MyDriver", &MyDriver{})`来注册了了一个自定义名称的驱动。我们也可以通过`gdb.Register("mysql", &MyDriver{})`来覆盖已有的框架`mysql`驱动为自己的驱动。

> 驱动名称`mysql`为框架默认的`DriverMysql`驱动的名称。

由于这里我们使用了一个新的驱动名称`MyDriver`，因此在`gdb`配置中的`type`数据库类型时，需要填写该驱动名称。以下是一个使用配置的示例：
```toml
[database]
	type = "MyDriver"
	link = "root:12345678@tcp(127.0.0.1:3306)/test"
```
















