[TOC]

`gfsnotify`能监控指定文件/目录的改变，如文件的增加、删除、修改、重命名等操作。

**使用方式**：
```go
import "github.com/gogf/gf/os/gfsnotify"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/os/gfsnotify


推荐使用`gfsnotify`模块提供的```Add```和```Remove```模块方法，用于添加监控和取消监控。推荐原因见随后章节说明。

此外也可能通过```New```方法创建一个监控管理对象之后再进行监控管理。其中，添加监控的时候需要给定触发监控时的回调函数，参数类型为```*gfsnotify.Event```对象指针。


# 添加监听

```go
package main

import (
    "github.com/gogf/gf/os/gfsnotify"
    "github.com/gogf/gf/os/glog"
)

func main() {
    // /home/john/temp 是一个目录，当然也可以指定文件
    path := "/home/john/temp"
    _, err := gfsnotify.Add(path, func(event *gfsnotify.Event) {
        if event.IsCreate() {
            glog.Println("创建文件 : ", event.Path)
        }
        if event.IsWrite() {
            glog.Println("写入文件 : ", event.Path)
        }
        if event.IsRemove() {
            glog.Println("删除文件 : ", event.Path)
        }
        if event.IsRename() {
            glog.Println("重命名文件 : ", event.Path)
        }
        if event.IsChmod() {
            glog.Println("修改权限 : ", event.Path)
        }
        glog.Println(event)
    })
    
    // 移除对该path的监听
    // gfsnotify.Remove(path)

    if err != nil {
        glog.Fatalln(err)
    } else {
        select {}
    }
}
```
其中`/home/john`参数为一个目录，`gfsnotify.Add`方法默认为**递归监控**，也就是说当目录下的文件(包括子目录下的文件)发生变化时，也会收到文件监控信息回调。

当我们在```/home/john```目录下创建/删除/修改文件时，可以看到`gfsnotify`监控到了文件的修改并输出了对应的事件信息。

# 移除监听

移除监听我们可以使用`Remove`方法，会移除对整个文件/目录的监听。

当对同一个文件/目录存在多个监听回调时，我们可以通过`RemoveCallback`方法移除指定的回调。方法参数`callbackId`是在添加监听时返回的`Callback`对象的唯一ID。

使用示例1：
```go
package main

import (
    "github.com/gogf/gf/os/gfsnotify"
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/os/gtime"
    "time"
)

func main() {
    c1, err := gfsnotify.Add("/home/john/temp/log", func(event *gfsnotify.Event) {
        glog.Println("callback1")
    })
    if err != nil {
        panic(err)
    }
    c2, err := gfsnotify.Add("/home/john/temp/log", func(event *gfsnotify.Event) {
        glog.Println("callback2")
    })
    if err != nil {
        panic(err)
    }
    // 5秒后移除c1的回调函数注册，仅剩c2
    gtime.SetTimeout(5*time.Second, func() {
        gfsnotify.RemoveCallback(c1.Id)
        glog.Println("remove callback c1")
    })
    // 10秒后移除c2的回调函数注册，所有的回调都移除，不再有任何打印信息输出
    gtime.SetTimeout(10*time.Second, func() {
        gfsnotify.RemoveCallback(c2.Id)
        glog.Println("remove callback c2")
    })

    select {}
}
```


使用示例2：
```go
package main

import (
    "github.com/gogf/gf/os/gfsnotify"
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/os/gtime"
    "time"
)

func main() {
    callback, err := gfsnotify.Add("/home/john/temp", func(event *gfsnotify.Event) {
        glog.Println("callback")
    })
    if err != nil {
        panic(err)
    }

    // 在此期间创建文件、目录、修改文件、删除文件

    // 20秒后移除回调函数注册，所有的回调都移除，不再有任何打印信息输出
    gtime.SetTimeout(20*time.Second, func() {
        gfsnotify.RemoveCallback(callback.Id)
        glog.Println("remove callback")
    })

    select {}
}
```




# fs.inotify.max_user_instances与fs.inotify.max_user_watches

在`*nix`系统下，`gfsnotify`模块使用的是系统的`inotify`特性来实现的文件/目录监控，因此该功能在使用时会受到系统的两个内核函数限制：

- `fs.inotify.max_user_instances`：表示当前用户可创建的`inotify`监控实例数量，即`gfsnotify.New`方法创建的`Watcher`对象数量，一个`Watcher`对象对应系统的一个`inotify`实例，系统默认数量为：`128`；

- `fs.inotify.max_user_watches`：表示一个`inotify`实例可添加的监控文件队列大小，往同一个`inotify`添加的监控文件超过该数量限制则会失败，并且会有系统错误日志，系统默认数量往往为：`8192`(有的系统该数值会比较大一些)；

