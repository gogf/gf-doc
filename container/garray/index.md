[TOC]

# garray

数组容器，提供普通数组，及排序数组，支持数据项唯一性矫正，支持并发安全开关控制。

**使用场景**：

数组操作。

**使用方式**：
```go
import "github.com/gogf/gf/container/garray"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/garray

简要说明：
1. `garray`模块下的对象及方法较多，建议仔细看看接口文档。
1. `garray`支持`int`/`string`/`interface{}`三种常用的数据类型。
1. `garray`支持普通数组和排序数组，普通数组的结构体名称定义为`*Array`格式，排序数组的结构体名称定义为`Sorted*Array`格式，如下：
    - `Array`, `intArray`, `StrArray`
    - `SortedArray`, `SortedIntArray`, `SortedStrArray`
    - 其中排序数组`SortedArray`，需要给定排序比较方法，在工具包`gutil`中也定义了很多`Comparator*`比较方法


> 以下示例主要展示一些常见数组用法，更多的方法请参考接口文档或源码。

## 普通数组
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/container/garray"
)


func main () {
    // 创建并发安全的int类型数组
    a := garray.NewIntArray(true)

    // 添加数据项
    for i := 0; i < 10; i++ {
        a.Append(i)
    }

    // 获取当前数组长度
    fmt.Println(a.Len())

    // 获取当前数据项列表
    fmt.Println(a.Slice())

    // 获取指定索引项
    fmt.Println(a.Get(6))

    // 在指定索引前插入数据项
    a.InsertAfter(9, 11)
    // 在指定索引后插入数据项
    a.InsertBefore(10, 10)
    fmt.Println(a.Slice())

    // 修改指定索引的数据项
    a.Set(0, 100)
    fmt.Println(a.Slice())

    // 搜索数据项，返回搜索到的索引位置
    fmt.Println(a.Search(5))

    // 删除指定索引的数据项
    a.Remove(0)
    fmt.Println(a.Slice())

    // 并发安全，写锁操作
    a.LockFunc(func(array []int) {
        // 将末尾项改为100
        array[len(array) - 1] = 100
    })

    // 并发安全，读锁操作
    a.RLockFunc(func(array []int) {
        fmt.Println(array[len(array) - 1])
    })

    // 清空数组
    fmt.Println(a.Slice())
    a.Clear()
    fmt.Println(a.Slice())
}
```
执行后，输出结果为：
```html
10
[0 1 2 3 4 5 6 7 8 9]
6 true
[0 1 2 3 4 5 6 7 8 9 10 11]
[100 1 2 3 4 5 6 7 8 9 10 11]
5
[1 2 3 4 5 6 7 8 9 10 11]
100
[1 2 3 4 5 6 7 8 9 10 100]
[]
```

## 排序数组

排序数组的方法与普通数组类似，但是带有自动排序功能及唯一性过滤功能。

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/container/garray"
)


func main () {
    // 自定义排序数组，降序排序(SortedIntArray管理的数据是升序)
    a := garray.NewSortedArray(func(v1, v2 interface{}) int {
        if v1.(int) < v2.(int) {
            return 1
        }
        if v1.(int) > v2.(int) {
            return -1
        }
        return 0
    })

    // 添加数据
    a.Add(2)
    a.Add(3)
    a.Add(1)
    fmt.Println(a.Slice())

    // 添加重复数据
    a.Add(3)
    fmt.Println(a.Slice())

    // 检索数据，返回最后对比的索引位置，检索结果
    // 检索结果：0: 匹配; <0:参数小于对比值; >0:参数大于对比值
    fmt.Println(a.Search(1))

    // 设置不可重复
    a.SetUnique(true)
    fmt.Println(a.Slice())
    a.Add(1)
    fmt.Println(a.Slice())
}
```
执行后，输出结果：
```html
[3 2 1]
[3 3 2 1]
3 0
[3 2 1]
[3 2 1]
```

## `Iterate*`数组遍历
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array := garray.NewStrArrayFrom(g.SliceStr{"a", "b", "c"})
	// Iterator is alias of IteratorAsc, which iterates the array readonly in ascending order
	//  with given callback function <f>.
	// If <f> returns true, then it continues iterating; or false to stop.
	array.Iterator(func(k int, v string) bool {
		fmt.Println(k, v)
		return true
	})
	// IteratorDesc iterates the array readonly in descending order with given callback function <f>.
	// If <f> returns true, then it continues iterating; or false to stop.
	array.IteratorDesc(func(k int, v string) bool {
		fmt.Println(k, v)
		return true
	})

	// Output:
	// 0 a
	// 1 b
	// 2 c
	// 2 c
	// 1 b
	// 0 a
}
```

## `Pop*`数组项出栈

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
)

func main() {
	array := garray.NewFrom([]interface{}{1, 2, 3, 4, 5, 6, 7, 8, 9})

	// Any Pop* functions pick, delete and return the item from array.

	fmt.Println(array.PopLeft())
	fmt.Println(array.PopLefts(2))
	fmt.Println(array.PopRight())
	fmt.Println(array.PopRights(2))

	// Output:
	// 1 true
	// [2 3]
	// 9 true
	// [7 8]
}
```

## `Rand/PopRand`数组项随机获取/出栈

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array := garray.NewFrom(g.Slice{1, 2, 3, 4, 5, 6, 7, 8, 9})

	// Randomly retrieve and return 2 items from the array.
	// It does not delete the items from array.
	fmt.Println(array.Rands(2))

	// Randomly pick and return one item from the array.
	// It deletes the picked up item from array.
	fmt.Println(array.PopRand())
}
```


## `Contains/ContainsI`包含判断

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
)

func main() {
	var array garray.StrArray
	array.Append("a")
	fmt.Println(array.Contains("a"))
	fmt.Println(array.Contains("A"))
	fmt.Println(array.ContainsI("A"))

	// Output:
	// true
	// false
	// true
}
```

## `FilterEmpty/FilterNil`空值过滤

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array1 := garray.NewFrom(g.Slice{0, 1, 2, nil, "", g.Slice{}, "john"})
	array2 := garray.NewFrom(g.Slice{0, 1, 2, nil, "", g.Slice{}, "john"})
	fmt.Printf("%#v\n", array1.FilterNil().Slice())
	fmt.Printf("%#v\n", array2.FilterEmpty().Slice())

	// Output:
	// []interface {}{0, 1, 2, "", []interface {}{}, "john"}
	// []interface {}{1, 2, "john"}
}
```

## `Reverse`数组翻转

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array := garray.NewFrom(g.Slice{1, 2, 3, 4, 5, 6, 7, 8, 9})

	// Reverse makes array with elements in reverse order.
	fmt.Println(array.Reverse().Slice())

	// Output:
	// [9 8 7 6 5 4 3 2 1]
}
```

## `Shuffle`随机排序

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array := garray.NewFrom(g.Slice{1, 2, 3, 4, 5, 6, 7, 8, 9})

	// Shuffle randomly shuffles the array.
	fmt.Println(array.Shuffle().Slice())
}
```

## `Walk`遍历修改

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	var array garray.StrArray
	tables := g.SliceStr{"user", "user_detail"}
	prefix := "gf_"
	array.Append(tables...)
	// Add prefix for given table names.
	array.Walk(func(value string) string {
		return prefix + value
	})
	fmt.Println(array.Slice())

	// Output:
	// [gf_user gf_user_detail]
}
```


## `Join`数组项串连
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array := garray.NewFrom(g.Slice{"a", "b", "c", "d"})
	fmt.Println(array.Join(","))

	// Output:
	// a,b,c,d
}
```

## `Chunk`数组拆分
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array := garray.NewFrom(g.Slice{1, 2, 3, 4, 5, 6, 7, 8, 9})

	// Chunk splits an array into multiple arrays,
	// the size of each array is determined by <size>.
	// The last chunk may contain less than size elements.
	fmt.Println(array.Chunk(2))

	// Output:
	// [[1 2] [3 4] [5 6] [7 8] [9]]
}
```

## `Merge`数组合并

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/frame/g"
)

func main() {
	array1 := garray.NewFrom(g.Slice{1, 2})
	array2 := garray.NewFrom(g.Slice{3, 4})
	slice1 := g.Slice{5, 6}
	slice2 := []int{7, 8}
	slice3 := []string{"9", "0"}
	fmt.Println(array1.Slice())
	array1.Merge(array1)
	array1.Merge(array2)
	array1.Merge(slice1)
	array1.Merge(slice2)
	array1.Merge(slice3)
	fmt.Println(array1.Slice())

	// Output:
	// [1 2]
	// [1 2 1 2 3 4 5 6 7 8 9 0]
}
```


## `JSON`序列化/反序列
`garray`模块下的所有容器类型均实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/garray"
    )

    func main() {
        type Student struct {
            Id     int
            Name   string
            Scores *garray.IntArray
        }
        s := Student{
            Id:     1,
            Name:   "john",
            Scores: garray.NewIntArrayFrom([]int{100, 99, 98}),
        }
        b, _ := json.Marshal(s)
        fmt.Println(string(b))
    }
    ```
    执行后，输出结果：
    ```
    {"Id":1,"Name":"john","Scores":[100,99,98]}
    ```
1. `Unmarshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/garray"
    )

    func main() {
        b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
        type Student struct {
            Id     int
            Name   string
            Scores *garray.IntArray
        }
        s := Student{}
        json.Unmarshal(b, &s)
        fmt.Println(s)
    }
    ```
    执行后，输出结果：
    ```
    {1 john [100,99,98]}
    ```
