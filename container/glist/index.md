[TOC]

# glist

带并发安全开关的双向列表。

**使用场景**：

双向链表。

**使用方式**：
```go
import "github.com/gogf/gf/container/glist"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/glist

## 性能测试

https://github.com/gogf/gf/blob/master/container/glist/glist_z_bench_test.go

```
goos: linux
goarch: amd64
pkg: github.com/gogf/gf/container/glist
Benchmark_PushBack-4             5000000               268 ns/op              56 B/op          2 allocs/op
Benchmark_PushFront-4           10000000               435 ns/op              56 B/op          2 allocs/op
Benchmark_Len-4                 30000000              44.5 ns/op               0 B/op          0 allocs/op
Benchmark_PopFront-4            20000000              71.1 ns/op               0 B/op          0 allocs/op
Benchmark_PopBack-4             30000000              70.1 ns/op               0 B/op          0 allocs/op
PASS
```

## 基础使用
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/glist"
)

func main() {
	// Not concurrent-safe in default.
	l := glist.New()
	// Push
	l.PushBack(1)
	l.PushBack(2)
	e := l.PushFront(0)
	// Insert
	l.InsertBefore(e, -1)
	l.InsertAfter(e, "a")
	fmt.Println(l)
	// Pop
	fmt.Println(l.PopFront())
	fmt.Println(l.PopBack())
	fmt.Println(l)
}
```
执行后，输出结果：
```
[-1,0,"a",1,2]
-1
2
[0,"a",1]
```

## 链表遍历

该示例中我们将通过读锁和写锁遍历一个并发安全的链表，分别通过`RLockFunc`和`LockFunc`实现。

```go
package main

import (
	"container/list"
	"fmt"
	"github.com/gogf/gf/container/garray"
	"github.com/gogf/gf/container/glist"
)

func main() {
	// concurrent-safe list.
	l := glist.NewFrom(garray.NewArrayRange(1, 10, 1).Slice(), true)
	// iterate reading from head.
	l.RLockFunc(func(list *list.List) {
		length := list.Len()
		if length > 0 {
			for i, e := 0, list.Front(); i < length; i, e = i+1, e.Next() {
				fmt.Print(e.Value)
			}
		}
	})
	fmt.Println()
	// iterate reading from tail.
	l.RLockFunc(func(list *list.List) {
		length := list.Len()
		if length > 0 {
			for i, e := 0, list.Back(); i < length; i, e = i+1, e.Prev() {
				fmt.Print(e.Value)
			}
		}
	})

    fmt.Println()
    
    // iterate reading from head using IteratorAsc.
	l.IteratorAsc(func(e *glist.Element) bool {
		fmt.Print(e.Value)
		return true
	})
	fmt.Println()
	// iterate reading from tail using IteratorDesc.
	l.IteratorDesc(func(e *glist.Element) bool {
		fmt.Print(e.Value)
		return true
	})

	fmt.Println()

	// iterate writing from head.
	l.LockFunc(func(list *list.List) {
		length := list.Len()
		if length > 0 {
			for i, e := 0, list.Front(); i < length; i, e = i+1, e.Next() {
				if e.Value == 6 {
					e.Value = "M"
					break
				}
			}
		}
	})
	fmt.Println(l)
}
```
执行后，输出结果为：
```
12345678910
10987654321
12345678910
10987654321
[1,2,3,4,5,"M",7,8,9,10]
```

## `Pop*`元素项出栈
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/glist"
	"github.com/gogf/gf/frame/g"
)

func main() {
	l := glist.NewFrom(g.Slice{1, 2, 3, 4, 5, 6, 7, 8, 9})

	fmt.Println(l.PopBack())
	fmt.Println(l.PopBacks(2))
	fmt.Println(l.PopFront())
	fmt.Println(l.PopFronts(2))

	// Output:
	// 9
	// [8 7]
	// 1
	// [2 3]
}
```

## `Join`元素项串连
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/glist"
	"github.com/gogf/gf/frame/g"
)

func main() {
	var l glist.List
	l.PushBacks(g.Slice{"a", "b", "c", "d"})

	fmt.Println(l.Join(","))

	// Output:
	// a,b,c,d
}
```

## `JSON`序列化/反序列
`glist`容器实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/glist"
        "github.com/gogf/gf/frame/g"
    )

    func main() {
        type Student struct {
            Id     int
            Name   string
            Scores *glist.List
        }
        s := Student{
            Id:     1,
            Name:   "john",
            Scores: glist.NewFrom(g.Slice{100, 99, 98}),
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
        "github.com/gogf/gf/container/glist"
    )

    func main() {
        b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
        type Student struct {
            Id     int
            Name   string
            Scores *glist.List
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














