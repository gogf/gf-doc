[TOC]

# `ghttp.Client`

`GF`框架提供了强大便捷易用的HTTP客户端，由`ghttp`模块实现。

方法列表：
https://godoc.org/github.com/gogf/gf/net/ghttp

常用方法：
```go
type Client
    func NewClient() *Client
    func (c *Client) Get(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Put(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Post(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Delete(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Connect(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Head(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Options(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Patch(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Trace(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) DoRequest(method, url string, data ...interface{}) (*ClientResponse, error)

    func (c *Client) GetBytes(url string, data ...interface{}) []byte
    func (c *Client) PutBytes(url string, data ...interface{}) []byte
    func (c *Client) PostBytes(url string, data ...interface{}) []byte
    func (c *Client) DeleteBytes(url string, data ...interface{}) []byte
    func (c *Client) ConnectBytes(url string, data ...interface{}) []byte
    func (c *Client) HeadBytes(url string, data ...interface{}) []byte
    func (c *Client) OptionsBytes(url string, data ...interface{}) []byte
    func (c *Client) PatchBytes(url string, data ...interface{}) []byte
    func (c *Client) TraceBytes(url string, data ...interface{}) []byte
    func (c *Client) RequestBytes(method string, url string, data ...interface{}) []byte

    func (c *Client) GetContent(url string, data ...interface{}) string
    func (c *Client) PutContent(url string, data ...interface{}) string
    func (c *Client) PostContent(url string, data ...interface{}) string
    func (c *Client) DeleteContent(url string, data ...interface{}) string
    func (c *Client) ConnectContent(url string, data ...interface{}) string
    func (c *Client) HeadContent(url string, data ...interface{}) string
    func (c *Client) OptionsContent(url string, data ...interface{}) string
    func (c *Client) PatchContent(url string, data ...interface{}) string
    func (c *Client) TraceContent(url string, data ...interface{}) string
    func (c *Client) RequestContent(method string, url string, data ...interface{}) string

    func (c *Client) SetBasicAuth(user, pass string) *Client
    func (c *Client) SetBrowserMode(enabled bool) *Client
    func (c *Client) SetContentType(contentType string) *Client
    func (c *Client) SetCookie(key, value string) *Client
    func (c *Client) SetCookieMap(m map[string]string) *Client
    func (c *Client) SetCtx(ctx context.Context) *Client
    func (c *Client) SetHeader(key, value string) *Client
    func (c *Client) SetHeaderMap(m map[string]string) *Client
    func (c *Client) SetHeaderRaw(headers string) *Client
    func (c *Client) SetPrefix(prefix string) *Client
    func (c *Client) SetRetry(retryCount int, retryInterval int) *Client
    func (c *Client) SetTimeout(t time.Duration) *Client
    func (c *Client) SetProxy(proxyURL string)
```

简要说明：
1. 我们可以使用`NewClient`创建一个自定义的HTTP客户端对象`Client`，随后可以使用该对象执行请求，该对象底层使用了连接池设计，因此没有`Close`关闭方法。HTTP客户端对象也可以通过`g.Client`快捷方法创建。
1. 客户端提供了一系列以`HTTP Method`命名的方法，调用这些方法将会发起对应的`HTTP Method`请求。常用的方法是`Get`和`Post`方法，同时`DoRequest`是核心的请求方法，用户可以调用该方法实现自定义的`HTTP Method`发送请求。
1. 请求返回结果为`*ClientResponse`对象，可以通过该结果对象获取对应的返回结果，通过`ReadAll`/`ReadAllString`方法可以获得返回的内容，该对象在使用完毕后需要通过`Close`方法关闭，防止内存溢出。
1. `*Bytes`方法用于获得服务端返回的二进制数据，如果请求失败返回`nil`；`*Content`方法用于请求获得字符串结果数据，如果请求失败返回空字符串；`Set*`方法用于`Client`的参数设置。
1. 可以看到，客户端的请求参数的数据参数`data`数据类型为`interface{}`类型，也就是说可以传递任意的数据类型，常见的参数数据类型为`string`/`map`，如果参数为`map`类型，参数值将会被自动`urlencode`编码。

# `ghttp.ClientResponse`

`ClientResponse`为HTTP对应请求的返回结果对象，该对象继承于`http.Response`，可以使用`http.Response`的所有方法。在此基础之上增加了以下几个方法：
```go
func (r *ClientResponse) GetCookie(key string) string
func (r *ClientResponse) GetCookieMap() map[string]string
func (r *ClientResponse) Raw() string
func (r *ClientResponse) RawDump()
func (r *ClientResponse) RawRequest() string
func (r *ClientResponse) RawResponse() string
func (r *ClientResponse) ReadAll() []byte
func (r *ClientResponse) ReadAllString() string
func (r *ClientResponse) Close() error
```
这里也要提醒的是，`ClientResponse`需要手动调用`Close`方法关闭，也就是说，不管你使用不使用返回的`ClientResponse`对象，你都需要将该返回对象赋值给一个变量，并且手动调用其`Close`方法进行关闭（往往使用`defer r.Close()`）。


# 一些重要说明

1. `ghttp`客户端默认关闭了`KeepAlive`功能以及对服务端`TLS`证书的校验功能，如果需要启用可自定义客户端的`Transport`属性。
1. 连接池参数设定、连接代理设置这些高级功能也可以通过自定义客户端的`Transport`属性实现，该数据继承于标准库的`http.Transport`对象。







