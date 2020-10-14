# gflock

文件锁常用于多进程的互斥锁操作，当一个进程对指定文件锁进程`Lock`之后，其他进程将会被阻塞等待，直到该文件锁被同一个进程`Unlock`(或者对应进程退出)。同时，`gflock`也支持类`TryLock/TryRLock`特性。`gflock`的文件锁特性是依靠底层系统提供的文件锁API实现的。

> 注意文件锁是针对于多进程互斥操作，而不是单进程多协程互斥。在多进程需要控制业务互斥操作的时候特别有用，例如多进程的定时任务管理场景。

**安装模块**：
```go
go get -u github.com/gogf/gflock
```

**使用方式**：
```go
import "github.com/gogf/gflock"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gflock


方法比较实用也很简单，我们这里来展示一个`Lock/Unlock`的示例。

```go
package main

import (
    "time"
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/os/gproc"
    "github.com/gogf/gflock"
)

func main() {
    l := gflock.New("demo.lock")
    l.Lock()
    glog.Printf("locked by pid: %d", gproc.Pid())
    time.Sleep(10*time.Second)
    l.UnLock()
    glog.Printf("unlocked by pid: %d", gproc.Pid())
}
```
我们将这个程序运行两次，会生成两个进程，进程ID分别为`25694`及`25737`(通过`ps`命令或者进程管理器查看)。两个进程都会操作同一个文件锁`demo.lock`文件，但是只有第一个进程`25694`得到了该文件锁操作权限，第二个进程`25737`只有等待该文件锁释放之后才能进一步操作，因此被阻塞。程序中的`Sleep`操作正是为了演示这一阻塞结果而特意设定的。当第一个进程`25694`释放文件锁之后，第二个进程`25737`将会立即开始执行。最终的执行结果如下（注意打印时间的差异）：

进程1（`25694`）：
```shell
2018-05-18 13:51:31.191 locked by pid: 25694
2018-05-18 13:51:34.191 unlocked by pid: 25694
```

进程2（`25737`）：
```shell
2018-05-18 13:51:34.191 locked by pid: 25737
2018-05-18 13:51:37.191 unlocked by pid: 25737
```