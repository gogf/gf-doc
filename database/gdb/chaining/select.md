[TOC]

# 数据查询


## `Where/And/Or`查询条件

这三个方法用于传递查询条件参数，支持的参数为任意的`string/map/slice/struct/*struct`类型。

`Where`条件参数推荐使用字符串的参数传递方式（并使用`?`占位符预处理），因为`map`/`struct`类型作为查询参数无法保证顺序性，且在部分情况下（数据库有时会帮助你自动进行查询索引优化），数据库的索引和你传递的查询条件顺序有一定关系。

当使用多个`Where`方法连接查询条件时，作用同`And`。
此外，当存在多个查询条件时，`gdb`会默认将多个条件分别使用`()`符号进行包含，这种设计可以非常友好地支持查询条件分组。

使用示例：
```go
// WHERE `uid`=1
Where("uid=1")
Where("uid", 1)
Where("uid=?", 1)
Where(g.Map("uid" : 1))
// WHERE `uid` <= 1000 AND `age` >= 18
Where(g.Map(
    "uid <=" : 1000,
    "age >=" : 18,
))
// WHERE (`uid` <= 1000) AND (`age` >= 18)
Where("uid <=?", 1000).Where("age >=?", 18)
Where("uid <=?", 1000).And("age >=?", 18)
// WHERE `level`=1 OR `money`>=1000000
Where("level=? OR money >=?", 1, 1000000)
// WHERE (`level`=1) OR (`money`>=1000000)
Where("level", 1).Or("money >=", 1000000)
// WHERE `uid` IN(1,2,3)
Where("uid IN(?)", g.Slice{1,2,3})
```
使用`struct`参数的示例，其中`orm`的`tag`用于指定`struct`属性与表字段的映射关系：
```go
type Condition struct{
    Sex int `orm:"sex"`
    Age int `orm:"age"`
}
Where(Condition{1, 18})
// WHERE `sex`=1 AND `age`=18
```

## `WherePri`支持主键的查询条件

`WherePri`方法的功能同`Where`，但提供了对表主键的智能识别，常用于根据主键的便捷数据查询。假如`user`表的主键为`uid`，我们来看一下`Where`与`WherePri`的区别：
```go
// WHERE `uid`=1
Table("user").Where("uid", 1)
Table("user").WherePri(1)
// WHERE `uid` IN(1,2,3)
Table("user").Where("uid", g.Slice{1,2,3})
Table("user").WherePri(g.Slice{1,2,3})
```
可以看到，当使用`WherePri`方法且给定参数为单一的参数基本类型或者`slice`类型时，将会被识别为主键的查询条件值。



## `All/One/Array/Value/Count`数据查询

这四个方法是数据查询比较常用的方法：
1. `All` 用于查询并返回多条记录的列表/数组。
1. `One` 用于查询并返回单条记录。
1. `Array` 用于查询指定字段列的数据，返回数组。
1. `Value` 用于查询并返回一个字段值，往往需要结合`Fields`方法使用。
1. `Count` 用于查询并返回记录数。

此外，也可以看得到这四个方法定义中也支持条件参数的直接输入，参数类型与`Where`方法一致。但需要注意，其中`Array`和`Value`方法的参数中至少应该输入字段参数。例如：
```go
// SELECT `name` FROM `user` WHERE `score`>60
Table("user").Array("name", "score>?", 60)

// SELECT `name` FROM `user` WHERE `uid`=1
Table("user").Value("name", "uid=1")
Table("user").Value("name", "uid", 1)
// SELECT `name` FROM `user` WHERE `uid` IN(1,2,3)
Table("user").Value("name", "uid", g.Slice{1,2,3})
```

## `Struct/Structs/Scan`数据查询

这三个方法用于便捷地将查询的结果转换为`struct/*struct`或者`[]struct/[]*struct`数据。
1. `Struct`: 将查询结果转换为一个`struct`对象，查询结果应当是特定的一条记录，并且`pointer`参数应当为`struct`对象的指针地址（`*struct`或者`**struct`），使用方式例如：
    ```go
    type User struct {
        Id         int
        Passport   string
        Password   string
        NickName   string
        CreateTime gtime.Time
    }
    user := new(User)
    err  := db.Table("user").Where("id", 1).Struct(user)
    ```
    或者
    ```go
    user ：= &User{}
    err  := db.Table("user").Where("id", 1).Struct(user)
    ```
    前两种方式都是预先初始化对象（提前分配内存），推荐的方式：
    ```go
    user := (*User)(nil)
    err  := db.Table("user").Where("id", 1).Struct(&user)
    ```
    这种方式只有在查询到数据的时候才会执行初始化及内存分配。注意在用法上的区别，特别是传递参数类型的差别（前两种方式传递的参数类型是`*User`，这里传递的参数类型其实是`**User`）。
1. `Structs`: 将多条查询结果集转换为一个`[]struct/[]*struct`数组，查询结果应当是多条记录组成的结果集，并且`pointer`应当为数组的指针地址，使用方式例如：
    ```go
    users := ([]User)(nil)
    // 或者 var users []User
    err := db.Table("user").Structs(&users)
    ```
    或者
    ```go
    users := ([]*User)(nil)
    // 或者 var user []*User
    err := db.Table("user").Structs(&users)
    ```
1. `Scan`: 该方法会根据输入参数`pointer`的类型选择调用`Struct`还是`Structs`方法。如果结果是特定的一条记录，那么调用`Struct`方法；如果结果是`slice`类型则调用`Structs`方法。



## `Find*`支持主键条件的数据查询

`Find*`方法包含：`FindAll/FindOne/FineValue/FindCount/FindScan`，这些方法与`All/One/Value/Count/Scan`方法的区别在于，当方法直接给定条件参数时，前者的效果与`WherePri`方法一致；而后者的效果与`Where`方法一致。也就是说`Find*`方法的条件参数支持智能主键识别特性。


## `LeftJoin/RightJoin/InnerJoin`关联查询

1. `LeftJoin` 左关联查询；
1. `RightJoin` 右关联查询；
1. `InnerJoin` 内关联查询；

使用示例：
```go
// 查询符合条件的单条记录(第一条)
// SELECT u.*,ud.site FROM user u LEFT JOIN user_detail ud ON u.uid=ud.uid WHERE u.uid=1 LIMIT 1
r, err := db.Table("user u").LeftJoin("user_detail ud", "u.uid=ud.uid").Fields("u.*,ud.site").Where("u.uid", 1).One()

// 查询指定字段值
// SELECT ud.site FROM user u RIGHT JOIN user_detail ud ON u.uid=ud.uid WHERE u.uid=1 LIMIT 1
r, err := db.Table("user u").RightJoin("user_detail ud", "u.uid=ud.uid").Fields("ud.site").Where("u.uid", 1).Value()

// 分组及排序
// SELECT u.*,ud.city FROM user u INNER JOIN user_detail ud ON u.uid=ud.uid GROUP BY city ORDER BY register_time asc
r, err := db.Table("user u").InnerJoin("user_detail ud", "u.uid=ud.uid").Fields("u.*,ud.city").Group("city").Order("register_time asc").All()

// 不使用join的联表查询
// SELECT u.*,ud.city FROM user u,user_detail ud WHERE u.uid=ud.uid
r, err := db.Table("user u,user_detail ud").Where("u.uid=ud.uid").Fields("u.*,ud.city").All()
```

## `Group/Order`分组与排序

`Group`方法用于查询分组，`Order`方法用于查询排序。使用示例：
```go
// SELECT COUNT(*) total,age FROM `user` GROUP BY age
r, err := db.Table("user").Fields("COUNT(*) total,age").Group("age").All()

// SELECT * FROM `student` ORDER BY class asc,course asc,score desc
r, err := db.Table("student").Order("class asc,course asc,score desc").All()
```

## 自定义数据表别名

```go
// SELECT * FROM `user` AS u LEFT JOIN `user_detail` as ud ON(ud.id=u.id) WHERE u.id=1 LIMIT 1
r, err := db.Table("user", "u").LeftJoin("user_detail", "ud", "ud.id=u.id").Where("u.id", 1).One()
r, err := db.Table("user").As("u").LeftJoin("user_detail", "ud", "ud.id=u.id").Where("u.id", 1).One()
```

## 示例1, 基本使用
`Where + string`，条件参数使用字符串和预处理。
```go
// 查询多条记录并使用Limit分页
// SELECT * FROM user WHERE uid>1 LIMIT 0,10
r, err := db.Table("user").Where("uid > ?", 1).Limit(0, 10).All()

// 使用Fields方法查询指定字段
// 未使用Fields方法指定查询字段时，默认查询为*
// SELECT uid,name FROM user WHERE uid>1 LIMIT 0,10
r, err := db.Table("user").Fileds("uid,name").Where("uid > ?", 1).Limit(0, 10).All()

// 支持多种Where条件参数类型
// SELECT * FROM user WHERE uid=1 LIMIT 1
r, err := db.Table("user").Where("u.uid=1",).One()
r, err := db.Table("user").Where("u.uid", 1).One()
r, err := db.Table("user").Where("u.uid=?", 1).One()
// SELECT * FROM user WHERE (uid=1) AND (name='john') LIMIT 1
r, err := db.Table("user").Where("uid", 1).Where("name", "john").One()
r, err := db.Table("user").Where("uid=?", 1).And("name=?", "john").One()
// SELECT * FROM user WHERE (uid=1) OR (name='john') LIMIT 1
r, err := db.Table("user").Where("uid=?", 1).Or("name=?", "john").One()
```
`Where + slice`，预处理参数可直接通过`slice`参数给定。
```go
// SELECT * FROM user WHERE age>18 AND name like '%john%'
r, err := db.Table("user").Where("age>? AND name like ?", g.Slice{18, "%john%"}).All()
// SELECT * FROM user WHERE status=1
r, err := db.Table("user").Where("status=?", g.Slice{1}).All()
```
`Where + map`，条件参数使用任意`map`类型传递。
```go
// SELECT * FROM user WHERE uid=1 AND name='john' LIMIT 1
r, err := db.Table("user").Where(g.Map{"uid" : 1, "name" : "john"}).One()
// SELECT * FROM user WHERE uid=1 AND age>18 LIMIT 1
r, err := db.Table("user").Where(g.Map{"uid" : 1, "age>" : 18}).One()
```
`Where + struct/*struct`，`struct`标签支持 `orm/json`，映射属性到字段名称关系。
```go
type User struct {
    Id       int    `json:"uid"`
    UserName string `orm:"name"`
}
// SELECT * FROM user WHERE uid =1 AND name='john' LIMIT 1
r, err := db.Table("user").Where(User{ Id : 1, UserName : "john"}).One()
// SELECT * FROM user WHERE uid =1 LIMIT 1
r, err := db.Table("user").Where(&User{ Id : 1}).One()
```

以上的查询条件相对比较简单，我们来看一个比较复杂的查询示例。
```go
condition := g.Map{
    "title like ?"         : "%九寨%",
    "online"               : 1,
    "hits between ? and ?" : g.Slice{1, 10},
    "exp > 0"              : nil,
    "category"             : g.Slice{100, 200},
}
result, err := db.Table("article").Where(condition).All()
// SELECT * FROM article WHERE title like '%九寨%' AND online=1 AND hits between 1 and 10 AND exp > 0 AND category IN(100,200)
```
## 示例2, `select in`查询
使用字符串、`slice`参数类型。当使用`slice`参数类型时，预处理占位符只需要一个`?`即可。
```go
// SELECT * FROM user WHERE uid IN(100,10000,90000)
r, err := db.Table("user").Where("uid IN(?,?,?)", 100, 10000, 90000).All()
r, err := db.Table("user").Where("uid", g.Slice{100, 10000, 90000}).All()
// SELECT * FROM user WHERE gender=1 AND uid IN(100,10000,90000)
r, err := db.Table("user").Where("gender=? AND uid IN(?)", 1, g.Slice{100, 10000, 90000}).All()
// SELECT COUNT(*) FROM user WHERE age in(18,50)
r, err := db.Table("user").Where("age IN(?,?)", 18, 50).Count()
r, err := db.Table("user").Where("age", g.Slice{18, 50}).Count()
```
使用任意`map`参数类型。
```go
// SELECT * FROM user WHERE gender=1 AND uid IN(100,10000,90000)
r, err := db.Table("user").Where(g.Map{
    "gender" : 1,
    "uid"    : g.Slice{100,10000,90000},
}).All()
```
使用`struct`参数类型，注意查询条件的顺序和`struct`的属性定义顺序有关。
```go
type User struct {
    Id     []int  `orm:"uid"`
    Gender int    `orm:"gender"`
}
// SELECT * FROM user WHERE uid IN(100,10000,90000) AND gender=1
r, err := db.Table("user").Where(User{
    "gender" : 1,
    "uid"    : []int{100, 10000, 90000},
}).All()
```

## 示例3, `like`查询
```go
// SELECT * FROM user WHERE name like '%john%'
r, err := db.Table("user").Where("name like ?", "%john%").All()
// SELECT * FROM user WHERE birthday like '1990-%'
r, err := db.Table("user").Where("birthday like ?", "1990-%").All()
```

## 示例4, `sum`查询
```go
// SELECT SUM(score) FROM user WHERE uid=1
r, err := db.Table("user").Fields("SUM(score)").Where("uid", 1).Value()
```

## 示例5, `count`查询
```go
// SELECT COUNT(1) FROM user WHERE `birthday`='1990-10-01'
r, err := db.Table("user").Where("birthday", "1990-10-01").Count()
// SELECT COUNT(uid) FROM user WHERE `birthday`='1990-10-01'
r, err := db.Table("user").Fields("uid").Where("birthday", "1990-10-01").Count()
```

## 示例6, `distinct`查询
```go
// SELECT DISTINCT uid,name FROM user 
r, err := db.Table("user").Fields("DISTINCT uid,name").All()
```