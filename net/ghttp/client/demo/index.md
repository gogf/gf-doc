[TOC]

# 基本使用
我们来看几个HTTP客户端请求的简单示例。

1. 发送`GET`请求，并打印出返回值
    ```go
    if r, err := g.Client().Get("https://goframe.org"); err != nil {
        panic(err)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```

1. 发送`GET`请求，下载远程文件
    ```go
    if r, err := g.Client().Get("https://goframe.org/cover.png"); err != nil {
        panic(err)
    } else {
        defer r.Close()
        gfile.PutBytes("/Users/john/Temp/cover.png", r.ReadAll())
    }
    ```
    下载文件操作，小文件下载非常简单。需要注意的是，如果远程文件比较大时，服务端会分批返回数据，因此会需要客户端发多个`GET`请求，每一次通过`Header`来请求分批的文件范围长度，感兴趣的同学可自行研究相关细节。

1. 发送`POST`请求，并打印出返回值
    ```go
    if r, err := g.Client().Post("http://127.0.0.1:8199/form", "name=john&age=18"); err != nil {
        panic(err)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```
    传递多参数的时候用户可以使用`&`符号进行连接，注意参数值往往需要通过`gurl.Encode`编码一下。

1. 发送`POST`请求，参数为`map`类型，并打印出返回值
    ```go
    if r, err := g.Client().Post(
        "http://127.0.0.1:8199/form", 
        g.Map{
            "submit"   : "1",
            "callback" : "http://127.0.0.1/callback?url=http://baidu.com",
        }
    )); err != nil {
        panic(err)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```
    传递多参数的时候用户可以使用`&`符号进行连接，也可以直接使用`map`（其实之前也提到，任意数据类型都支持，包括`struct`）。

1. 发送`POST`请求，参数为`JSON`数据，并打印出返回值
    ```go
    if r, err := g.Client().Post(
        "http://user.svc/v1/user/create", 
        `{"passport":"john","password":"123456","password-confirm":"123456"}`,
    ); err != nil {
        panic(err)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```
    可以看到，通过`ghttp`客户端发送`JSON`数据请求非常方便，直接通过`Post`方法提交即可，客户端会自动将请求的`Content-Type`设置为`application/json`。

1. 发送`DELETE`请求，并打印出返回值
    ```go
    if r, err := g.Client().Delete("http://user.svc/v1/user/delete/1", "10000"); err != nil {
        panic(err)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```

# `*Bytes`及`*Content`方法

以`Bytes`及`Content`后缀结尾的请求方法为直接获取返回内容的快捷方法，这些方法将会自动读取服务端返回内容并自动关闭请求连接。需要注意的是，如果请求执行失败，返回内容将会为空。

1. 发送`GET`请求，返回服务端返回内容
    ```go
    // 返回content为[]bytes类型
    content := g.Client().GetBytes("https://goframe.org")
    ```
    ```go
    // 返回content为string类型
    content := g.Client().GetContent("https://goframe.org")
    ```
1. 发送`POST`请求，返回服务端返回内容
    ```go
    // 返回content为[]bytes类型
    content := g.Client().PostBytes(
        "http://user.svc/v1/user/create", 
        `{"passport":"john","password":"123456","password-confirm":"123456"}`,
    )
    ```
    ```go
    // 返回content为string类型
    content := g.Client().PostContent(
        "http://user.svc/v1/user/create", 
        `{"passport":"john","password":"123456","password-confirm":"123456"}`,
    )
    ```

# `*Var`方法

以`Var`后缀结尾的请求方法直接请求并获取HTTP接口结果为泛型类型便于转换。往往用于服务端返回格式为`JSON/XML`的情况，通过返回的`gvar.Var`泛型对象可根据需要自动解析。此外，如果请求失败或者请求结果为空，会返回一个空的`g.Var`泛型对象，不影响转换方法调用。

使用示例：
```go
type User struct {
    Id   int
    Name string
}
```
```go
// Struct
var user *User
// 也可以
// var user = new(User)
g.Client().GetVar(url).Scan(&user)
```
```go
// Struct数组
var users []*User
// 也可以
// var users []User
g.Client().GetVar(url).Scan(&users)
```

