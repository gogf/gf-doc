[TOC]


# I18N国际化

`GF`框架支持I18N国际化，由`gi18n`模块实现。

> 为方便开发者使用，提高开发效率，该模块已完美地集成到了`WebServer`及视图引擎组件中。

**使用方式**：
```go
import "github.com/gogf/gf/i18n/gi18n"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/i18n/gi18n

```go
func SetDelimiters(left, right string)
func SetLanguage(language string)
func SetPath(path string) error
func T(content string, language ...string) string
func Translate(content string, language ...string) string

type Manager
    func Instance(name ...string) *Manager
    func New(options ...Options) *Manager
    func (m *Manager) SetDelimiters(left, right string)
    func (m *Manager) SetLanguage(language string)
    func (m *Manager) SetPath(path string) error
    func (m *Manager) T(content string, language ...string) string
    func (m *Manager) Translate(content string, language ...string) string
type Options
    func DefaultOptions() Options
```

## 使用配置

### 文件格式
`gi18n`国际化模块支持框架通用的五种配置文件格式：`xml/ini/yaml/toml/json`。同样的，和配置管理模块一样，框架推荐使用`toml`文件格式。

### 文件目录
默认情况下`gi18n`会自动读取当前项目根目录下的`i18n`目录，默认将该目录作为国际化转译文件存储目录。开发者也可以通过`SetPath`方法自定义`i18n`文件的存储目录路径。

在`i18n`目录下可以直接给定国际化文件如：`en.toml`/`ja.toml`/`zh-CN.toml`；也可以给定国际化文件目录，如：`en/editor.toml`/`en/user.toml`、`zh-CN/editor.toml`/`zh-CN/user.toml`。

例如，以下的`i18n`目录配置以及文件格式都是支持的：
```
├── i18n
│   ├── en.toml
│   ├── ja.toml
│   ├── ru.toml
│   ├── zh-CN.toml
│   └── zh-TW.toml
├── i18n-dir
│   ├── en
│   │   ├── hello.toml
│   │   └── world.toml
│   ├── ja
│   │   ├── hello.yaml
│   │   └── world.yaml
│   ├── ru
│   │   ├── hello.ini
│   │   └── world.ini
│   ├── zh-CN
│   │   ├── hello.json
│   │   └── world.json
│   └── zh-TW
│       ├── hello.xml
│       └── world.xml
└── i18n-file
    ├── en.toml
    ├── ja.yaml
    ├── ru.ini
    ├── zh-CN.json
    └── zh-TW.xml
```

### 资源管理器
`gi18n`默认支持资源管理器，也就是说，默认情况下会从`gres`配置管理器中检索`i18n`目录，或者开发者设置的`i18n`目录路径。

## `T/Translate`方法
其中`T`方法为`Translate`方法的简写，也是大多数时候我们推荐使用的方法名称。`T`方法可以给定关键字名称，也可以直接给定模板内容，将会被自动转译并返回转译后的字符串内容。

此外，`T`方法可以通过第二个语言参数指定需要转译的目标语言名称，该名称需要和配置文件/路径中的名称一致，往往是标准化的国际化语言缩写名称如：`en/ja/ru/zh-CN/zh-TW`等等。否则，将会自动使用`Manager`转译对象中设置的语言进行转译。

### 关键字转译

关键字的转译直接给`T`方法传递关键字即可，例如：`T("hello")`、`T("world")`。

### 模板内容转译

`T`方法支持模板内容转换，模板中的关键字默认使用`{#`和`}`标签进行包含，模板解析时将会自动替换该标签中的关键字内容。使用示例：
1. 转译文件
	- `ja.toml`
		```toml
		hello = "こんにちは"
		world = "世界"
		```
	- `ru.toml`
		```toml
		hello = "Привет"
		world = "мир"
		```
	- `zh-CN.toml`
		```toml
		hello = "你好"
		world = "世界"
		```
1. 示例代码
	```go
	package main

	import (
		"fmt"

		"github.com/gogf/gf/i18n/gi18n"
	)

	func main() {
		t := gi18n.New()
		t.SetLanguage("en")
		fmt.Println(t.Translate(`hello`))
		fmt.Println(t.Translate(`GF says: {#hello}{#world}!`))

		t.SetLanguage("ja")
		fmt.Println(t.Translate(`hello`))
		fmt.Println(t.Translate(`GF says: {#hello}{#world}!`))

		t.SetLanguage("ru")
		fmt.Println(t.Translate(`hello`))
		fmt.Println(t.Translate(`GF says: {#hello}{#world}!`))

		fmt.Println(t.Translate(`hello`, "zh-CN"))
		fmt.Println(t.Translate(`GF says: {#hello}{#world}!`, "zh-CN"))
	}
	```
	执行后，终端输出为：
	```html
	Hello
	GF says: HelloWorld!
	こんにちは
	GF says: こんにちは世界!
	Привет
	GF says: Приветмир!
	你好
	GF says: 你好世界!
	```

## `I18N`与视图引擎

`gi18n`默认已经集成到了`GF`框架的视图引擎中，直接在模板文件/内容中使用`gi18n`的关键字标签即可。

此外，也可以通过设置模板变量`I18nLanguage`设置当前模板的解析语言，该变量可以控制不同的模板内容按照不同的国际化语言进行解析。

使用示例：
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.WriteTplContent(`GF says: {#hello}{#world}!`, g.Map{
			"I18nLanguage": r.Get("lang", "zh-CN"),
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，访问以下页面，将会输出：
1. http://127.0.0.1:8199 
	```
	GF says: 你好世界!
	```
1. http://127.0.0.1:8199/?lang=ja  
	```
	GF says: こんにちは世界!
	```

