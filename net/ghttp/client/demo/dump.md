
# 请求信息打印

`ghttp`客户端支持对HTTP请求的输入与输出原始信息获取与打印，方便调试，相关方法如下：
```go
func (r *ClientResponse) Raw() string
func (r *ClientResponse) RawDump()
func (r *ClientResponse) RawRequest() string
func (r *ClientResponse) RawResponse() string
```
可以看到，所有的方法均绑定在`ClientResponse`对象上，也就是说必须要请求结束后才能打印。

## 基本使用

```go
package main

import (
	"github.com/gogf/gf/frame/g"
)

func main() {
	response, err := g.Client().Get("https://goframe.org")
	if err != nil {
		panic(err)
	}
	response.RawDump()
}
```
执行后，终端输出为：
```
+---------------------------------------------+
|                   REQUEST                   |
+---------------------------------------------+
GET / HTTP/1.1
Host: goframe.org
User-Agent: Go-http-client/1.1
Accept-Encoding: gzip




+---------------------------------------------+
|                   RESPONSE                  |
+---------------------------------------------+
HTTP/1.1 200 OK
Connection: close
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Date: Mon, 08 Jun 2020 11:33:26 GMT
Server: nginx/1.10.3 (Ubuntu)


<!DOCTYPE html>
...
```
