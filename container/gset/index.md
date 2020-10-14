[TOC]

# gset

集合，即不可重复的一组元素，元素项可以为任意类型。

同时，`gset`支持可选的并发安全参数选项，支持并发安全的场景。

**使用场景**：

集合操作。

**使用方式**：
```go
import "github.com/gogf/gf/container/gset"
```

**接口文档**：
https://godoc.org/github.com/gogf/gf/container/gset



## 使用示例

### 基本使用
```go
package main

import (
	"github.com/gogf/gf/container/gset"
	"fmt"
)

func main() {
	// 创建一个并发安全的集合对象
	s := gset.New(true)

	// 添加数据项
	s.Add(1)

	// 批量添加数据项
	s.Add([]interface{}{1, 2, 3}...)

	// 集合数据项大小
	fmt.Println(s.Size())

	// 集合中是否存在指定数据项
	fmt.Println(s.Contains(2))

	// 返回数据项slice
	fmt.Println(s.Slice())

	// 删除数据项
	s.Remove(3)

	// 遍历数据项
	s.Iterator(func(v interface{}) bool {
		fmt.Println("Iterator:", v)
		return true
	})

	// 将集合转换为字符串
	fmt.Println(s.String())

	// 并发安全写锁操作
	s.LockFunc(func(m map[interface{}]struct{}) {
		m[4] = struct{}{}
	})

	// 并发安全读锁操作
	s.RLockFunc(func(m map[interface{}]struct{}) {
		fmt.Println(m)
	})

	// 清空集合
	s.Clear()
	fmt.Println(s.Size())
}
```
执行后，输出结果为：
```html
3
true
[1 2 3]
Iterator: 1
Iterator: 2
[1 2]
map[1:{} 2:{} 4:{}]
0
```

### 交差并补集

我们可以使用以下方法实现交差并补集，并返回一个新的结果集合，
```go
func (set *Set) Intersect(others ...*Set) (newSet *Set)
func (set *Set) Diff(others ...*Set) (newSet *Set)
func (set *Set) Union(others ...*Set) (newSet *Set)
func (set *Set) Complement(full *Set) (newSet *Set)
```
1. `Intersect`: 交集，属于set且属于others的元素为元素的集合。
1. `Diff`: 差集，属于set且不属于others的元素为元素的集合。
1. `Union`: 并集，属于set或属于others的元素为元素的集合。
1. `Complement`: 补集，(前提: set应当为full的子集)属于全集full不属于集合set的元素组成的集合。如果给定的full集合不是set的全集时，返回full与set的差集.

通过集合方法我们可以发现，交差并集方法支持多个集合参数进行计算。以下为简化示例，只使用一个参数集合。

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/container/gset"
)

func main() {
	s1 := gset.NewFrom(g.Slice{1, 2, 3})
	s2 := gset.NewFrom(g.Slice{4, 5, 6})
	s3 := gset.NewFrom(g.Slice{1, 2, 3, 4, 5, 6, 7})

	// 交集
	fmt.Println(s3.Intersect(s1).Slice())
	// 差集
	fmt.Println(s3.Diff(s1).Slice())
	// 并集
	fmt.Println(s1.Union(s2).Slice())
	// 补集
	fmt.Println(s1.Complement(s3).Slice())
}
```
执行后，输出结果为：
```html
[1 2 3]
[4 5 6 7]
[1 2 3 4 5 6]
[7 4 5 6]
```

### `Contains/ContainsI`包含判断

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
)

func main() {
	var set gset.StrSet
	set.Add("a")
	fmt.Println(set.Contains("a"))
	fmt.Println(set.Contains("A"))
	fmt.Println(set.ContainsI("A"))

	// Output:
	// true
	// false
	// true
}
```

### `Pop/Pops`集合项出栈

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
)

func main() {
	var set gset.Set
	set.Add(1, 2, 3, 4)
	fmt.Println(set.Pop())
	fmt.Println(set.Pops(2))
	fmt.Println(set.Size())

	// May Output:
	// 1
	// [2 3]
	// 1
}
```

### `Join`集合项串连
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
)

func main() {
	var set gset.Set
	set.Add("a", "b", "c", "d")
	fmt.Println(set.Join(","))

	// May Output:
	// a,b,c,d
}
```

### `IsSubsetOf`子集判断
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
	"github.com/gogf/gf/frame/g"
)

func main() {
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

### `AddIfNotExist*`判断性写入

判断性写入是指当指定的数据项不存在时则写入并且方法返回`true`，否则忽略吸入并且方法返回`false`。相关方法如下：
- `AddIfNotExist`
- `AddIfNotExistFunc`
- `AddIfNotExistFuncLock`

方法具体描述请查看接口文档或源码注释。

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
)

func main() {
	var set gset.Set
	fmt.Println(set.AddIfNotExist(1))
	fmt.Println(set.AddIfNotExist(1))
	fmt.Println(set.Slice())

	// Output:
	// true
	// false
	// [1]
}
```


### `Walk`遍历修改

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/gset"
	"github.com/gogf/gf/frame/g"
)

func main() {
	var (
		set    gset.StrSet
		names  = g.SliceStr{"user", "user_detail"}
		prefix = "gf_"
	)
	set.Add(names...)
	// Add prefix for given table names.
	set.Walk(func(item string) string {
		return prefix + item
	})
	fmt.Println(set.Slice())

	// May Output:
	// [gf_user gf_user_detail]
}
```

### `JSON`序列化/反序列
`gset`模块下的所有容器类型均实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
	package main

	import (
		"encoding/json"
		"fmt"
		"github.com/gogf/gf/container/gset"
	)

	func main() {
		type Student struct {
			Id     int
			Name   string
			Scores *gset.IntSet
		}
		s := Student{
			Id:     1,
			Name:   "john",
			Scores: gset.NewIntSetFrom([]int{100, 99, 98}),
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
		"github.com/gogf/gf/container/gset"
	)

	func main() {
		b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
		type Student struct {
			Id     int
			Name   string
			Scores *gset.IntSet
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



## 性能测试

https://github.com/gogf/gf/blob/master/container/gset/gset_z_bench_test.go

```html
goos: linux
goarch: amd64
Benchmark_IntSet_Add-4                  10000000               277 ns/op               8 B/op          0 allocs/op
Benchmark_IntSet_Contains-4             20000000              60.6 ns/op               0 B/op          0 allocs/op
Benchmark_IntSet_Remove-4               10000000               211 ns/op               0 B/op          0 allocs/op
Benchmark_AnySet_Add-4                   5000000               312 ns/op              21 B/op          1 allocs/op
Benchmark_AnySet_Contains-4             30000000              68.2 ns/op               0 B/op          0 allocs/op
Benchmark_AnySet_Remove-4                5000000               267 ns/op               0 B/op          0 allocs/op
Benchmark_StrSet_Add-4                   5000000               383 ns/op              20 B/op          1 allocs/op
Benchmark_StrSet_Contains-4             10000000               160 ns/op               7 B/op          0 allocs/op
Benchmark_StrSet_Remove-4                5000000               306 ns/op               7 B/op          0 allocs/op
Benchmark_Unsafe_IntSet_Add-4           10000000               258 ns/op              35 B/op          0 allocs/op
Benchmark_Unsafe_IntSet_Contains-4      20000000               146 ns/op               0 B/op          0 allocs/op
Benchmark_Unsafe_IntSet_Remove-4        10000000               173 ns/op               0 B/op          0 allocs/op
Benchmark_Unsafe_AnySet_Add-4            5000000               355 ns/op              41 B/op          1 allocs/op
Benchmark_Unsafe_AnySet_Contains-4      10000000               150 ns/op               0 B/op          0 allocs/op
Benchmark_Unsafe_AnySet_Remove-4        200000000             11.9 ns/op               0 B/op          0 allocs/op
Benchmark_Unsafe_StrSet_Add-4            5000000               486 ns/op              59 B/op          1 allocs/op
Benchmark_Unsafe_StrSet_Contains-4       5000000               298 ns/op               7 B/op          0 allocs/op
Benchmark_Unsafe_StrSet_Remove-4        10000000               158 ns/op               7 B/op          0 allocs/op
```
