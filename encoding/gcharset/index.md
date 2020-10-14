
# gcharset

强大的字符编码转换模块。

支持的字符集: 

1. 中文 : `GBK/GB18030/GB2312/Big5`
1. 日文 : `EUCJP/ISO2022JP/ShiftJIS`
1. 韩文 : `EUCKR`
1. Unicode : `UTF-8/UTF-16/UTF-16BE/UTF-16LE`
1. 其他编码 : `macintosh/IBM*/Windows*/ISO-*`

**使用方式**：
```go
import "github.com/gogf/gf/encoding/gcharset"
```
**接口文档**：

https://godoc.org/github.com/gogf/gf/encoding/gcharset

**使用示例**：

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/encoding/gcharset"
)

func main() {
	src        := "~{;(<dR;:x>F#,6@WCN^O`GW!#"
	srcCharset := "GB2312"
	dstCharset := "UTF-8"
	str, err := gcharset.Convert(dstCharset, srcCharset, src)
	if err != nil {
		panic(err)
	}
	fmt.Println(str)
}
```
执行后，终端输出结果为：
```html
花间一壶酒，独酌无相亲。
```
