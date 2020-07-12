[TOC]


# 驱动开发

我们可以通过`gdb`模块的接口设计实现：新增框架默认不支持的第三方数据库驱动、对已有支持的驱动进行定制化修改等。

## 驱动注册

之前我们有提到`Driver`的驱动接口，在实现该接口之后，我们可以通过以下方法注册自定义驱动到`gdb`模块：
```go
// Register registers custom database driver to gdb.
func Register(name string, driver Driver) error 
```
其中的驱动名称`name`可以是已有的驱动名称，例如`mysql`, `mssql`, `pgsql`等等，当出现同名的驱动注册时，新的驱动将会覆盖老的驱动。

## 驱动实现

开发一个自定义的驱动并注册到`gdb`模块中非常简单，可以参考`gdb`模块源码中已对接的数据库类型代码示例：
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_mysql.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_mssql.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_pgsql.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_oracle.go
1. https://github.com/gogf/gf/blob/master/database/gdb/gdb_driver_sqlite.go
1. 更多： https://github.com/gogf/gf/blob/master/database/gdb

需要说明的是，最常见的驱动开发或者修改方式是直接继承于现有`*Core`类型，因为在`Driver`接口中会传递该类型的对象，例如：
```go
// DriverMysql is the driver for mysql database.
type DriverMysql struct {
	*Core
}

// New creates and returns a database object for mysql.
// It implements the interface of gdb.Driver for extra database driver installation.
func (d *DriverMysql) New(core *Core, node *ConfigNode) (DB, error) {
	return &DriverMysql{
		Core: core,
	}, nil
}
```