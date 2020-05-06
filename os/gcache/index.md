[TOC]


# gcache

`gcache`是一个高速的单进程缓存模块，提供了并发安全的缓存控制接口。

**使用方式**：
```go
import "github.com/gogf/gf/os/gcache"
```

**接口文档**： 

https://godoc.org/github.com/gogf/gf/os/gcache

`gcache`可以使用`New`方法创建使用，并且也可以使用包方法使用。在通过包方法使用缓存功能时，操作的是`gcache`默认提供的一个`gcache.Cache`对象，具有全局性，因此在使用时注意全局键名的覆盖。

`gcache`使用的键名类型是`interface{}`，而不是`string`类型，这意味着我们可以使用任意类型的变量作为键名，但大多数时候建议使用`string`或者`[]byte`作为键名，并且统一键名的数据类型，以便维护。

`gcache`存储的键值类型是`interface{}`，也就是说可以存储任意的数据类型，当获取数据时返回的也是`interface{}`类型，若需要转换为其他的类型可以通过`gcache`的`Get*`方法便捷获取常见类型。

另外需要注意的是，`gcache`的缓存过期时间参数`duration`的类型为`time.Duration`类型，在`Set`缓存变量时，如果缓存时间参数`duration = 0`表示不过期，`duration < 0`表示立即过期，`duration > 0`表示超时过期。



## 示例1，基本使用

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gcache"
)

func main() {
    // 创建一个缓存对象，
    // 当然也可以便捷地直接使用gcache包方法
    c := gcache.New()

    // 设置缓存，不过期
    c.Set("k1", "v1", 0)

    // 获取缓存
    fmt.Println(c.Get("k1"))

    // 获取缓存大小
    fmt.Println(c.Size())

    // 缓存中是否存在指定键名
    fmt.Println(c.Contains("k1"))

    // 删除并返回被删除的键值
    fmt.Println(c.Remove("k1"))

    // 关闭缓存对象，让GC回收资源
    c.Close()
}
```

执行后，输出结果为：

```html
v1
1
true
v1
```


## 示例2，缓存控制

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gcache"
    "time"
)

func main() {
    // 当键名不存在时写入，设置过期时间1000毫秒
    gcache.SetIfNotExist("k1", "v1", 1000*time.Millisecond)

    // 打印当前的键名列表
    fmt.Println(gcache.Keys())

    // 打印当前的键值列表
    fmt.Println(gcache.Values())

    // 获取指定键值，如果不存在时写入，并返回键值
    fmt.Println(gcache.GetOrSet("k2", "v2", 0))

    // 打印当前的键值对
    fmt.Println(gcache.Data())

    // 等待1秒，以便k1:v1自动过期
    time.Sleep(time.Second)

    // 再次打印当前的键值对，发现k1:v1已经过期，只剩下k2:v2
    fmt.Println(gcache.Data())
}
```

执行后，输出结果为：

```html
[k1]
[v1]
v2
map[k1:v1 k2:v2]
map[k2:v2]
```

## 示例3，`GetOrSetFunc`/`GetOrSetFuncLock`

`GetOrSetFunc`获取一个缓存值，当缓存不存在时执行指定的`f func() interface{}`，缓存该`f`方法的结果值，并返回该结果。

需要注意的是，`GetOrSetFunc`的缓存方法参数`f`是在缓存的**锁机制外执行**，因此在`f`内部也可以嵌套调用`GetOrSetFunc`。但如果`f`的执行比较耗时，高并发的时候容易出现`f`被多次执行的情况(缓存设置只有第一个执行的`f`返回结果能够设置成功，其余的被抛弃掉)。

而`GetOrSetFuncLock`的缓存方法`f`是在缓存的**锁机制内执行**，因此可以保证当缓存项不存在时只会执行一次`f`，但是缓存写锁的时间随着`f`方法的执行时间而定。

我们来看一个在`gf-home`项目中使用`GetOrSetFunc`的示例，该示例遍历检索`markdown`文件进行字符串检索，并根据指定的搜索`key`缓存该结果值，因此多次搜索该`key`时，第一次会执行目录遍历搜索，后续将直接使用缓存结果。

https://github.com/gogf/gf-home/blob/master/app/lib/doc/doc.go

```go
// 根据关键字进行markdown文档搜索，返回文档path列表
func SearchMdByKey(key string) []string {
    v := cache.GetOrSetFunc("doc_search_result_" + key, func() interface{} {
        // 当该key的检索缓存不存在时，执行检索
        array    := garray.NewStringArray(0, 0, false)
        docPath  := g.Config().GetString("doc.path")
        paths    := cache.GetOrSetFunc("doc_files_recursive", func() interface{} {
            // 当目录列表不存在时，执行检索
            paths, _ := gfile.ScanDir(docPath, "*.md", true)
            return paths
        }, 0)
        // 遍历markdown文件列表，执行字符串搜索
        for _, path := range gconv.Strings(paths) {
            content := gfcache.GetContents(path)
            if len(content) > 0 {
                if strings.Index(content, key) != -1 {
                    index := gstr.Replace(path, ".md", "")
                    index  = gstr.Replace(index, docPath, "")
                    array.Append(index)
                }
            }
        }
        return array.Slice()
    }, 0)

    return gconv.Strings(v)
}
```

## 示例4，`LRU`缓存淘汰控制

```go
package main

import (
    "github.com/gogf/gf/os/gcache"
    "time"
    "fmt"
)

func main() {
    // 设置LRU淘汰数量
    c := gcache.New(2)

    // 添加10个元素，不过期
    for i := 0; i < 10; i++ {
        c.Set(i, i, 0)
    }
    fmt.Println(c.Size())
    fmt.Println(c.Keys())

    // 读取键名1，保证该键名是优先保留
    fmt.Println(c.Get(1))

    // 等待一定时间后(默认1秒检查一次)，
    // 元素会被按照从旧到新的顺序进行淘汰
    time.Sleep(2*time.Second)
    fmt.Println(c.Size())
    fmt.Println(c.Keys())
}
```

执行后，输出结果为：

```html
10
[2 4 5 7 8 9 0 1 3 6]
1
2
[1 9]
```


## 性能测试

### 测试环境

```shell
CPU: Intel(R) Core(TM) i5-4460  CPU @ 3.20GHz
MEM: 8GB
SYS: Ubuntu 16.04 amd64
```

### 测试结果

```html
john@john-B85M:~/Workspace/Go/GOPATH/src/github.com/gogf/gf/os/gcache$ go test *.go -bench=".*" -benchmem
goos: linux
goarch: amd64
Benchmark_CacheSet-4                       2000000        897 ns/op      249 B/op        4 allocs/op
Benchmark_CacheGet-4                       5000000        202 ns/op       49 B/op        1 allocs/op
Benchmark_CacheRemove-4                   50000000       35.7 ns/op        0 B/op        0 allocs/op
Benchmark_CacheLruSet-4                    2000000        880 ns/op      399 B/op        4 allocs/op
Benchmark_CacheLruGet-4                    3000000        212 ns/op       33 B/op        1 allocs/op
Benchmark_CacheLruRemove-4                50000000       35.9 ns/op        0 B/op        0 allocs/op
Benchmark_InterfaceMapWithLockSet-4        3000000        477 ns/op       73 B/op        2 allocs/op
Benchmark_InterfaceMapWithLockGet-4       10000000        149 ns/op        0 B/op        0 allocs/op
Benchmark_InterfaceMapWithLockRemove-4    50000000       39.8 ns/op        0 B/op        0 allocs/op
Benchmark_IntMapWithLockWithLockSet-4      5000000        304 ns/op       53 B/op        0 allocs/op
Benchmark_IntMapWithLockGet-4             20000000        164 ns/op        0 B/op        0 allocs/op
Benchmark_IntMapWithLockRemove-4          50000000       33.1 ns/op        0 B/op        0 allocs/op
PASS
ok   command-line-arguments 47.503s
```