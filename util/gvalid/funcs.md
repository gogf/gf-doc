# 校验方法

**接口文档**：
https://godoc.org/github.com/gogf/gf/util/gvalid
```go
func Check(value interface{}, rules string, msgs interface{}, params ...map[string]interface{}) *Error
func CheckMap(params map[string]interface{}, rules interface{}, msgs ...CustomMsg) *Error
func CheckStruct(object interface{}, rules interface{}, msgs ...CustomMsg) *Error
```
简要说明：
1. `Check`方法用于单条数据校验，比较简单，方法详细介绍请看后续章节；
1. `CheckMap`方法用于多条数据校验，校验的主体变量为`map`类型，方法详细介绍请看后续章节；
1. `CheckStruct`方法用于多条数据校验，校验的主体变量为结构体对象类型，方法详细介绍请看后续章节；
1. `Check*`方法只有在返回`nil`的情况下，表示数据校验成功，否则返回校验出错的错误信息对象指针`*Error`；


