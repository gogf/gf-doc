[TOC]


# 代理`Proxy`设置

HTTP客户端发起请求时可以设置代理服务器地址`proxyURL`，该该特性使用`SetProxy*`相关方法实现。代理主要支持http和socks5两种形式，分别为`http://USER:PASSWORD@IP:PORT`或`socks5://USER:PASSWORD@IP:PORT`形式。

方法列表：
```go
func (c *Client) SetProxy(proxyURL string)
func (c *Client) Proxy(proxyURL string) *Client
```


我们来看下客户端设置`proxyURL`的示例。

## 普通调用示例
    
1. 使用`SetProxy`方法
```go
package main

import (
   "fmt"
   "github.com/gogf/gf/net/ghttp"
)

func main() {
    client := ghttp.NewClient()
    client.SetProxy("http://127.0.0.1:1081")
    client.SetTimeout(5 * time.Second) 
    response, err := client.Get("https://api.ip.sb/ip")
    if err != nil {
        fmt.Println(err)
    }
    response.RawDump()
}
```


## 链式调用示例

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    client := ghttp.NewClient()
	response, err := client.Proxy("http://127.0.0.1:1081").Get("https://api.ip.sb/ip")
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(response.RawResponse())
}
```
