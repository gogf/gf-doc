[TOC]

# 数据更新/删除

## `Update`更新方法

`Update`用于数据的更新，往往需要结合`Data`及`Where`方法共同使用。`Data`方法用于指定需要更新的数据，`Where`方法用于指定更新的条件范围。同时，`Update`方法也支持直接给定数据和条件参数。

> 需要注意部分数据库类型支持更新条件结合查询、排序、分组、条数等共同使用。

使用示例：
```go
// UPDATE `user` SET `name`='john guo' WHERE name='john'
r, err := db.Table("user").Data(g.Map{"name" : "john guo"}).Where("name", "john").Update()
r, err := db.Table("user").Data("name='john guo'").Where("name", "john").Update()
// UPDATE `user` SET `status`=1 ORDER BY `login_time` asc LIMIT 10
r, err := db.Table("user").Data("status", 1).Order("login_time asc").Limit(10).Update()

// UPDATE `user` SET `status`=1
r, err := db.Table("user").Data("status=1").Update()
r, err := db.Table("user").Data("status", 1).Update()
r, err := db.Table("user").Data(g.Map{"status" : 1}).Update()
```
也可以直接给`Update`方法传递`data`及`where`参数：
```go
// UPDATE `user` SET `name`='john guo' WHERE name='john'
r, err := db.Table("user").Update(g.Map{"name" : "john guo"}, "name", "john")
r, err := db.Table("user").Update("name='john guo'", "name", "john")

// UPDATE `user` SET `status`=1
r, err := db.Table("user").Update("status=1")
r, err := db.Table("user").Update(g.Map{"status" : 1})
```

## `Delete`删除方法

`Delete`方法用于数据的删除。

使用示例
```go
// DELETE FROM `user` WHERE uid=10
r, err := db.Table("user").Where("uid", 10).Delete()
// DELETE FROM `user` ORDER BY `login_time` asc LIMIT 10
r, err := db.Table("user").Order("login_time asc").Limit(10).Delete()
```
也可以直接给`Delete`方法传递`where`参数：
```go
// DELETE FROM `user` WHERE `uid`=10
r, err := db.Table("user").Delete("uid", 10)
// DELETE FROM `user` WHERE `score`<60
r, err := db.Table("user").Delete("score < ", 60)
```
