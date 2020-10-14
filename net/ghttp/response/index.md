
# 基本介绍

`ghttp.Response`对象实现了标准库的`http.ResponseWriter`接口。数据输出使用`Write*`相关方法实现，并且数据输出采用了`Buffer`机制，因此数据的处理效率比较高。任何时候可以通过`OutputBuffer`方法输出缓冲区数据到客户端，并清空缓冲区数据。

接口文档：https://godoc.org/github.com/gogf/gf/net/ghttp#Response

简要说明:
1. `Write*`方法用于数据的输出，可为任意的数据格式，内部通过断言对参数做自动分析。
1. `Write*Exit`方法用于数据输出后退出当前服务方法，可用于替代`return`返回方法。
1. `WriteJson*`/`WriteXml`方法用于特定数据格式的输出，这是为开发者提供的简便方法。
1. `WriteTpl*`方法用于模板输出，解析并输出模板文件，也可以直接解析并输出给定的模板内容。
1. `ParseTpl*`方法用于模板解析，解析模板文件或者模板内容，返回解析后的内容。
1. 其他方法详见接口文档；

此外，需要提一下，`Header`的操作可以通过标准库的方法来实现，例如：
```go
Response.Header().Set("Content-Type", "text/plain; charset=utf-8")
```