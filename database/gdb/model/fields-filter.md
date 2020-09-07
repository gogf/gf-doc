[TOC]


# `Fields/FieldsEx`字段过滤

1. `Fields` 用于指定需要操作的表字段，包括查询字段、写入字段、更新字段等过滤；
1. `FieldsEx` 用于例外的字段指定，可用于查询字段、写入字段、更新字段等过滤；

## `Fields`示例
1. 假如`user`表有4个字段`uid`, `nickname`, `passport`, `password`。
1. 查询字段过滤
    ```go
    // SELECT `uid`,`nickname` FROM `user` ORDER BY `uid` asc
    db.Table("user").Fields("uid, nickname").Order("uid asc").All()
    ```
1. 写入字段过滤
    ```go
    m := g.Map{
        "uid"      : 10000,
        "nickname" : "John Guo",
        "passport" : "john",
        "password" : "123456",
    }
    db.Table(table).Fields("nickname,passport,password").Data(m).Insert()
    // INSERT INTO `user`(`nickname`,`passport`,`password`) VALUES('John Guo','john','123456')
    ```

## `FieldsEx`示例
1. 假如`user`表有4个字段`uid`, `nickname`, `passport`, `password`。
1. 查询字段排除
    ```go
    // SELECT `uid`,`nickname` FROM `user`
    db.Table("user").FieldsEx("passport, password").All()
    ```
1. 写入字段排除
    ```go
    m := g.Map{
        "uid"      : 10000,
        "nickname" : "John Guo",
        "passport" : "john",
        "password" : "123456",
    }
    db.Table(table).FieldsEx("uid").Data(m).Insert()
    // INSERT INTO `user`(`nickname`,`passport`,`password`) VALUES('John Guo','john','123456')
    ```
    
# `Filter`字段过滤

`gdb`可以自动同步**数据表结构**到程序缓存中(缓存不过期，直至程序重启/重新部署)，并且可以过滤提交参数中不符合表结构的数据项，该特性可以使用`Filter`方法实现。

使用示例，假如`user`表有4个字段`uid`, `nickname`, `passport`, `password`：
```go
r, err := db.Table("user").Filter().Data(g.Map{
    "id"          : 1,
    "uid"         : 1,
    "passport"    : "john",
    "password"    : "123456",
}).Insert()
// INSERT INTO user(uid,passport,password) VALUES(1, "john", "123456")
```
其中`id`为不存在的字段，在写入数据时将会被过滤掉，不至于被构造成写入SQL中产生执行错误。


# `OmitEmpty`空值过滤

## 写入/更新操作

空值会影响于写入/更新操作方法，如`Insert`, `Replace`, `Update`, `Save`操作。

当 `map`/`struct` 中存在空值如 `nil`,`""`,`0` 时，默认情况下，`gdb`将会将其当做正常的输入参数，因此这些参数也会被更新到数据表。如以下操作（以`map`为例，`struct`同理）：
```go
// UPDATE `user` SET `name`='john',update_time=null WHERE `id`=1
db.Table("user").Data(g.Map{
    "name"        : "john",
    "update_time" : nil,
}).Where("id", 1).Update()
```
针对空值情况，我们可以通过`OmitEmpty`方法来过滤掉这些空值。例如，以上示例可以修改为：
```go
// UPDATE `user` SET `name`='john' WHERE `id`=1
db.Table("user").OmitEmpty().Data(g.Map{
    "name"        : "john",
    "update_time" : nil,
}).Where("id", 1).Update()
```
对于`struct`数据参数，我们也可以进行空值过滤。操作示例：
```go
type User struct {
    Id         int    `orm:"id"`
    Passport   string `orm:"passport"`
    Password   string `orm:"password"`
    NickName   string `orm:"nickname"`
    CreateTime string `orm:"create_time"`
    UpdateTime string `orm:"update_time"`
}
user := User{
    Id        : 1,
    NickName  : "john",
    UpdateTime: gtime.Now().String(),
}
db.Table("user").OmitEmpty().Data(user).Insert()
// INSERT INTO `user`(`id`,`nickname`,`update_time`) VALUES(1,'john','2019-10-01 12:00:00')
```

> 注意，批量写入/更新操作中`OmitEmpty`方法将会失效，因为在批量操作中，必须保证每个写入记录的字段是统一的。

<hr>

关于`omitempty`标签与`OmitEmpty`方法：
1. 针对于`struct`的空值过滤大家会想到`omitempty`的标签。该标签常用于`json`转换的空值过滤，也在某一些第三方的`ORM`库中用作`struct`到数据表字段的空值过滤，即当属性为空值时不做转换。
1. `omitempty`标签与`OmitEmpty`方法所达到的效果是一样的。在`ORM`操作中，我们不建议对`struct`使用`omitempty`的标签来控制字段的空值过滤，而建议使用`OmitEmpty`方法来做控制。因为该标签一旦加上之后便绑定到了`struct`上，没有办法做灵活控制；而通过`OmitEmpty`方法使得开发者可以选择性地、根据业务场景对`struct`做空值过滤，操作更加灵活。

## 数据查询操作

空值也会影响数据查询操作，主要是影响`where`条件参数。我们可以通过`OmitEmpty`方法过滤条件参数中的空值。

使用示例：
```go
// SELECT * FROM `user` WHERE `passport`='john' LIMIT 1
r, err := db.Table("user").Where(g.Map{
    "nickname" : "",
    "passport" : "john",
}).OmitEmpty().One()
```

```go
type User struct {
    Id         int    `orm:"id"`
    Passport   string `orm:"passport"`
    Password   string `orm:"password"`
    NickName   string `orm:"nickname"`
    CreateTime string `orm:"create_time"`
    UpdateTime string `orm:"update_time"`
}
user := User{
    Passport : "john",
}
r, err := db.Table("user").OmitEmpty().Where(user).One()
// SELECT * FROM `user` WHERE `passport`='john' LIMIT 1
```






