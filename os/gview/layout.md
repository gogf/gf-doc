[TOC]

# 模板布局

`gview`模板引擎支持两种`layout`模板布局方式：
1. `define` + `template`方式
1. `include`模板嵌入方式

这两种方式均支持对模板变量的传入。

## `define` + `template`

由于`gview`底层采用了`ParseFiles`方式批量解析模板文件，因此可以使用`define`标签定义模板内容块，通过`template`标签在其他任意的模板文件中引入指定的模板内容块。`template`标签支持跨模板引用，也就是说`define`标签定义的模板内容块可能是在其他模板文件中，`template`也可以随意引入。

> 注意，为嵌套的子模板传递模板变量时，应当使用：`{{template "xxx" .}}` 的语法。

使用示例：

![](/images/layout1-1.png)

1. `layout.html`
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>GoFrame Layout</title>
        {{template "header" .}}
    </head>
    <body>
        <div class="container">
        {{template "container" .}}
        </div>
        <div class="footer">
        {{template "footer" .}}
        </div>
    </body>
    </html>
    ```
1. `header.html`
    ```html
    {{define "header"}}
        <h1>{{.header}}</h1>
    {{end}}
    ```
1. `container.html`
    ```html
    {{define "container"}}
    <h1>{{.container}}</h1>
    {{end}}
    ```
1. `footer.html`
    ```html
    {{define "footer"}}
    <h1>{{.footer}}</h1>
    {{end}}
    ```
1. `main.go`
    ```go
    package main

    import (
        "github.com/gogf/gf/frame/g"
        "github.com/gogf/gf/net/ghttp"
    )

    func main() {
        s := g.Server()
        s.BindHandler("/", func(r *ghttp.Request) {
            r.Response.WriteTpl("layout.html", g.Map{
                "header":    "This is header",
                "container": "This is container",
                "footer":    "This is footer",
            })
        })
        s.SetPort(8199)
        s.Run()
    }
    ```

执行后，访问  http://127.0.0.1:8199  结果如下：

![](/images/layout1-2.png)



## `include`模板嵌入

当然我们也可以使用`include`标签来实现页面布局。

> 注意，为嵌套的子模板传递模板变量时，应当使用：`{{include "xxx" .}}` 的语法。

使用示例：

![](/images/layout2-1.png)

1. `layout.html`
    ```html
    {{include "header.html" .}}
    {{include .mainTpl .}}
    {{include "footer.html" .}}
    ```
1. `header.html`
    ```html
    <h1>HEADER</h1>
    ```
1. `footer.html`
    ```html
    <h1>FOOTER</h1>
    ```
1. `main1.html`
    ```html
    <h1>MAIN1</h1>
    ```
1. `main2.html`
    ```html
    <h1>MAIN2</h1>
    ```
1. `main.go`
    ```go
    package main

    import (
        "github.com/gogf/gf/frame/g"
        "github.com/gogf/gf/net/ghttp"
    )

    func main() {
        s := g.Server()
        s.BindHandler("/main1", func(r *ghttp.Request) {
            r.Response.WriteTpl("layout.html", g.Map{
                "mainTpl": "main/main1.html",
            })
        })
        s.BindHandler("/main2", func(r *ghttp.Request) {
            r.Response.WriteTpl("layout.html", g.Map{
                "mainTpl": "main/main2.html",
            })
        })
        s.SetPort(8199)
        s.Run()
    }
    ```

执行后，访问不同的路由地址，将会看到不同的结果：

1.  http://127.0.0.1:8199/main1 

    ![](/images/layout2-2.png)

1.  http://127.0.0.1:8199/main2 

    ![](/images/layout2-3.png)