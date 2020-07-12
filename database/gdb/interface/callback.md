
## 回调处理

自定义回调处理是最常见的接口开发实现，我们往往只需要对接口中的部分实现进行替换修改，以在原有的实现逻辑中注入自定义的逻辑实现。我们来看一个自定义回调处理的示例，我们需要将所有执行的`SQL`语句记录到`monitor`表中，以方便于进行`SQL`审计。

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
















