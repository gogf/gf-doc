
[TOC]


# 调试模式

为便于开发阶段调试，`gdb`支持调试模式，可以通过`Debug`配置文件项或者`SetDebug`配置方式开启调试模式，
随后任何的数据库SQL操作语句都将会由内置的日志对象中输出到终端或者日志文件中。


# 日志输出

日志输出往往是打印一些调试或者`SQL`语句，日志对象可以通过`SetLogger`方法来设置，也可以通过配置文件来做简单配置，日志的配置请查看`ORM`的配置管理章节。

# 空跑特性

`ORM`空跑可以通过`DryRun`配置项来启用，默认关闭。当`ORM`的空跑特性开启时，读取操作将会提交，而写入、更新、删除操作将会被忽略。该特性往往结合调试模式和日志输出一起使用，用于校验当前的程序（特别是脚本）执行的`SQL`是否符合预期。

# 类型识别

使用`gdb`查询数据时，返回的数据类型将会被自动识别映射到`Go变量类型`。例如: 当字段类型为`int(xx)`时，查询到的字段值类型将会被识别会`int`类型；当字段类型为`varchar(xxx)`/`char(xxx)`/`text`等类型时将会被自动识别为`string`类型。以下以`mysql`类型为例，介绍数据库类型与Go变量类型的自动识别映射关系: https://github.com/gogf/gf/blob/master/database/gdb/gdb_structure.go

|数据库类型 | Go变量类型
|---|---
|`*char`   | `string`
|`*text`   | `string`
|`*binary` | `bytes`
|`*blob`   | `bytes`
|`*int`    | `int`
|`*money`  | `float64`
|`bit`     | `int`
|`big_int` | `int64`
|`float`   | `float64`
|`double`  | `float64`
|`decimal` | `float64`
|`bool`    | `bool`
|`date`      | `time.Time`
|`datetime`  | `time.Time`
|`timestamp` | `time.Time`
|`其他`     | `string`

这一特性对于需要将查询结果进行编码，并通过例如`JSON`方式直接返回给客户端来说将会非常友好。

# 类型转换

`gdb`的数据记录结果（`Value`）支持非常灵活的类型转换，并内置支持常用的数十种数据类型的转换。`Result`/`Record`的类型转换请查看后续【[ORM高级特性](database/gdb/senior.md)】章节。

> `Value`类型是`*gvar.Var`类型的别名，因此可以使用`gvar.Var`数据类型的所有转换方法，具体请查看【[通用动态变量](container/gvar/index.md)】章节

使用示例：

首先，数据表定义如下：
```sql
# 商品表
CREATE TABLE `goods` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(300) NOT NULL COMMENT '商品名称',
  `price` decimal(10,2) NOT NULL COMMENT '商品价格',
  ...
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
其次，数据表中的数据如下：
```html
id   title     price
1    IPhoneX   5999.99
```
最后，示例代码如下：
```go
if r, err := db.Table("goods").FindOne(1); err == nil {
    fmt.Printf("goods    id: %d\n",   r["id"].Int())
    fmt.Printf("goods title: %s\n",   r["title"].String())
    fmt.Printf("goods proce: %.2f\n", r["price"].Float32())
} else {
    glog.Error(err)
}
```
执行后，输出结果为：
```shell
goods    id: 1
goods title: IPhoneX
goods proce: 5999.99
```

# 继承支持

`gdb`模块针对于`struct`继承提供了良好的支持，包括参数传递、结果处理。例如：
```go
type Base struct {
    Uid        int         `orm:"uid"`
    CreateAt   *gtime.Time `orm:"create_at"`
    UpdateAt   *gtime.Time `orm:"update_at"`
    DeleteAt   *gtime.Time `orm:"delete_at"`
}
type User struct {
    Base
    Passport   string `orm:"passport"`
    Password   string `orm:"password"`
    Nickname   string `orm:"nickname"`
}
```
并且，无论多少层级的`struct`继承，`gdb`的参数传递和结果处理都支持。

