[TOC]

# 控制器实现

## 基本介绍

### 结构化约束
控制器的输入与输出使用了结构体定义进行约束，结构化维护输入输出数据结构是推荐的方式。例如：
```go
// 账号唯一性检测请求参数，用于前后端交互参数格式约定
type CheckPassportRequest struct {
	Passport string
}
```
虽然仅有一个参数，但也采用了结构化定义，我们直接查看该结构体便可得知该接口的输入参数格式，而不用进入代码中去分析，从而极大提高维护效率。

### 结构体转换
结构体转换可以使用`GetStruct`或者`Parse`方法，其中`Parse`同时可以执行数据校验。结构体转换方法的参数都可以给定一个结构体的空指针，内部会自动初始化结构体对象，转换失败（例如提交参数不存在）不会执行初始化。例如：
```go
var data *SignUpRequest
// 这里没有使用Parse而是仅用GetStruct获取对象，
// 数据校验交给后续的service层统一处理。
if err := r.GetStruct(&data); err != nil {
    response.JsonExit(r, 1, err.Error())
}
```

### 数据校验

可以通过给结构体绑定`v`的标签进行设定校验规则以及定义的错误提示。例如：
```go
// 登录请求参数，用于前后端交互参数格式约定
type SignInRequest struct {
	Passport string `v:"required#账号不能为空"`
	Password string `v:"required#密码不能为空"`
}
```

### 数据传参

控制器负责接收、转换、校验、处理请求参数后，将所需的参数传递给调用的`service`层一个或者多个包方法，而不是直接将`Request`对象传递给`service`。例如：
```go
// 用户登录接口
func (c *C) SignIn(r *ghttp.Request) {
	var data *SignInRequest
	if err := r.Parse(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if err := user.SignIn(data.Passport, data.Password, r.Session); err != nil {
		response.JsonExit(r, 1, err.Error())
	} else {
		response.JsonExit(r, 0, "ok")
	}
}
```
需要注意的是，其中的`Session`对象也是通过控制器传递给`service`。

> 在`Go`的HTTP请求流程中，不存在"全局变量"获取请求参数的方式，只有根据`service`的需要按需传递参数。


## 实现代码

https://github.com/gogf/gf-demos/blob/master/app/api/user/user.go
