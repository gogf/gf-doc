[TOC]

# 链式操作

`GF`框架的客户端支持便捷的链式操作，常用方法如下：
```go
func (c *Client) Ctx(ctx context.Context) *Client
func (c *Client) Timeout(t time.Duration) *Client
func (c *Client) Cookie(m map[string]string) *Client
func (c *Client) Header(m map[string]string) *Client
func (c *Client) HeaderRaw(headers string) *Client
func (c *Client) ContentType(contentType string) *Client
func (c *Client) ContentJson() *Client
func (c *Client) ContentXml() *Client
func (c *Client) BasicAuth(user, pass string) *Client
func (c *Client) Retry(retryCount int, retryInterval time.Duration) *Client
func (c *Client) Proxy(proxyURL string) *Client
func (c *Client) RedirectLimit(redirectLimit int) *Client
```

简要说明：
1. `Timeout`方法用于设置当前请求超时时间。
1. `Ctx`方法用于设置当前请求的上下文对象`context.Context`。
1. `Cookie`方法用于设置当前请求的自定义`Cookie`信息。
1. `Header*`方法用于设置当前请求的自定义`Header`信息。
1. `Content*`方法用于设置当前请求的`Content-Type`信息，并且支持根据该信息自动检查提交参数并自动编码。
1. `BasicAuth`方法用于设置`HTTP Basic Auth`校验信息。
1. `Retry`方法用于设置请求失败时重连次数和重连间隔。
1. `Proxy`方法用于设置http访问代理。
1. `RedirectLimit`方法用于限制重定向跳转次数。

# 使用示例

## 示例1，请求超时控制
```go
g.Client().Timeout(3*time.Second).GetContent("http://user.svc/v1/user/info/1")
```
```go
g.Client().Timeout(10*time.Second).PostContent("http://order.svc/v1/order/create", g.Map{
    "uid"         : 1,
    "sku_id"      : 10000,
    "amount"      : 19.99,
    "create_time" : "2020-03-26 12:00:00",
})
```

## 示例2，自定义`Cookie`
```go
g.Client().Cookie("sessionid", "MNV5432PIY76").GetContent("http://user.svc/v1/user/info/1")
```

## 示例3，自定义`Header`
```go
g.Client().Header("Trace-Id", "XVF654RT98UJNMN641V06Y").GetContent("http://user.svc/v1/user/info/1")
```
```go
g.Client().HeaderRaw(`
Referer: https://goframe.org/
User-Agent: MyTesyClient
`).GetContent("http://user.svc/v1/user/info")
```

## 示例4，提交`Json`请求
```go
g.Client().ContentJson().PostContent("http://order.svc/v1/order/create", g.Map{
    "uid"         : 1,
    "sku_id"      : 10000,
    "amount"      : 19.99,
    "create_time" : "2020-03-26 12:00:00",
})
```
该请求将会将`Content-Type`设置为`application/json`，并且将提交参数自动编码为`Json`: 
```json
{"uid":1,"sku_id":10000,"amount":19.99,"create_time":"2020-03-26 12:00:00"}
```

## 示例5，提交`Xml`请求
```go
g.Client().ContentXml().PostContent("http://order.svc/v1/order/create", g.Map{
    "uid"         : 1,
    "sku_id"      : 10000,
    "amount"      : 19.99,
    "create_time" : "2020-03-26 12:00:00",
})
```
该请求将会将`Content-Type`设置为`application/xml`，并且将提交参数自动编码为`Xml`: 
```xml
<doc><amount>19.99</amount><create_time>2020-03-26 12:00:00</create_time><sku_id>10000</sku_id><uid>1</uid></doc>
```



