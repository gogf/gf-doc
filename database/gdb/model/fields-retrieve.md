[TOC]


# `FieldsStr/FieldsExStr`字段获取

1. `FieldsStr` 用于获取指定表的字段，并可给定字段前缀，字段之间使用"`,`"符号连接成字符串返回；
1. `FieldsExStr` 用于获取指定表中例外的字段，并可给定字段前缀，字段之间使用"`,`"符号连接成字符串返回；

## `FieldsStr`示例
1. 假如`user`表有4个字段`uid`, `nickname`, `passport`, `password`。
1. 查询字段
    ```go
    // uid,nickname,passport,password
    db.Table("user").FieldsStr()
    ```
1. 查询字段给给定前缀
    ```go
    // gf_uid,gf_nickname,gf_passport,gf_password
    db.Table("user").FieldsStr("gf_")
    ```

## `FieldsExStr`示例
1. 假如`user`表有4个字段`uid`, `nickname`, `passport`, `password`。
1. 查询字段排除
    ```go
    // uid,nickname
    db.Table("user").FieldsExStr("passport, password")
    ```
1. 查询字段排除并给定前缀
    ```go
    // gf_uid,gf_nickname
    db.Table("user").FieldsExStr("passport, password", "gf_")
    ```