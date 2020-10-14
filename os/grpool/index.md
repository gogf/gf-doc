
# grpool

Go语言中的`goroutine`虽然相对于系统线程来说比较轻量级（初始栈大小仅`2KB`），但是在高并发量下的`goroutine`频繁创建和销毁对于性能损耗以及`GC`来说压力也不小。充分将`goroutine`复用，减少`goroutine`的创建/销毁的性能损耗，这便是`grpool`对`goroutine`进行池化封装的目的。例如，针对于`100W`个执行任务，使用`goroutine`的话需要不停创建并销毁`100W`个`goroutine`，而使用`grpool`也许底层只需要几万个`goroutine`便能充分复用地执行完成所有任务。

经测试，`goroutine`池对于业务逻辑的执行效率（降低执行时间/CPU使用率）提升不大，甚至没有原生的`goroutine`执行快速（池化`goroutine`执行调度并没有底层go调度器高效，因为池化`goroutine`的执行调度也是基于底层go调度器），但是由于采用了复用的设计，池化后对内存的使用率得到极大的降低。

概念：
1. `Pool`：`goroutine`池，用于管理若干可复用的`goroutine`协程资源；
1. `Worker`：池对象中参与任务执行的`goroutine`，一个`Worker`可以执行若干个`Job`，直到队列中再无等待的`Job`；
1. `Job`：添加到池对象的任务队列中等待执行的任务，是一个`func()`的方法，一个`Job`同时只能被一个`Worker`获取并执行；

**使用方式**：
```go
import "github.com/gogf/gf/os/grpool"
```

**使用场景**：

管理大量异步任务的场景、需要异步协程复用的场景、需要降低内存使用率的场景。

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/os/grpool

```go
func Add(f func()) error
func Jobs() int
func Size() int
type Pool
    func New(limit ...int) *Pool
    func (p *Pool) Add(f func()) error
    func (p *Pool) Cap() int
    func (p *Pool) Close()
    func (p *Pool) IsClosed() bool
    func (p *Pool) Jobs() int
    func (p *Pool) Size() int
```


通过`grpool.New`方法创建一个`goroutine池`对象，参数`limit`为非必需参数，用于限定池中的工作`goroutine`数量，默认为不限制。需要注意的是，任务可以不停地往池中添加，没有限制，但是工作的`goroutine`是可以做限制的。我们可以通过`Size()`方法查询当前的工作`goroutine`数量，使用`Jobs()`方法查询当前池中待处理的任务数量。

同时，为便于使用，`grpool`包提供了默认的`goroutine`池，默认的池对象不限制`goroutine`数量，直接通过`grpool.Add`即可往默认的池中添加任务，任务参数必须是一个 `func()`类型的函数/方法。

## 使用示例

**1、使用默认的goroutine池，限制100个工作goroutine执行1000个任务**

https://github.com/gogf/gf/blob/master/.example/os/grpool/grpool1.go

```go
package main

import (
	"time"
	"fmt"
	"github.com/gogf/gf/os/grpool"
	"github.com/gogf/gf/os/gtimer"
)

func job() {
	time.Sleep(1*time.Second)
}

func main() {
	pool := grpool.New(100)
	for i := 0; i < 1000; i++ {
		pool.Add(job)
	}
	fmt.Println("worker:", pool.Size())
	fmt.Println("  jobs:", pool.Jobs())
	gtimer.SetInterval(time.Second, func() {
		fmt.Println("worker:", pool.Size())
		fmt.Println("  jobs:", pool.Jobs())
		fmt.Println()
	})

	select {}
}
```
这段程序中的任务函数的功能是```sleep 1秒钟```，这样便能充分展示出goroutine数量限制功能。其中，我们使用了```gtime.SetInterval```定时器每隔1秒钟打印出当前默认池中的工作goroutine数量以及待处理的任务数量。

**2、我们再来看一个新手经常容易出错的例子**

https://github.com/gogf/gf/blob/master/.example/os/grpool/grpool2.go

```go
package main

import (
    "fmt"
    "sync"
    "github.com/gogf/gf/os/grpool"
)

func main() {
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        grpool.Add(func() {
            fmt.Println(i)
            wg.Done()
        })
    }
    wg.Wait()
}
```
我们这段代码的目的是要顺序地打印出0-9，然而运行后却输出：
```
10
10
10
10
10
10
10
10
10
10
```
为什么呢？这里的执行结果无论是采用```go```关键字来执行还是```grpool```来执行都是如此。原因是，对于异步线程/协程来讲，函数进行异步执行注册时，该函数并未真正开始执行(注册时只在goroutine的栈中保存了变量i的内存地址)，而一旦开始执行时函数才会去读取变量i的值，而这个时候变量i的值已经自增到了10。
清楚原因之后，改进方案也很简单了，就是在注册异步执行函数的时候，把当时变量i的值也一并传递获取；或者把当前变量i的值赋值给一个不会改变的临时变量，在函数中使用该临时变量而不是直接使用变量i。

改进后的示例代码如下：

1)、使用go关键字

https://github.com/gogf/gf/blob/master/.example/os/grpool/grpool3.go

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(v int){
            fmt.Println(v)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```
执行后，输出结果为：
```
9
0
1
2
3
4
5
6
7
8
```
注意，异步执行时并不会保证按照函数注册时的顺序执行，以下同理。

2)、使用临时变量

https://github.com/gogf/gf/blob/master/.example/os/grpool/grpool4.go

```go
package main

import (
    "fmt"
    "sync"
    "github.com/gogf/gf/os/grpool"
)

func main() {
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        v := i
        grpool.Add(func() {
            fmt.Println(v)
            wg.Done()
        })
    }
    wg.Wait()
}

```
执行后，输出结果为：
```
9
0
1
2
3
4
5
6
7
8
```

这里可以看到，使用grpool进行任务注册时，只能使用func()类型的参数，因此无法在任务注册时把变量i的值注册进去，因此只能采用临时变量的形式来传递当前变量i的值。


**3、最后我们使用程序测试一下grpool和原生的goroutine之间的性能**

1)、grpool
```go
package main

import (
    "fmt"
    "sync"
    "time"
    "github.com/gogf/gf/os/gtime"
    "github.com/gogf/gf/os/grpool"
)

func main() {
    start := gtime.Millisecond()
    wg    := sync.WaitGroup{}
    for i := 0; i < 10000000; i++ {
        wg.Add(1)
        grpool.Add(func() {
            time.Sleep(time.Millisecond)
            wg.Done()
        })
    }
    wg.Wait()
    fmt.Println(grpool.Size())
    fmt.Println("time spent:", gtime.Millisecond() - start)
}
```

2)、goroutine
```go
package main

import (
    "fmt"
    "sync"
    "time"
    "github.com/gogf/gf/os/gtime"
)


func main() {
    start := gtime.Millisecond()
    wg    := sync.WaitGroup{}
    for i := 0; i < 10000000; i++ {
        wg.Add(1)
        go func() {
            time.Sleep(time.Millisecond)
            wg.Done()
        }()
    }
    wg.Wait()
    fmt.Println("time spent:", gtime.Millisecond() - start)
}
```


3)、运行结果比较

测试结果为两个程序各运行3次取平均值。
```shell
grpool:
    goroutine count: 847313
     memory   spent: ~2.1 G
     time     spent: 37792 ms

goroutine:
    goroutine count: 1000W
    memory    spent: ~4.8 GB
    time      spent: 27085 ms
```

可以看到池化过后，执行相同数量的任务，`goroutine`数量减少很多，相对的内存也降低了一倍以上，CPU时间耗时也勉强可以接受。
    
    
