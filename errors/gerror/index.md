[TOC]

# gerror

`gerror`模块提供了更丰富的错误处理功能。

**使用方式**：
```go
import "github.com/gogf/gf/errors/gerror"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/errors/gerror

```go
func New(value interface{}) error
func NewText(text string) error
func Cause(err error) error
func Stack(err error) string
func Wrap(err error, text string) error
func Wrapf(err error, format string, args ...interface{}) error
type Error
    func (err *Error) Cause() error
    func (err *Error) Error() string
    func (err *Error) Format(s fmt.State, verb rune)
    func (err *Error) Stack() string
```

## 错误堆栈

标准库的`error`错误实现比较简单，无法进行堆栈追溯，对于产生错误时的上层调用者来讲不是很友好，无法获得错误的调用链详细信息。`gerror`支持错误堆栈记录，通过`New*`/`Wrap*`均会自动记录当前错误产生时的堆栈信息。

使用示例：
```go
package main

import (
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func OpenFile() error {
	return gerror.New("permission denied")
}

func OpenConfig() error {
	return gerror.Wrap(OpenFile(), "configuration file opening failed")
}

func ReadConfig() error {
	return gerror.Wrap(OpenConfig(), "reading configuration failed")
}

func main() {
	fmt.Printf("%+v", ReadConfig())
}
```
执行后，终端输出：
```html
reading configuration failed: configuration file opening failed: permission denied
1. reading configuration failed
    1). main.ReadConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:18
    2). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:25
2. configuration file opening failed
    1). main.OpenConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:14
    2). main.ReadConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:18
    3). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:25
3. permission denied
    1). main.OpenFile
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:10
    2). main.OpenConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:14
    3). main.ReadConfig
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:18
    4). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror2.go:25
```
可以看到，调用端可以通过`Wrap`方法将底层的错误信息进行层级叠加，并且包含完整的错误堆栈信息。

## `fmt`格式化

通过以上示例我们可以看到，通过`%+v`的打印格式可以打印出完整的堆栈信息，当然`gerror.Error`对象支持多种fmt格式：

|格式符|输出内容
|---|---
|`%v`, `%s`| 打印所有的层级错误信息，构成完成的字符串返回，多个层级使用`: `拼接。
|`%-v`, `%-s`| 打印当前层级的错误信息，返回字符串。
|`%+s`| 打印完整的堆栈信息列表。
|`%+v`| 打印所有的层级错误信息字符串，以及完整的堆栈信息，等同于`%s\n%+s`。

使用示例：
```go
package main

import (
	"errors"
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func Error1() error {
	return errors.New("test1")
}

func Error2() error {
	return gerror.New("test2")
}

func main() {
	err1 := Error1()
	err2 := Error2()
	fmt.Printf("%s, %-s, %+s\n", err1, err1, err1)
	fmt.Printf("%v, %-v, %+v\n", err1, err1, err1)
	fmt.Printf("%s, %-s, %+s\n", err2, err2, err2)
	fmt.Printf("%v, %-v, %+v\n", err2, err2, err2)
}
```
执行后，终端输出为：
```html
test1, test1, test1
test1, test1, test1
test2, test2, 1.	test2
    1). main.Error2
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror1.go:15
    2). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror1.go:20

test2, test2, test2
1. test2
    1). main.Error2
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror1.go:15
    2). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror1.go:20
```

## `Stack`方法

```go
func Stack(err error) string
```
通过`Stack`方法我们可以获得`error`对象的完整堆栈信息，返回堆栈列表字符串。
注意参数为标准库`error`类型，当该参数为`gerror`模块生成的`error`时，
或者开发者自定义的`error`对象实现了`gerror.ApiStack`接口时支持打印，否则，返回空字符串。

使用示例：
```go
package main

import (
	"errors"
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func Error1() error {
	return errors.New("test1")
}

func Error2() error {
	return gerror.New("test2")
}

func main() {
	err1 := Error1()
	err2 := Error2()
	fmt.Println("err1:", gerror.Stack(err1))
	fmt.Println("err2:", gerror.Stack(err2))
}
```
执行后，终端输出：
```html
err1:
 
err2:
 1. test2
    1). main.Error2
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror3.go:15
    2). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror3.go:20
```

## `Cause`方法

```go
func Cause(err error) error
```
通过`Cause`方法我们可以获得`error`对象的根错误信息（原始错误）。
注意参数为标准库`error`类型，当该参数为`gerror`模块生成的`error`时，
或者开发者自定义的`error`对象实现了`gerror.ApiCause`接口时支持打印，否则，返回输出的`error`对象。

使用示例：
```go
package main

import (
	"fmt"

	"github.com/gogf/gf/errors/gerror"
)

func OpenFile() error {
	return gerror.New("permission denied")
}

func OpenConfig() error {
	return gerror.Wrap(OpenFile(), "configuration file opening failed")
}

func ReadConfig() error {
	return gerror.Wrap(OpenConfig(), "reading configuration failed")
}

func main() {
	fmt.Println(gerror.Cause(ReadConfig()))
}
```
执行后，终端输出：
```html
permission denied
```


## 日志输出支持

`glog`日志管理模块天然支持对`gerror`错误堆栈打印支持，这种支持不是强耦合性的，而是通过`fmt`格式化打印接口支持的。

使用示例：
```go
package main

import (
	"errors"

	"github.com/gogf/gf/os/glog"

	"github.com/gogf/gf/errors/gerror"
)

func Error1() error {
	return errors.New("test1")
}

func Error2() error {
	return gerror.New("test2")
}

func main() {
	glog.Println(Error1())
	glog.Println(Error2())
}
```
执行后，终端输出：
```html
2019-07-13 15:01:31.131 test1
2019-07-13 15:01:31.131 test2
1. test2
    1). main.Error2
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror5.go:16
    2). main.main
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/errors/gerror/gerror5.go:21
```

