[TOC]
# 代码示例

`Golang`采用了`package`包设计的概念，对于包所有导出的包方法以及类型方法都应当有代码示例，代码示例不仅仅是作为包使用的示例，更是单元测试的补充。换句话来讲，你开发的包不仅仅是自己知道怎么使用，更需要通过代码示例告诉团队的其他成员如何使用你开发的包，并且可以通过提高单元测试覆盖率来进一步完善包的代码质量。

> 代码示例的单元测试覆盖率只能作为单元测试的一个补充，因为代码示例能够实现的单元测试功能比较简单。

## 文件名称

代码示例的文件放置于包目录中，文件名称一般使用`example_test.go`，这也是标准库的命名方式。在`GF`框架的代码中，同一个包下往往存在多个测试文件，代码示例文件的规则一般使用`包名_example_test.go`文件，并且代码示例文件也可以存在多个，例如：`gset_example_any_test.go`, `gset_example_str_test.go`, `gset_example_int_test.go`用于区分内部不同类型方法的示例。

> `GF`源码文件中有时为了文件排序更优雅，往往会在测试文件命名中加上`z`的字符便于将所有的测试文件排序到所有包文件末尾，例如：`gset_example_z_any_test`。

<img src="/images/WX20200610-231714@2x.png" width="300"/>


## 代码格式

### 包方法
如果是导出的包方法示例，那么示例方法定义为：
```go
func ExampleXXX() {
    // ...
}
```
其中`XXX`表示示例的包方法。

### 类型方法
如果是导出的类型方法示例，那么示例方法定义为：
```go
func ExampleYYY_XXX() {
    // ...
}
```
其中`YYY`表示类型名称，`XXX`表示该类型对应的方法。

### 示例展示

为了能更好地阐述，我们这里以`gset`包为例，代码示例文件内容的格式如下：
```go
package gset_test

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
	"github.com/gogf/gf/frame/g"
)

func ExampleSet_IsSubsetOf() {
	var s1, s2 gset.Set
	s1.Add(g.Slice{1, 2, 3}...)
	s2.Add(g.Slice{2, 3}...)
	fmt.Println(s1.IsSubsetOf(&s2))
	fmt.Println(s2.IsSubsetOf(&s1))

	// Output:
	// false
	// true
}
```

### `Output`断言注释

在代码示例中，`Output`断言注释是非常重要的，在`Output`后面的注释都是该示例在执行后的输出内容断言。`Output`断言注释的存在也将该代码示例作为单元测试的额外补充，提高代码覆盖率。

`Output`断言注释非必需，但是很有必要，我们推荐所有的代码示例方法都加上该注释。如果没有`Output`断言注释，那么该代码示例方法仅仅是作为一个示例，没有更多的意义。

> 我们统一约定注释后保留1个空格，以便注释内容的展示统一和优雅。

当`Output`断言注释存在时，`Goland`会自动识别该方法可直接运行，方法左侧出现运行图标：

<img src="/images/WX20200610-233056@2x.png" width="300"/>

## 示例展示

代码示例写完后，如果是带有`Output`断言注释的方法将会在`go test`单元测试运行时同时运行该示例代码并执行对应的断言判断。如果是单纯的代码示例，将会展示在`Godoc`文档上，并且在`Goland`的方法注释中也会带有该代码注释。

我们仍然使用`gset`包来举例。

### `Godoc`代码示例

https://godoc.org/github.com/gogf/gf/container/gset#Set.IsSubsetOf

<img src="/images/WX20200610-233845@2x.png" width="300"/>

### `Goland`代码示例

鼠标移动到指定方法上即刻便有代码提示框：

<img src="/images/WX20200610-234127@2x.png" width="300"/>









