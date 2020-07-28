
[TOC]

# 数据库操作

`gdb`方法操作相对链式操作更偏底层操作一些，在项目开发中常用链式操作，因为链式操作更简单灵活，但链式操作执行不了太过于复杂的SQL操作，可以交给方法操作来处理。

接口文档：
https://godoc.org/github.com/gogf/gf/database/gdb

```go
// SQL操作方法，返回原生的标准库sql对象
Query(query string, args ...interface{}) (*sql.Rows, error)
Exec(query string, args ...interface{}) (sql.Result, error)
Prepare(query string) (*sql.Stmt, error)

// 数据表记录查询：
// 查询单条记录、查询多条记录、获取记录对象、查询单个字段值(链式操作同理)
GetAll(query string, args ...interface{}) (Result, error)
GetOne(query string, args ...interface{}) (Record, error)
GetValue(query string, args ...interface{}) (Value, error)
GetCount(query string, args ...interface{}) (int, error)
GetStruct(objPointer interface{}, query string, args ...interface{}) error
GetStructs(objPointerSlice interface{}, query string, args ...interface{}) error
GetScan(objPointer interface{}, query string, args ...interface{}) error

// 数据单条操作
Insert(table string, data interface{}, batch...int) (sql.Result, error)
Replace(table string, data interface{}, batch...int) (sql.Result, error)
Save(table string, data interface{}, batch...int) (sql.Result, error)

// 数据批量操作
BatchInsert(table string, list interface{}, batch...int) (sql.Result, error)
BatchReplace(table string, list interface{}, batch...int) (sql.Result, error)
BatchSave(table string, list interface{}, batch...int) (sql.Result, error)

// 数据修改/删除
Update(table string, data interface{}, condition interface{}, args ...interface{}) (sql.Result, error)
Delete(table string, condition interface{}, args ...interface{}) (sql.Result, error)

// 创建链式操作对象(Table为From的别名)
Table(tables string) (*Model)
From(tables string) (*Model)

// 开启事务操作
Begin() (*TX, error)

// 设置管理
SetDebug(debug bool)
SetMaxIdleConns(n int)
SetMaxOpenConns(n int)
SetConnMaxLifetime(n int)
```

需要注意的是：
1. `Query`返回的是原生的标准库的结果集对象，需要自行解析；
1. 在执行数据查询时推荐使用`Get*`系列查询方法；
1. `Insert`/`Replace`/`Save`方法中的`data`参数支持的数据类型为：`string/map/slice/struct/*struct`，当传递为`slice`类型时，自动识别为批量操作，此时`batch`参数有效；
1. `Batch*`方法中的`list`参数类型支持任意的`slice`类型；


## 操作示例


### 1. ORM对象
```go
// 获取默认配置的数据库对象(配置名称为"default")
db := g.Database()
// 或者(别名方式, 推荐)
db := g.DB()

// 获取配置分组名称为"user-center"的数据库对象
db := g.Database("user-center")
// 或者 (别名方式, 推荐)
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

### 2. 数据写入
```go
r, err := db.Insert("user", gdb.Map {
    "name": "john",
})
```

### 3. 数据查询(列表)
```go
list, err := db.GetAll("select * from user limit 2")
list, err := db.GetAll("select * from user where age > ? and name like ?", g.Slice{18, "%john%"})
list, err := db.GetAll("select * from user where status=?", g.Slice{1})
```
### 4. 数据查询(单条)
```go
one, err := db.GetOne("select * from user limit 2")
one, err := db.GetOne("select * from user where uid=1000")
one, err := db.GetOne("select * from user where uid=?", 1000)
one, err := db.GetOne("select * from user where uid=?", g.Slice{1000})
```

### 5. 数据保存
```go
r, err := db.Save("user", gdb.Map {
    "uid"  :  1,
    "name" : "john",
})
```

### 6. 批量操作
```go
// BatchInsert/BatchReplace/BatchSave 同理
_, err := db.BatchInsert("user", gdb.List {
    {"name": "john_1"},
    {"name": "john_2"},
    {"name": "john_3"},
    {"name": "john_4"},
}, 10)
```

### 7. 数据更新/删除
```go
// db.Update/db.Delete 同理
// UPDATE `user` SET `name`='john' WHERE `uid`=10000
r, err := db.Update("user", gdb.Map {"name": "john"}, "uid=?", 10000)
// UPDATE `user` SET `name`='john' WHERE `uid`=10000
r, err := db.Update("user", "name='john'", "uid=10000")
// UPDATE `user` SET `name`='john' WHERE `uid`=10000
r, err := db.Update("user", "name=?", "uid=?", "john", 10000)
```
注意，参数域支持并建议使用预处理模式（使用`?`占位符）进行输入，避免SQL注入风险。
