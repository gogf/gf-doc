# 校验结果

校验结果为一个`Error`对象指针。当数据规则校验成功时，校验方法返回的结果为`nil`。当数据规则校验失败时，返回的该对象是一个结构化的层级map，包含多个字段及其规则及对应错误信息，以便于接收端能够准确定位错误规则。相关数据结构及方法如下：
https://godoc.org/github.com/gogf/gf/util/gvalid
```go
// 校验错误信息: map[键名]map[规则名]错误信息
type ErrorMap map[string]map[string]string

// 校验错误对象
type Error
    func (e *Error) FirstItem() (key string, msgs map[string]string)
    func (e *Error) FirstRule() (rule string, err string)
    func (e *Error) FirstString() (err string)
    func (e *Error) Map() map[string]string
    func (e *Error) Maps() ErrorMap
    func (e *Error) String() string
    func (e *Error) Strings() (errs []string)
```
可以结合后续的示例理解这个数据结构。我们可以通过`Maps()`方法获得该原始错误信息数据结构map。但在大多数时候我们可以通过`Error`对象的其他方法来方便地获取特定的错误信息。

> Tips: 大多数情况下，我们不关心具体出错的校验规则，可以使用`String`方法直接返回所有的错误信息即可。

## 方法说明

获取校验结果的值可以通过多个校验结果方法获取，为让各位开发者有充分的理解，详细说明以下：
1. `FirstItem` 在有多个键名/属性校验错误的时候，用以获取出错的第一个键名，以及其对应的出错规则和错误信息；其顺序性只有使用顺序校验规则时有效，否则返回的结果是随机的；
1. `FirstRule` 会返回`FirstItem`中得第一条出错的规则及错误信息；
1. `FirstString` 会返回`FirstRule`中得第一条规则错误信息；
1. `Map` 会返回`FirstItem`中得出错自规则及对应错误信息`map`;
1. `Maps` 会返回所有的出错键名及对应的出错规则及对应的错误信息(`map[string]map[string]string`)；
1. `String` 会返回所有的错误信息，构成一条字符串返回，多个规则错误信息之间以`;`符号连接；
1. `Strings` 会返回所有的错误信息，构成`[]string`类型返回；