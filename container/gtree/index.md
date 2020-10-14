
[TOC]

# gtree

支持并发安全开关特性的树形容器，树形数据结构的特点是支持有序遍历、内存占用低、复杂度稳定、适合大数据量存储。该模块包含多个数据结构的树形容器：`RedBlackTree`、`AVLTree`和`BTree`。

|类型|数据结构|平均复杂度|支持排序|有序遍历|说明
|---|---|---|---|---|---
|`RedBlackTree`|红黑树|O(log N)|是|是|写入性能比较好
|`AVLTree`|高度平衡树|O(log N)|是|是|查找性能比较好
|`BTree`|B树/B-树|O(log N)|是|是|常用于外部存储

> 参考连接：https://en.wikipedia.org/wiki/Binary_tree


**使用场景**：

关联数组场景、排序键值对场景、大数据量内存CURD场景等等。

**使用方式**：
```go
import "github.com/gogf/gf/container/gtree"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/container/gtree

几种容器的API方法都非常类似，特点是需要在初始化时提供用于排序的方法。


在`gutil`模块中提供了常用的一些基本类型比较方法，可以直接在程序中直接使用，后续也有示例。
```go
func ComparatorByte(a, b interface{}) int
func ComparatorFloat32(a, b interface{}) int
func ComparatorFloat64(a, b interface{}) int
func ComparatorInt(a, b interface{}) int
func ComparatorInt16(a, b interface{}) int
func ComparatorInt32(a, b interface{}) int
func ComparatorInt64(a, b interface{}) int
func ComparatorInt8(a, b interface{}) int
func ComparatorRune(a, b interface{}) int
func ComparatorString(a, b interface{}) int
func ComparatorTime(a, b interface{}) int
func ComparatorUint(a, b interface{}) int
func ComparatorUint16(a, b interface{}) int
func ComparatorUint32(a, b interface{}) int
func ComparatorUint64(a, b interface{}) int
func ComparatorUint8(a, b interface{}) int
```

## 使用示例

### 基本使用

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gtree"
	"github.com/gogf/gf/util/gutil"
)

func main() {
	m := gtree.NewRedBlackTree(gutil.ComparatorInt)

	// 设置键值对
	for i := 0; i < 10; i++ {
		m.Set(i, i*10)
	}
	// 查询大小
	fmt.Println(m.Size())
	// 批量设置键值对(不同的数据类型对象参数不同)
	m.Sets(map[interface{}]interface{}{
		10: 10,
		11: 11,
	})
	fmt.Println(m.Size())

	// 查询是否存在
	fmt.Println(m.Contains(1))

	// 查询键值
	fmt.Println(m.Get(1))

	// 删除数据项
	m.Remove(9)
	fmt.Println(m.Size())

	// 批量删除
	m.Removes([]interface{}{10, 11})
	fmt.Println(m.Size())

	// 当前键名列表(随机排序)
	fmt.Println(m.Keys())
	// 当前键值列表(随机排序)
	fmt.Println(m.Values())

	// 查询键名，当键值不存在时，写入给定的默认值
	fmt.Println(m.GetOrSet(100, 100))

	// 删除键值对，并返回对应的键值
	fmt.Println(m.Remove(100))

	// 遍历map
	m.IteratorAsc(func(k interface{}, v interface{}) bool {
		fmt.Printf("%v:%v ", k, v)
		return true
	})
	fmt.Println()

	// 清空map
	m.Clear()

	// 判断map是否为空
	fmt.Println(m.IsEmpty())
}
```
执行后，输出结果为：
```html
10
12
true
10
11
9
[0 1 2 3 4 5 6 7 8]
[0 10 20 30 40 50 60 70 80]
100
100
0:0 1:10 2:20 3:30 4:40 5:50 6:60 7:70 8:80 
true
```

### 前序/后续遍历

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gtree"
	"github.com/gogf/gf/util/gutil"
)

func main() {
	tree := gtree.NewAVLTree(gutil.ComparatorInt)
	for i := 0; i < 10; i++ {
		tree.Set(i, i*10)
	}
	// 打印树形
	tree.Print()
	// 前序遍历
	fmt.Println("ASC:")
	tree.IteratorAsc(func(key, value interface{}) bool {
		fmt.Println(key, value)
		return true
	})
	// 后续遍历
	fmt.Println("DESC:")
	tree.IteratorDesc(func(key, value interface{}) bool {
		fmt.Println(key, value)
		return true
	})
}
```
执行后，输出结果为：
```html
AVLTree
│           ┌── 9
│       ┌── 8
│   ┌── 7
│   │   │   ┌── 6
│   │   └── 5
│   │       └── 4
└── 3
    │   ┌── 2
    └── 1
        └── 0

ASC:
0 0
1 10
2 20
3 30
4 40
5 50
6 60
7 70
8 80
9 90
DESC:
9 90
8 80
7 70
6 60
5 50
4 40
3 30
2 20
1 10
0 0
```







