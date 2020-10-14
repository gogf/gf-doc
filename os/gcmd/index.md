[TOC]

# gcmd

`gcmd`模块提供了对命令行参数、选项的读取功能，以及对应参数的回调函数绑定功能。

**名称解释**：
- **参数**：程序命令行按照顺序进行传递的数据，参数的输入具有顺序性；
- **选项**：控制程序逻辑的附加输入数据，选项名称以`-`或者`--`字符串作为前缀，选项是无序的，可以放置于命令行中任意位置；

**使用方式**：
```go
import "github.com/gogf/gf/os/gcmd"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/os/gcmd

```go
func GetArg(index int, def ...string) string
func GetArgAll() []string
func GetArgVar(index int, def ...string) *gvar.Var
func GetOpt(name string, def ...string) string
func GetOptAll() map[string]string
func GetOptVar(name string, def ...string) *gvar.Var
func Scan(info ...interface{}) string
func Scanf(format string, info ...interface{}) string
func ContainsOpt(name string, def ...string) bool
func AutoRun() error
func BindHandle(cmd string, f func()) error
func BindHandleMap(m map[string]func()) error
func RunHandle(cmd string) error
type Parser
    func Parse(supportedOptions map[string]bool) (*Parser, error)
    func ParseWithArgs(args []string, supportedOptions map[string]bool) (*Parser, error)
    func (p *Parser) GetArg(index int, def ...string) string
    func (p *Parser) GetArgAll() []string
    func (p *Parser) GetArgVar(index int, def ...string) *gvar.Var
    func (p *Parser) GetOpt(name string, def ...string) string
    func (p *Parser) GetOptAll() map[string]string
    func (p *Parser) GetOptVar(name string, def ...string) *gvar.Var
    func (p *Parser) ContainsOpt(name string, def ...string) bool
    func (p *Parser) AutoRun() error
    func (p *Parser) BindHandle(cmd string, f func()) error
    func (p *Parser) BindHandleMap(m map[string]func()) error
    func (p *Parser) RunHandle(cmd string) error
```
主要方法介绍：
1. 模块方法
    - `GetArg*`方法用以获取默认解析的命令行参数，参数通过输入索引位置获取，索引位置从`0`开始，但往往我们需要获取的参数是从`1`开始，因为索引`0`的参数是程序名称；
    - `GetOpt*`方法用以获取默认解析的命令行选项，选项通过名称获取，并且选项的输入没有顺序性，可以输入到任意的命令行位置；
    - `Scan*`方法用户交互式命令中，输入可选择的提示信息，并获取用户输入的参数信息返回；
1. 参数解析
    - `Parse*`方法用以给定解析规则解析命令行参数，解析规则是一个`map`参数，用以指定需要解析的选项信息，并且指定选项是否需要读取数值；
    - 其他方法的作用同模块方法；

## `Parse` 自定义解析
大多数场景中我们需要自定义参数解析。其实参数解析最主要的是针对于选项的解析，因此`Parse*`方法的输入参数需要指定的是选项的解析配置，包括有哪些选项名称，每个选项是否带有数值。根据这一配置便可将所有的参数和选项进行解析归类。

我们来看一个示例。比如在`gf`命令行工具的`build`交叉编译命令中，有这么一段解析代码：https://github.com/gogf/gf-cli/blob/master/commands/build/build.go

```go
parser, err := gcmd.Parse(g.MapStrBool{
    "n,name":    true,
    "v,version": true,
    "a,arch":    true,
    "o,os":      true,
    "p,path":    true,
})
```
可以看到，选项输入参数其实是一个`map`类型。其中键值为选项名称，同一个选项的不同名称可以通过`,`符号进行分隔。比如，该示例中`n`和`name`选项是同一个选项，当用户输入`-n john`的时候，`n`和`name`选项都会获得到数据`john`。

而键值是一个布尔类型，标识该选项是否需要解析参数。这一选项配置是非常重要的，因为有的选项是不需要获得数据的，仅仅作为一个标识。例如，`-f force`这个输入，在需要解析数据的情况下，选项`f`的值为`force`；而在不需要解析选项数据的情况下，其中的`force`便是命令行的一个参数，而不是选项。

## 选项位置与`=`符号
之前提到过，选项的位置在命令行中是任意的，也就是说，以下命令行选项输入其实意义是一样的：
```html
gf build main.go -a amd64 -o linux -n app
gf -a amd64 -o linux build main.go -n app
gf -n app build -o linux -a amd64 main.go 
```
其中，
- `gf`/`build`/`main.go`是参数，索引分别为 `0`, `1`, `2`；因为参数是有序性的，因此无论命令行怎么修改，这三者的顺序却无法改变。
- `a`/`o`/`n`是选项，由于是顺序无关的，通过选项名称获取数据，因此可以随意放置位置。

此外，使用`Parse`方法创建自定义解析的情况下，命令行的选项与数据之间可以通过空格，也可以通过`=`符号进行连接，如：
```
gf build main.go -a=amd64 -o=linux -n=app
```


## 默认命令行解析规则
由于`gcmd`模块提供了一些包方法用以获取默认的命令行解析规则。在默认规则下，选项名称与数据之间应当使用`=`连接（不支持空格），没有`=`符号的选项表示无数据选项。例如，
```
gf build main.go -a=amd64 -o=linux -n=app -f
```
在默认规则下，`gf`/`build`/`main.go`是参数，索引分别为 `0`, `1`, `2`；`a`/`o`/`n`/`f`将会被解析为选项，并且`f`为无数据选项。

假如默认规则下，不使用`=`符号来连接选项参数，例如：
```
gf build main.go -a amd64 -o linux -n app -f
```
那么，`gf`/`build`/`main.go`/`amd64`/`linux`/`app`将会被解析为参数，索引为 `0`到`5`；而`a`/`o`/`n`/`f`将会被解析为无数据选项。




## 回调函数绑定

对于命令行的可执行程序来讲，需要根据执行参选定位到对应的入口函数进行处理，`gcmd`模块提供了以下方法来实现：
```go
func AutoRun() error
func BindHandle(cmd string, f func()) error
func BindHandleMap(m map[string]func()) error
func RunHandle(cmd string) error

func (p *Parser) AutoRun() error
func (p *Parser) BindHandle(cmd string, f func()) error
func (p *Parser) BindHandleMap(m map[string]func()) error
func (p *Parser) RunHandle(cmd string) error
```

1. `BindHandler/BindHandleMap`绑定执行参数对应的回调函数，执行参数索引为`1`，回调函数参数类型为`func()`。
1. `RunHandler`运行指定执行参数的回调函数；
1. `AutoRun`根据执行`参数[1]`自动运行对应注册的回调函数； 
1. 命令行解析对象`Parser`也有类似的回调函数绑定、执行方法；

使用示例：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gcmd"
)

func help() {
    fmt.Println("This is help.")
}

func test() {
    fmt.Println("This is test.")
}

func main() {
    gcmd.BindHandle("help", help)
    gcmd.BindHandle("test", test)
    gcmd.AutoRun()
}
```
编译成二进制文件后进行执行测试：
```shell
$ go build main.go 
$ ./main test
This is test.
$ ./main help
This is help.
$ ./main 
$ 
```

## 更多的实战示例
由于`gf`的命令行工具中大量使用了`gcmd`模块，因此该示例非常丰富，具体请参考源码：https://github.com/gogf/gf-cli





