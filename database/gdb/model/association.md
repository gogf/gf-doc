[TOC]

文档更新中。。。

# 模型关联

`gf`的`ORM`没有采用其他`ORM`常见的`BelongsTo`, `HasOne`, `HasMany`, `ManyToMany`这样的模型关联设计，这样的关联关系维护较繁琐，例如外键约束、额外的标签备注等，对开发者有一定的心智负担。

`gf`框架一如既往地尝试着简化模型关联设计，不倾向于通过向模型结构体中注入过多复杂的标签内容，目标是使得模型关联查询尽可能得易于理解、使用便捷。接下来关于`gf ORM`提供的模型关联实现，目前属于实验性特性。


那么我们就使用一个例子来介绍`gf ORM`提供的模型关联吧。


## 数据结构

为简化示例，我们这里设计得表都尽可能简单，每张表仅包含2-3个字段，方便阐述关联关系即可。

### 1. 用户表
```sql
CREATE TABLE `user` (
  uid int(10) unsigned NOT NULL AUTO_INCREMENT,
  name varchar(45) NOT NULL,
  PRIMARY KEY (uid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2. 用户详情
```sql
CREATE TABLE `user_detail` (
  uid  int(10) unsigned NOT NULL AUTO_INCREMENT,
  address varchar(45) NOT NULL,
  PRIMARY KEY (uid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 3. 用户学分
```sql
CREATE TABLE `user_scores` (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  uid int(10) unsigned NOT NULL,
  score int(10) unsigned NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
## 数据模型
根据表定义，我们可以得知：
1. 用户表与用户详情是`1:1`关系。
1. 用户表与用户学分是`1:N`关系。

那么`Golang`的模型可定义如下：
```go
// 用户表
type EntityUser struct {
    Uid  int    `json:"uid"`
    Name string `json:"name"`
}
// 用户详情
type EntityUserDetail struct {
    Uid     int    `json:"uid"`
    Address string `json:"address"`
}
// 用户学分
type EntityUserScores struct {
    Id    int `json:"id"`
    Uid   int `json:"uid"`
    Score int `json:"score"`
}
// 组合模型，用户信息
type Entity struct {
    User       *EntityUser
    UserDetail *EntityUserDetail
    UserScores []*EntityUserScores
}
```
其中，`EntityUser`, `EntityUserDetail`, `EntityUserScores`分别对应的是用户表、用户详情、用户学分数据表的数据模型。`Entity`是一个组合模型，对应的是一个用户的所有详细信息。

## 数据写入
写入数据时我们这里不涉及也不考虑分布式事务，只是简单的数据库事务即可。
```go
db.Transaction(func(tx *gdb.TX) error {
    r, err := tx.Table(tableUser).Save(EntityUser{
        Name: "john",
    })
    if err != nil {
        return err
    }
    uid, err := r.LastInsertId()
    if err != nil {
        return err
    }
    _, err = tx.Table(tableUserDetail).Save(EntityUserDetail{
        Uid:     int(uid),
        Address: "Beijing DongZhiMen #66",
    })
    if err != nil {
        return err
    }
    _, err = tx.Table(tableUserScores).Save(g.Slice{
        EntityUserScores{Uid: int(uid), Score: 100},
        EntityUserScores{Uid: int(uid), Score: 99},
    })
    return err
})
```

## 数据查询

```go
// 定义用户列表
var users []Entity
// 查询用户基础数据
err := db.Table("user").ScanList(&users, "User")
// 查询用户详情数据
err := db.Table("user_detail").
       Where("uid", gdb.ListItemValues(users, "User", "Uid")).
       ScanList(&users, , "UserDetail", "User", "uid:Uid")
// 查询用户学分数据
err := db.Table("user_detail").
       Where("uid", gdb.ListItemValues(users, "User", "Uid")).
       ScanList(&users, , "UserScores", "User", "uid:Uid")
```








