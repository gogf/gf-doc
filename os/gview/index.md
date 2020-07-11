
[TOC]

# 基本介绍


## 模板引擎特点

1. 简单、易用、强大；
1. 支持多模板目录搜索；
1. 支持`layout`模板设计；
1. 支持模板视图对象单例模式；
1. 与配置管理模块原生集成，使用方便；
1. 底层采用了二级缓存设计，性能高效；
1. 新增模板标签及大量的内置模板变量、模板函数；
1. 支持模板文件修改后自动更新缓存机制，对开发过程更友好；
1. `define`/`template`标签支持跨模板调用（同一模板路径包括子目录下的模板文件）；
1. `include`标签支持任意路径的模板文件引入；

## 通用视图管理

通用视图管理即使用原生的模板引擎`gview`模块来实现模板管理，包括模板读取展示，模板变量渲染等等。可以使用通过方法`gview.Instance`来获取视图单例对象，并可以按照单例对象名称进行获取。同时也可以通过对象管理器`g.View()`来获取默认的单例`gview`对象。

**接口文档**：

https://godoc.org/github.com/gogf/gf/os/gview


简要说明：
1. `gview.Get`用于根据给定的一个模板目录路径，获得对应的单例模板引擎对象；
1. `gview.New`同样可以根据给定的模板目录路径创建模板引擎对象，但没有单例管理；
1. `SetPath/AddPath`用于设置/添加当前模板引擎对象的模板目录路径，其中`SetPath`会覆盖所有的模板目录设置，推荐使用`AddPath`；
1. `Assign/Assigns`用于设置模板变量，通过该模板引擎解析的所有模板均可以使用这些模板变量；
1. `BindFunc`用于绑定模板函数，具体使用方法参考后续示例程序；
1. `Parse/ParseContent`用于解析模板文件/内容，可以在解析时给定临时的模板变量及模板函数；
1. `SetDelimiters`用于设置该模板引擎对象的模板解析分隔符号，默认为`{{ }}`（与`vuejs`前端框架有冲突）；



### 示例1，解析模板文件
`index.tpl`
```html
id:{{.id}}, name:{{.name}}
```
`main.go`
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/template", func(r *ghttp.Request) {
		r.Response.WriteTpl("index.tpl", g.Map{
			"id":   123,
			"name": "john",
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，访问 http://127.0.0.1:8199/template 可以看到解析后的内容为：`id:123, name:john`

### 示例2，解析模板内容
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/template", func(r *ghttp.Request){
		tplContent := `id:{{.id}}, name:{{.name}}`
		r.Response.WriteTplContent(tplContent, g.Map{
			"id"   : 123,
			"name" : "john",
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，访问 http://127.0.0.1:8199/template 可以看到解析后的内容为：`id:123, name:john`

### 示例3，自定义模板变量分隔符

在项目中我们经常会遇到Golang默认模板变量分隔符号与```Vue```的变量分隔符号冲突的情况（都使用的是```{{ }}```），我们可以使用```SetDelimiters```方法来自定义全局的Golang模板变量分隔符号：
```go
// main.go
package main

import (
    "fmt"
    "github.com/gogf/gf/frame/g"
)

func main() {
    v := g.View()
    v.SetDelimiters("${", "}")
    b, err := v.Parse("gview_delimiters.tpl", map[string]interface{} {
        "k" : "v",
    })
    fmt.Println(err)
    fmt.Println(string(b))
}
```
```html
<!-- gview_delimiters.tpl -->
test.tpl content, vars: ${.}
```
执行后，输出结果为：
```shell
<nil>
test.tpl content, vars: map[k:v]
```


## 目录配置方法

`gf`框架的模板引擎支持非常灵活的多目录自动搜索功能，通过`SetPath`可以修改模板目录为唯一的一个目录地址，同时，我们可以通过`AddPath`方法添加多个搜索目录，模板引擎底层将会按照添加目录的顺序作为优先级进行自动检索。直到检索到一个匹配的文件路径为止，如果在所有搜索目录下查找不到模板文件，那么会返回失败。

**默认目录配置**：

`gview`视图对象初始化时，默认会自动添加以下模板文件搜索目录：
1. **当前工作目录及其下的`template`目录**：例如当前的工作目录为`/home/www`时，将会添加`/home/www`及`/home/www/template`；
1. **当前可执行文件所在目录及其下的`template`目录**：例如二进制文件所在目录为`/tmp`时，将会添加`/tmp`及`/tmp/template`；
1. **当前`main`源代码包所在目录及其下的`template`目录**（仅对源码开发环境有效）：例如`main`包所在目录为`/home/john/workspace/gf-app`时，将会添加`/home/john/workspace/gf-app`及`/home/john/workspace/gf-app/template`；

## 修改模板目录
我们可以通过以下方式修改视图对象的模板文件搜索目录，视图对象将会只在该指定目录执行配置文件检索：
1. （推荐）单例模式获取全局View对象，通过`SetPath`方法手动修改；
2. 修改命令行启动参数 - `gf.gview.path`；
3. 修改指定的环境变量 - `GF_GVIEW_PATH`；

例如，我们的执行程序文件为`main`，那么可以通过以下方式修改模板引擎的模板目录(Linux下)：

1. (推荐)通过单例模式
	```go
    g.View().SetPath("/opt/template")
    ```
3. 通过命令行参数
    ```shell
    ./main --gf.gview.path=/opt/template/
    ```
1. 通过环境变量
    * 启动时修改环境变量：
        ```shell
        GF_GVIEW_PATH=/opt/config/; ./main
        ```
    * 使用`genv`模块来修改环境变量：
        ```go
        genv.Set("GF_GVIEW_PATH", "/opt/template")
        ```

## 自动检测更新

模板引擎使用了精心设计的缓存机制，当模板文件第一次被读取后会被缓存到内存，下一次读取时将会直接从缓存中获取，以提高执行效率。并且，模板引擎提供了对模板文件的**自动检测更新机制**，当模板文件在外部被修改后，模板引擎能够即时地监控到并刷新模板文件的缓存内容。

模板引擎的自动检测更新机制是`gf`框架特有的一大特色。

