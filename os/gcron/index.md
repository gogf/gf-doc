[TOC]

`gcron`模块提供了对定时任务的实现，支持类似`crontab`的配置管理方式，并支持最小粒度到`秒`的定时任务管理。

**使用方式**：
```go
import "github.com/gogf/gf/os/gcron"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/os/gcron


简要说明：
1. `New`方法用于创建自定义的定时任务管理对象；
1. `Add`方法用于添加定时任务，其中：
    - `pattern` 参数使用`CRON语法格式`(具体说明见本章后续相关说明)；
    - `job` 参数为需要执行的任务方法(方法地址)；
    - `name` 为非必需参数，用于给定时任务指定一个**唯一的**名称，注意如果已存在相同名称的任务，那么添加定时任务将会失败；
1. `AddSingleton`方法用于添加单例定时任务，即同时只能有一个该任务正在运行；
1. `AddOnce`方法用于添加只运行一次的定时任务，当运行一次数后该定时任务自动销毁；
1. `AddTimes`方法用于添加运行指定次数的定时任务，当运行`times`次数后该定时任务自动销毁；
1. `Entries`方法用于获取当前所有已注册的定时任务信息；
1. `Remove`方法用于根据名称删除定时任务(停止并删除)；
1. `Search`方法用于根据名称进行定时任务搜索(返回定时任务`*Entry`对象指针)；
1. `Start`方法用于启动定时任务(`Add`后自动启动定时任务), 可通过`name`参数指定需要启动的任务名称；
1. `Stop`方法用于停止定时任务(`Remove`会停止并删除), 可通过`name`参数指定需要停止的任务名称；
1. `Close`方法用于关闭自定义的定时任务管理对象；



# CRON表达格式

`cron表达式`表示一组时间，使用`6`个空格分隔的字段。

```
Field name    | Allowed values  | Allowed special characters
----------    | --------------  | --------------------------
Seconds       | 0-59            | * / , -
Minutes       | 0-59            | * / , -
Hours         | 0-23            | * / , -
Day           | 1-31            | * / , - ?
Month         | 1-12 or JAN-DEC | * / , -
Week          | 0-6 or SUN-SAT  | * / , - ?
```

注意：月份和星期几字段值不区分大小写。 “SUN”，“Sun”，
和“sun”同样被接受。

## 特殊字符

### 星号（`*`）

星号表示cron表达式将匹配所有的值。例如，在第五个字段(`Month`)中使用星号表示每个月。

### 斜线（`/`）

斜杠用于描述范围的增量。例如：第二个字段使用`3-59/15`表示每小时的第3分钟开始到第59分钟，每隔15分钟执行。

### 逗号（`,`）

逗号用于分隔列表的项目。例如，第五个字段使用`MON,WED,FRI`将指每周一，周三和周五执行。

### 连字符（`-`）

连字符用于定义范围。例如，第三个字段使用`9-17`表示每天上午9点至下午5点（含）。

### 问号（`?`）

可以使用`问号`而不是`*`来让`Day`或`Week`字段为空。

### 预定义的时间表

您可以使用几个预定义的时间来代替cron表达式。

```
Entry                  | Description                                | Equivalent To
-----                  | -----------                                | -------------
@yearly (or @annually) | Run once a year, midnight, Jan. 1st        | 0 0 0 1 1 *
@monthly               | Run once a month, midnight, first of month | 0 0 0 1 * *
@weekly                | Run once a week, midnight between Sat/Sun  | 0 0 0 * * 0
@daily (or @midnight)  | Run once a day, midnight                   | 0 0 0 * * *
@hourly                | Run once an hour, beginning of hour        | 0 0 * * * *
```

### 间隔

您还可以定义任务以固定的时间间隔执行，从添加时开始运行。这可以通过格式化`cron`规范来支持，如下所示：
```
@every <duration>
```
其中`duration`是`time.ParseDuration`接受的字符串
（[http://golang.org/pkg/time/#ParseDuration](http://golang.org/pkg/time/#ParseDuration)）。

例如，`@every 1h30m10s`将表示添加任务之后每隔`1小时30分10秒`执行。

> 注意：间隔不会考虑任务的执行时间。例如，如果一项工作需要3分钟才能执行完成，并且计划每隔5分钟运行一次，那么每次任务之间只有2分钟的空闲时间。



## 示例1, 基本使用 

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/os/gcron"
    "github.com/gogf/gf/os/glog"
    "time"
)

func main() {
    gcron.Add("0 30 * * * *", func() { glog.Println("Every hour on the half hour") })
    gcron.Add("* * * * * *",  func() { glog.Println("Every second") }, "second-cron")
    gcron.Add("@hourly",      func() { glog.Println("Every hour") })
    gcron.Add("@every 1h30m", func() { glog.Println("Every hour thirty") })
    g.Dump(gcron.Entries())

    time.Sleep(3*time.Second)

    gcron.Stop("second-cron")

    time.Sleep(3*time.Second)

    gcron.Start("second-cron")

    time.Sleep(3*time.Second)
}
```

执行后，输出结果为：

```html
[
	{
		"Spec": "0 30 * * * *",
		"Cmd": "main.main.func1",
		"Time": "2018-10-25T10:05:28.898768522+08:00",
		"Name": "",
		"Status": {}
	},
	{
		"Spec": "* * * * * *",
		"Cmd": "main.main.func2",
		"Time": "2018-10-25T10:05:28.898773631+08:00",
		"Name": "second-cron",
		"Status": {}
	},
	{
		"Spec": "@hourly",
		"Cmd": "main.main.func3",
		"Time": "2018-10-25T10:05:28.89885461+08:00",
		"Name": "",
		"Status": {}
	},
	{
		"Spec": "@every 1h30m",
		"Cmd": "main.main.func4",
		"Time": "2018-10-25T10:05:28.89885883+08:00",
		"Name": "",
		"Status": {}
	}
]
2018-10-25 10:05:29.000 Every second
2018-10-25 10:05:30.000 Every second
2018-10-25 10:05:31.000 Every second
2018-10-25 10:05:35.000 Every second
2018-10-25 10:05:36.000 Every second
2018-10-25 10:05:37.000 Every second
```
## 示例2, 单例定时任务

单例定时任务，即同时只能有一个该任务正在运行。当第二个相同的定时任务触发执行时，如果发现已有该任务正在执行，第二个任务将会退出不执行，定时器将会继续等待下一次定时任务的触发检测，以此类推。可以使用`AddSingleton`添加单例定时任务。

```go
package main

import (
    "github.com/gogf/gf/os/gcron"
    "github.com/gogf/gf/os/glog"
    "time"
)

func main() {
    gcron.AddSingleton("* * * * * *", func() {
        glog.Println("doing")
        time.Sleep(2*time.Second)
    })
    select { }
}
```
执行后，输出结果为：
```html
2019-01-16 22:49:27.699 doing
2019-01-16 22:49:30.696 doing
2019-01-16 22:49:33.699 doing
...
```

## 示例3, 日志记录功能

`gcron`支持日志记录功能，并可设置日志输出的文件以及级别。默认情况下仅会输出`LEVEL_WARN | LEVEL_ERRO | LEVEL_CRIT`错误级别的日志（包括定时任务运行异常日志），运行日志以`LEVEL_DEBUG`的级别进行记录，因此默认不会记录。
相关方法：
```go
func SetLogPath(path string)
func SetLogLevel(level int)
func GetLogPath() string
func GetLogLevel() int
```

使用示例：
```go
package main

import (
	"github.com/gogf/gf/os/gcron"
	"github.com/gogf/gf/os/glog"
	"time"
)

func main() {
	gcron.SetLogLevel(glog.LEVEL_ALL)
	gcron.Add("* * * * * ?", func() {
		glog.Println("test")
	})
	time.Sleep(3 * time.Second)
}
```
执行后，终端输出结果为：
```html
2019-04-08 23:29:10.119 [DEBU] [gcron] gcron-1(* * * * * ?) main.main.func1 start
2019-04-08 23:29:10.119 test
2019-04-08 23:29:10.120 [DEBU] [gcron] gcron-1(* * * * * ?) main.main.func1 end
2019-04-08 23:29:11.119 [DEBU] [gcron] gcron-1(* * * * * ?) main.main.func1 start
2019-04-08 23:29:11.119 test
2019-04-08 23:29:11.119 [DEBU] [gcron] gcron-1(* * * * * ?) main.main.func1 end
```




