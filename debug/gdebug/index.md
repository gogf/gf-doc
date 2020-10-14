# gdebug

`gdebug`模块提供了一些实用的“调试”相关的方法。所谓的“调试”方法大多数和开发环境有一定关系，包含堆栈和调用链信息分析，并且性能往往不是特别高。

**使用方式**：
```go
import "github.com/gogf/gf/debug/gdebug"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/debug/gdebug

```go
func Caller(skip ...int) (function string, path string, line int)
func CallerDirectory() string
func CallerFileLine() string
func CallerFilePath() string
func CallerFunction() string
func CallerPackage() string
func CallerWithFilter(filter string, skip ...int) (function string, path string, line int)
func PrintStack(skip ...int)
func Stack(skip ...int) string
func StackWithFilter(filter string, skip ...int) string
func StackWithFilters(filters []string, skip ...int) string
```

> 熟悉`PHP`的同学可能比较好理解，这里某些方法其实和`PHP`的部分[魔术常量]( https://www.php.net/manual/en/language.constants.predefined.php )功能一致。`CallerDirectory`对应`__DIR__`，`CallerFilePath`对应`__FILE__`，`CallerFunction`对应`__FUNCTION__`。