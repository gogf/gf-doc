[TOC]

# 文件上传

`GF`支持非常方便的表单文件上传功能，并且HTTP客户端对上传功能进行了必要的封装并极大简化了上传功能调用。

> 注意哦：上传文件大小受到`ghttp.Server`的`ClientMaxBodySize`配置影响：https://pkg.go.dev/github.com/gogf/gf/net/ghttp#ServerConfig 默认支持的上传文件大小为`8MB`。
## 服务端

https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/server.go

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

// Upload uploads files to /tmp .
func Upload(r *ghttp.Request) {
	files := r.GetUploadFiles("upload-file")
    names, err := files.Save("/tmp/")
    if err != nil {
		r.Response.WriteExit(err)
	}
	r.Response.WriteExit("upload successfully: ", names)
}

// UploadShow shows uploading simgle file page.
func UploadShow(r *ghttp.Request) {
	r.Response.Write(`
    <html>
    <head>
        <title>GF Upload File Demo</title>
    </head>
        <body>
            <form enctype="multipart/form-data" action="/upload" method="post">
                <input type="file" name="upload-file" />
                <input type="submit" value="upload" />
            </form>
        </body>
    </html>
    `)
}

// UploadShowBatch shows uploading multiple files page.
func UploadShowBatch(r *ghttp.Request) {
	r.Response.Write(`
    <html>
    <head>
        <title>GF Upload Files Demo</title>
    </head>
        <body>
            <form enctype="multipart/form-data" action="/upload" method="post">
                <input type="file" name="upload-file" />
                <input type="file" name="upload-file" />
                <input type="submit" value="upload" />
            </form>
        </body>
    </html>
    `)
}

func main() {
	s := g.Server()
	s.Group("/upload", func(group *ghttp.RouterGroup) {
		group.POST("/", Upload)
		group.ALL("/show", UploadShow)
		group.ALL("/batch", UploadShowBatch)
	})
	s.SetPort(8199)
	s.Run()
}
```
该服务端提供了3个接口：
1. http://127.0.0.1:8199/upload/show  地址用于展示单个文件上传的H5页面；
1. http://127.0.0.1:8199/upload/batch  地址用于展示多个文件上传的H5页面；
1. http://127.0.0.1:8199/upload  接口用于真实的表单文件上传，该接口同时支持单个文件或者多个文件上传；

我们这里访问  http://127.0.0.1:8199/upload/show  选择需要上传的单个文件，提交之后可以看到文件上传成功到服务器上。

**关键代码说明**
1. 我们在服务端可以通过`r.GetUploadFiles`方法获得上传的所有文件对象，也可以通过`r.GetUploadFile`获取单个上传的文件对象。
1. 在`r.GetUploadFiles("upload-file")`中的参数`"upload-file"`为本示例中客户端上传时的表单文件域名称，开发者可以根据前后端约定在客户端中定义，以方便服务端接收表单文件域参数。
1. 通过`files.Save`可以将上传的多个文件方便地保存到指定的目录下，并返回保存成功的文件名。如果是批量保存，只要任意一个文件保存失败，都将会立即返回错误。此外，`Save`方法的第二个参数支持随机自动命名上传文件。
1. 通过`group.POST("/", Upload)`注册的路由仅支持`POST`方式访问。

## 客户端
    
### 单文件上传

https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/client.go

```go
package main

import (
    "fmt"

    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/os/glog"
)

func main() {
    path := "/home/john/Workspace/Go/github.com/gogf/gf/version.go"
    r, e := ghttp.Post("http://127.0.0.1:8199/upload", "upload-file=@file:"+path)
    if e != nil {
        glog.Error(e)
    } else {
        fmt.Println(string(r.ReadAll()))
        r.Close()
    }
}
```

注意到了吗？文件上传参数格式使用了 `参数名=@file:文件路径` ，HTTP客户端将会自动解析**文件路径**对应的文件内容并读取提交给服务端。原本复杂的文件上传操作被`gf`进行了封装处理，用户只需要使用 `@file:+文件路径` 来构成参数值即可。其中，`文件路径`请使用本地文件绝对路径。

首先运行服务端程序之后，我们再运行这个上传客户端（注意修改上传的文件路径为本地真实文件路径），执行后可以看到文件被成功上传到服务器的指定路径下。

### 多文件上传

https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload-batch/client.go

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/glog"
)

func main() {
	path1 := "/Users/john/Pictures/logo1.png"
	path2 := "/Users/john/Pictures/logo2.png"
	r, e := ghttp.Post(
		"http://127.0.0.1:8199/upload",
		fmt.Sprintf(`upload-file=@file:%s&upload-file=@file:%s`, path1, path2),
	)
	if e != nil {
		glog.Error(e)
	} else {
		fmt.Println(string(r.ReadAll()))
		r.Close()
	}
}
```
可以看到，多个文件上传提交参数格式为`参数名=@file:xxx&参数名=@file:xxx...`，也可以使用`参数名[]=@file:xxx&参数名[]=@file:xxx...`的形式。

首先运行服务端程序之后，我们再运行这个上传客户端（注意修改上传的文件路径为本地真实文件路径），执行后可以看到文件被成功上传到服务器的指定路径下。






## 自定义上传文件名称

很简单，修改`FileName`属性即可。

```go
s := g.Server()
s.BindHandler("/upload", func(r *ghttp.Request) {
    file := r.GetUploadFile("TestFile")
    if file == nil {
        r.Response.Write("empty file")
        return
    }
    file.Filename = "MyCustomFileName.txt"
    fileName, err := file.Save(gfile.TempDir())
    if err != nil {
        r.Response.Write(err)
        return
    }
    r.Response.Write(fileName)
})
s.SetPort(8999)
s.Run()
```









