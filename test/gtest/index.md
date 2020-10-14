
# `gtest`单元测试

`gtest`模块提供了常用的单元测试方法。

**使用方式**：
```go
import "github.com/gogf/gf/test/gtest"
```

**接口文档**： 

https://pkg.go.dev/github.com/gogf/gf/test/gtest

```go
func C(t *testing.T, f func(t *T))
func Assert(value, expect interface{})
func AssertEQ(value, expect interface{})
func AssertGE(value, expect interface{})
func AssertGT(value, expect interface{})
func AssertIN(value, expect interface{})
func AssertLE(value, expect interface{})
func AssertLT(value, expect interface{})
func AssertNE(value, expect interface{})
func AssertNI(value, expect interface{})
func Error(message ...interface{})
func Fatal(message ...interface{})
```

**主要说明**：
1. 使用`C`方法创建一个`Case`，表示一个单元测试用例。一个单元测试方法可以包含多个`C`，每一个`C`包含的用例往往表示该方法的其中一种可能性测试。
1. 断言方法`Assert`支持任意类型的变量比较。`AssertEQ`进行断言比较时，会同时比较类型，即严格断言。
1. 使用大小比较断言方法如`AssertGE`时，参数支持字符串及数字比较，其中字符串比较为大小写敏感。
1. 包含断言方法`AssertIN`及`AssertNI`支持`slice`类型参数，暂不支持`map`类型参数。

> 用于单元测试的包名既可以使用`包名_test`，也可直接使用`包名`（即与测试包同名）。两种使用方式都比较常见，且在`Go`官方标准库中也均有涉及。但需要注意的是，当需要测试包的私有方法/私有变量时，必须使用`包名`命名形式。且在使用`包名`命名方式时，注意仅用于单元测试的相关方法（非`Test*`测试方法）一般定义为私有，不要公开。


**使用示例**： 

例如`gstr`模块其中一个单元测试用例：

```go
package gstr_test

import (
    "github.com/gogf/gf/test/gtest"
    "github.com/gogf/gf/text/gstr"
    "testing"
)

func Test_Trim(t *testing.T) {
    gtest.C(t, func(t *T) {
        t.Assert(gstr.Trim(" 123456\n "),      "123456")
        t.Assert(gstr.Trim("#123456#;", "#;"), "123456")
    })
}
```
也可以这样使用：

```go
package gstr_test

import (
    . "github.com/gogf/gf/test/gtest"
    "github.com/gogf/gf/text/gstr"
    "testing"
)

func Test_Trim(t *testing.T) {
    C(t, func() {
        Assert(gstr.Trim(" 123456\n "),      "123456")
        Assert(gstr.Trim("#123456#;", "#;"), "123456")
    })
}
```
一个单元测试用例可以包含多个`C`，一个`C`也可以执行多个断言。
断言成功时直接PASS，但是如果断言失败，会输出如下类似的错误信息，并终止当前单元测试用例的继续执行（不会终止后续的其他单元测试用例）。
```html
=== RUN   Test_Trim
[ASSERT] EXPECT 123456#; == 123456
1. /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/text/gstr/gstr_z_unit_trim_test.go:20
2. /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/text/gstr/gstr_z_unit_trim_test.go:18
--- FAIL: Test_Trim (0.00s)
FAIL
```
