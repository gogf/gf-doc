[TOC]

# 时间更新

`gdb`模块支持对数据记录的写入、更新、删除时间自动填充，提高开发维护效率。为了便于时间字段名称、类型的统一维护，如果使用该特性，我们约定：
- 字段应当设置允许值为`null`。
- 字段的类型必须为时间类型，如`datetime`, `timestamp`。不支持数字类型字段，如`int`。
- 字段的名称不支持自定义设置，并且固定名称约定为：
    - `create_at`用于记录创建时更新，仅会写入一次。
    - `update_at`用于记录修改时更新，每次记录变更时更新。
    - `delete_at`用于记录的软删除特性，只有当记录删除时会写入一次。

> 字段名称其实不区分大小写，也不判断特殊字符，例如`CreateAt`, `UpdateAt`, `DeleteAt`也是支持的。

## 特性的启用

当数据表包含`create_at`、`update_at`、`delete_at`任意一个或多个字段时，该特性自动启用。

以下的示例中，我们默认示例中的数据表均包含了这3个字段。

## `create_at`写入时间

```go
// INSERT INTO `user`(`name`,`create_at`,`update_at`) VALUES('john', `2020-06-06 21:00:00`, `2020-06-06 21:00:00`)
r, err := db.Table("user").Data(g.Map{"name": "john"}).Insert()
// INSERT IGNORE INTO `user`(`uid`,`name`,`create_at`,`update_at`) VALUES(10000,'john', `2020-06-06 21:00:00`, `2020-06-06 21:00:00`)
r, err := db.Table("user").Data(g.Map{"uid": 10000, "name": "john"}).InsertIgnore()
// REPLACE INTO `user`(`uid`,`name`,`create_at`,`update_at`) VALUES(10000,'john', `2020-06-06 21:00:00`, `2020-06-06 21:00:00`)
r, err := db.Table("user").Data(g.Map{"uid": 10000, "name": "john"}).Replace()
// INSERT INTO `user`(`uid`,`name`,`create_at`,`update_at`) VALUES(10001,'john', `2020-06-06 21:00:00`, `2020-06-06 21:00:00`) ON DUPLICATE KEY UPDATE `uid`=VALUES(`uid`),`name`=VALUES(`name`),`update_at`=VALUES(`update_at`)
r, err := db.Table("user").Data(g.Map{"uid": 10001, "name": "john"}).Save()
```

## `update_at`更新时间
```go
// UPDATE `user` SET `name`='john guo',`update_at`='2020-06-06 21:00:00' WHERE name='john'
r, err := db.Table("user").Data(g.Map{"name" : "john guo"}).Where("name", "john").Update()
// UPDATE `user` SET `status`=1,`update_at`='2020-06-06 21:00:00' ORDER BY `login_time` asc LIMIT 10
r, err := db.Table("user").Data("status", 1).Order("login_time asc").Limit(10).Update()
```


## `delete_at`数据软删除

软删除会稍微比较复杂一些，当软删除存在时，所有的查询语句都将会自动加上`delete_at`的条件。
```go
// UPDATE `user` SET `delete_at`='2020-06-06 21:00:00' WHERE uid=10
r, err := db.Table("user").Where("uid", 10).Delete()
```
查询的时候会发生一些变化，例如：
```go
// SELECT * FROM `user` WHERE uid>1 AND `delete_at` IS NULL
r, err := db.Table("user").Where("uid>?", 1).All()
```
可以看到当数据表中存在`delete_at`字段时，所有涉及到该表的查询操作都将自动加上`delete_at IS NULL`的条件

### 联表查询的场景

如果关联查询的几个表都启用了软删除特性时，会发生以下这种情况，即条件语句中会增加所有相关表的软删除时间判断。

```go
// SELECT * FROM `user` AS `u` LEFT JOIN `user_detail` AS `ud` ON (ud.uid=u.uid) WHERE u.uid=10 AND `u`.`delete_at` IS NULL AND `ud`.`deleteat` IS NULL LIMIT 1
r, err := db.Table("user", "u").LeftJoin("user_detail", "ud", "ud.uid=u.uid").Where("u.uid", 10).One()
```

### `Unscoped`忽略软删除

`Unscoped`用于在链式操作中忽略软删除时间条件，例如上面的示例，加上`Unscoped`方法后：

```go
// SELECT * FROM `user` WHERE uid>1
r, err := db.Table("user").Unscoped().Where("uid>?", 1).All()

// SELECT * FROM `user` AS `u` LEFT JOIN `user_detail` AS `ud` ON (ud.uid=u.uid) WHERE u.uid=10 LIMIT 1
r, err := db.Table("user", "u").LeftJoin("user_detail", "ud", "ud.uid=u.uid").Where("u.uid", 10).Unscoped().One()
```



