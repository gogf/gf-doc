[TOC]

# 配置引入

由于`boot`和`router`包使用了`init`包初始化方式来进行相关配置，因此我们需要使用：
```go
import _ "PATH"
```
方式来引入。

> 需要注意，由于`Go`的`import`是存在先后顺序的，往往需要将这两个包置于所有业务包的最上方引入。

# main包

每个项目至少存在一个`package main`，用于程序的入口执行。

`/main.go`
```go
package main

import (
	_ "github.com/gogf/gf-demos/boot"
  _ "github.com/gogf/gf-demos/router"
  
	"github.com/gogf/gf/frame/g"
)

func main() {
	g.Server().Run()
}
```

需要非常注意：`boot`包的引入往往需要在`main`包中的最顶层位置，以保证应用的初始化最开始进行。特别是一些单例对象的初始化/配置管理操作往往放在`boot`包中，这样其他包才能正常使用到这些单例对象。

这里建议引入`boot`包、`router`包和其他包之间加入一个空行以作区分，特别是`Goland` IDE的`import`插件不会将引入包进行自动排序。

# 编译运行
我们可以使用`IDE`执行运行，也可以使用以下命令编译运行。
```shell
$ go build main.go
$ ./main
```
执行后，终端输出的路由表如下：
```
  SERVER  | DOMAIN  | ADDRESS | METHOD |        ROUTE        |                              HANDLER                              |           MIDDLEWARE             
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/checknickname | github.com/gogf/gf-demos/app/api/user.(*Controller).CheckNickName | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/checkpassport | github.com/gogf/gf-demos/app/api/user.(*Controller).CheckPassport | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/issignedin    | github.com/gogf/gf-demos/app/api/user.(*Controller).IsSignedIn    | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/profile       | github.com/gogf/gf-demos/app/api/user.(*Controller).Profile       | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/profile       | github.com/gogf/gf-demos/app/api/user.(*Controller).Profile       | middleware.CORS,middleware.Auth  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/signin        | github.com/gogf/gf-demos/app/api/user.(*Controller).SignIn        | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/signout       | github.com/gogf/gf-demos/app/api/user.(*Controller).SignOut       | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
  default | default | :8199   | ALL    | /user/signup        | github.com/gogf/gf-demos/app/api/user.(*Controller).SignUp        | middleware.CORS                  
|---------|---------|---------|--------|---------------------|-------------------------------------------------------------------|---------------------------------|
```

# 接口测试

我们通过`curl`命令来对其中两个接口执行简单的测试。

## 1. 用户注册 - `/user/signup`
注册一个账号`test001`，昵称为`john`，密码为`123456`。
```shell
curl -d 'nickname=john&passport=test001&password=123456&password2=123456' http://127.0.0.1:8199/user/signup
```
```json
{"data":null,"code":0,"message":"ok"}
```
我们再次使用刚才的信息注册一次试试。
```shell
curl -d 'nickname=john&passport=test001&password=123456&password2=123456' http://127.0.0.1:8199/user/signup
```
```json
{"data":null,"code":1,"message":"账号 test001 已经存在"}
```
可以看到注册失败了，相同名称只能注册一个账号。

## 2.用户登录 - `/user/signin`
我们先访问获取用户信息的接口，验证鉴权中间件是否生效。
```shell
curl http://127.0.0.1:8199/user/profile
```
```
Forbidden
```
我们用刚才注册的账号登录。
```shell
curl -i -d 'passport=test001&password=123456' http://127.0.0.1:8199/user/signin
```
```
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Origin,Content-Type,Accept,User-Agent,Cookie,Authorization,X-Auth-Token,X-Requested-With
Access-Control-Allow-Methods: GET,PUT,POST,DELETE,PATCH,HEAD,CONNECT,OPTIONS,TRACE
Access-Control-Allow-Origin: *
Access-Control-Max-Age: 3628800
Content-Type: application/json
Server: gf-demos
Set-Cookie: gfsessionid=BZT1SP2OX980EHALYV; Path=/; Expires=Sun, 10 Jan 2021 14:56:36 GMT
Date: Sat, 11 Jan 2020 14:56:36 GMT
Content-Length: 37

{"code":0,"message":"ok","data":null}
```
我们这里使用了`-i`选项用于终端打印出服务端返回的`Header`信息，目的是为了获取`sessionid`。`GF`框架默认的`sessionid`名称为`gfsessionid`，我们看到返回的`Header`中已经有了，并且是通过`Cookie`方式返回的。

随后我们再次访问获取用户信息接口，并且这次提交`gfsessionid`，该信息可以通过`Header`也可以通过`Cookie`提交，服务端都是能够自动识别的。

```shell
curl -H 'gfsessionid:BZT1SP2OX980EHALYV' http://127.0.0.1:8199/user/profile
```

```json
{"code":0,"message":"","data":{"id":1,"passport":"test001","password":"123456","nickname":"john","create_time":"2020-01-10 23:51:41"}}
```

