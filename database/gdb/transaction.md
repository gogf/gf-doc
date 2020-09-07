
[TOC]


# 事务操作

`gdb`事务操作比较简单，可以通过两种操作方式来实现。一种是开启事务之后会返回一个事务操作对象`*gdb.TX`，随后可以使用该对象进行如之前章节介绍的方法操作和链式操作；一种是以闭包的形式来操作事务，所有的事务逻辑在闭包中实现。接口文档：
https://godoc.org/github.com/gogf/gf/database/gdb#TX



## `Begin/Commit/Rollback`事务操作

开启事务操作可以通过执行`db.Begin`方法，该方法返回事务的操作对象，类型为`*gdb.Tx`，通过该对象执行后续的数据库操作，并可通过`tx.Commit`提交修改，或者通过`tx.Rollback`回滚修改。

> 开启事务操作后，请务必在不需要使用该对象时，使用`Commit`/`Rollback`操作关闭掉该事务，推荐充分利用好`defer`方法。

### 1. 开启事务操作
```go
if tx, err := db.Begin(); err == nil {
    fmt.Println("开启事务操作")
}
```
事务操作对象可以执行所有db对象的方法，具体请参考[API文档](https://godoc.org/github.com/gogf/gf/database/gdb)。

### 2. 事务回滚操作
```go
if tx, err := db.Begin(); err == nil {
    r, err := tx.Save("user", gdb.Map{
        "uid"  :  1,
        "name" : "john",
    })
    tx.Rollback()
    fmt.Println(r, err)
}
```

### 3. 事务提交操作
```go
if tx, err := db.Begin(); err == nil {
    r, err := tx.Save("user", gdb.Map{
        "uid"  :  1,
        "name" : "john",
    })
    tx.Commit()
    fmt.Println(r, err)
}
```

### 4. 事务链式操作
事务操作对象仍然可以通过`tx.Table`或者`tx.From`方法返回一个链式操作的对象，该对象与`db.Table`或者`db.From`方法返回值相同，只不过数据库操作在事务上执行，可提交或回滚。
```go
if tx, err := db.Begin(); err == nil {
    r, err := tx.Table("user").Data(gdb.Map{"uid":1, "name": "john_1"}).Save()
    tx.Commit()
    fmt.Println(r, err)
}
```
其他链式操作请参考【[链式操作](database/gdb/model/index.md)】章节。

## `Transaction`闭包操作

为方便事务操作，`gdb`提供了事务的闭包操作，通过`Transaction`方法实现，该方法定义如下：
```go
func (db DB) Transaction(f func(tx *TX) error) (err error)
```
当给定的闭包方法返回的`error`为`nil`时，那么闭包执行结束后当前事务自动执行`Commit`提交操作；否则自动执行`Rollback`回滚操作。

> 如果闭包内部操作产生`panic`中断，该事务也将进行回滚。

使用示例：
```go
db.Transaction(func(tx *gdb.TX) error {
    // user
    result, err := tx.Insert("user", g.Map{
        "passport": "john",
        "password": "12345678",
        "nickname": "JohnGuo",
    })
    if err != nil {
        return err
    }
    // user_detail
    id, err := result.LastInsertId()
    if err != nil {
        return err
    }
    _, err = tx.Insert("user_detail", g.Map{
        "uid":       id,
        "site":      "https://johng.cn",
        "true_name": "GuoQiang",
    })
    if err != nil {
        return err
    }
    return nil
})
```