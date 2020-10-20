[TOC]

# 业务逻辑封装

## 基本介绍

### 职责划分

所有的业务逻辑实现均封装于`service`层中，不推荐实现于控制器`api`中。`service`层之间的包存在相互调用，不推荐`api`在代码层级相互调用。

### 数据校验

针对于注册逻辑，请求参数输入结构体以及简单的校验规则的放置于控制器`api`中管理，随后往往通过`gconv.Struct`方法转换为`service`对应方法需要的输入参数。`api`的输入参数与`service`的输入参数数据结构往往比较类似，但不是完全一致，但两者相同属性的数据类型往往需要一一对应，以方便转换。

`api`层请求输入参数：
```go
// 注册请求参数，用于前后端交互参数格式约定
type SignUpRequest struct {
	Passport  string `v:"required|length:6,16#账号不能为空|账号长度应当在:min到:max之间"`
	Password  string `v:"required|length:6,16#请输入确认密码|密码长度应当在:min到:max之间"`
	Password2 string `v:"required|length:6,16|same:Password#密码不能为空|密码长度应当在:min到:max之间|两次密码输入不相等"`
	Nickname  string
}
```

`service`层对应方法输入参数：
```go
// 注册输入参数
type SignUpParam struct {
	Passport  string
	Password  string
	Password2 string
	Nickname  string
}
```

`api`请求参数到`service`输入参数的转换：
```go
func (c *C) SignUp(r *ghttp.Request) {
	var (
		data        *SignUpRequest
		signUpParam *user.SignUpParam
	)
	if err := r.Parse(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if err := gconv.Struct(data, &signUpParam); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if err := user.SignUp(signUpParam); err != nil {
		response.JsonExit(r, 1, err.Error())
	} else {
		response.JsonExit(r, 0, "ok")
	}
}
```

> 针对参数比较复杂且后续可能会变动的参数，推荐定义并使用结构体作为输入参数。

## 实现代码

https://github.com/gogf/gf-demos/blob/master/app/service/user
