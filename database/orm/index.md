
[TOC]

# 基本介绍

gf框架的ORM模块由`gdb`包实现([API文档](https://godoc.org/github.com/johng-cn/gf/g/database/gdb))，最大的特色在于底层默认使用了```map```作为基础的数据表记录载体，而不是使用```struct```，开发者无需预先定义数据表记录struct便可直接对数据表记录执行各种操作。这样的设计赋予了开发者更高的灵活度和简便性，并且由于没有使用反射特性，使得执行性能更加高效（当然ORM也支持数据表记录与struct的映射转换，详细介绍请查看后续【[ORM高级特性](database/orm/senior.md)】章节）。

**注意：为提高数据库操作安全性，在ORM操作中不建议直接将参数拼接成SQL执行，又或者将参数拼接称字符串执行，建议尽量使用预处理的方式（充分使用```?```占位符）来传递SQL参数。**



## 数据结构

为便于数据表记录的操作，ORM定义了5种基本的数据类型：

```go
type Map         map[string]interface{} // 数据记录
type List        []Map                  // 数据记录列表

type Value       []byte                 // 返回数据表记录值
type Record      map[string]Value       // 返回数据表记录键值对
type Result      []Record               // 返回数据表记录列表
```

1. ```Map```与```List```用于ORM操作过程中的输入参数类型（与全局类型```g.Map```和```g.List```一致，在项目开发中常用`g.Map`和`g.List`替换）；
2. ```Value/Record/Result```用于ORM操作的结果数据类型，其中```Result```表示数据表记录列表，```Record```表示一条数据表记录，```Value```表示记录中的一条键值数据；



## 类型转换

gf-orm的数据记录结果（```Value```）支持非常灵活的类型转换，并内置支持常用的17种数据类型的转换。```Result```/```Record```的类型转换请查看后续【[ORM高级特性](database/orm/senior.md)】章节。

方法列表：
```go
func (v Value) IsNil() bool
func (v Value) Bool() bool
func (v Value) Bytes() []byte
func (v Value) Float32() float32
func (v Value) Float64() float64
func (v Value) Int() int
func (v Value) Int16() int16
func (v Value) Int32() int32
func (v Value) Int64() int64
func (v Value) Int8() int8
func (v Value) String() string
func (v Value) Time() time.Time
func (v Value) TimeDuration() time.Duration
func (v Value) Uint() uint
func (v Value) Uint16() uint16
func (v Value) Uint32() uint32
func (v Value) Uint64() uint64
func (v Value) Uint8() uint8
```

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
最后，完整的示例程序如下：
```go
package main

import (
    "fmt"
    "gitee.com/johng/gf/g"
    "gitee.com/johng/gf/g/os/glog"
)

func main() {
	g.Config().SetPath("/home/john/Workspace/gitee.com/johng/gf/geg/frame")
    db := g.Database()
    if r, err := db.Table("goods").Where("id=?", 1).One(); err == nil {
        fmt.Printf("goods    id: %d\n",   r["id"].Int())
        fmt.Printf("goods title: %s\n",   r["title"].String())
        fmt.Printf("goods proce: %.2f\n", r["price"].Float32())
    } else {
        glog.Error(err)
    }
}
```
执行后，输出结果为：
```shell
goods    id: 1
goods title: IPhoneX
goods proce: 5999.99
```