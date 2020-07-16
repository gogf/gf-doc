# UnmarshalValue

当然，想必您已经猜到了，在对一些复杂类型（如`struct`）的转换时，`gconv`模块内部其实使用了反射的特性来实现的。这虽然为开发者提供了极大的便捷，但是这确实是以性能损耗为代价的。其实在对于`struct`转换时，如果开发者已经明确转换规则，并且对于其中的性能损耗比较在意，那么可以对特定的`struct`实现`UnmarshalValue`接口来实现自定义转换。当使用`gconv`模块对该`struct`进行转换时，无论该`struct`是直接作为转换对象或者转换对象的属性，`gconv`都将会自动识别其实现的`UnmarshalValue`接口并直接调用该接口实现类型转换，而不会使用反射特性来实现转换。

## 接口定义
```go
// apiUnmarshalValue is the interface for custom defined types customizing value assignment.
// Note that only pointer can implement interface apiUnmarshalValue.
type apiUnmarshalValue interface {
	UnmarshalValue(interface{}) error
}
```
可以看到，自定义的类型可以通过定义`UnmarshalValue`方法来实现自定义的类型转换。注意参数为`interface{}`类型，开发者可以在实际使用场景中通过类型断言或者其他方式进行类型转换。

## 使用示例
```go
package main

import (
	"errors"
	"fmt"
	"github.com/gogf/gf/crypto/gcrc32"
	"github.com/gogf/gf/encoding/gbinary"
	"github.com/gogf/gf/util/gconv"
)

type Pkg struct {
	Length uint16 // Total length.
	Crc32  uint32 // CRC32.
	Data   []byte
}

// NewPkg creates and returns a package with given data.
func NewPkg(data []byte) *Pkg {
	return &Pkg{
		Length: uint16(len(data) + 6),
		Crc32:  gcrc32.Encrypt(data),
		Data:   data,
	}
}

// Marshal encodes the protocol struct to bytes.
func (p *Pkg) Marshal() []byte {
	b := make([]byte, 6+len(p.Data))
	copy(b, gbinary.EncodeUint16(p.Length))
	copy(b[2:], gbinary.EncodeUint32(p.Crc32))
	copy(b[6:], p.Data)
	return b
}

// UnmarshalValue decodes bytes to protocol struct.
func (p *Pkg) UnmarshalValue(v interface{}) error {
	b := gconv.Bytes(v)
	if len(b) < 6 {
		return errors.New("invalid package length")
	}
	p.Length = gbinary.DecodeToUint16(b[:2])
	if len(b) < int(p.Length) {
		return errors.New("invalid data length")
	}
	p.Crc32 = gbinary.DecodeToUint32(b[2:6])
	p.Data = b[6:]
	if gcrc32.Encrypt(p.Data) != p.Crc32 {
		return errors.New("crc32 validation failed")
	}
	return nil
}

func main() {
	var p1, p2 *Pkg

	// Create a demo pkg as p1.
	p1 = NewPkg([]byte("123"))
	fmt.Println(p1)

	// Convert bytes from p1 to p2 using gconv.Struct.
	err := gconv.Struct(p1.Marshal(), &p2)
	if err != nil {
		panic(err)
	}
	fmt.Println(p2)
}
```
执行后，终端输出：
```html
&{9 2286445522 [49 50 51]}
&{9 2286445522 [49 50 51]}
```
























