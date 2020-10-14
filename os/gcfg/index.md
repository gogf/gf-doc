[TOC]


`GF`的配置管理由`gcfg`模块实现，`gcfg`模块是并发安全的，仅提供配置文件读取功能，不提供数据写入/修改功能，**支持的数据文件格式包括： `JSON`、`XML`、`YAML/YML`、`TOML`、`INI`**，项目中开发者可以灵活地选择自己熟悉的配置文件格式来进行配置管理。

**使用方式**：
```go
import "github.com/gogf/gf/os/gcfg"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/os/gcfg


# 配置管理

## 配置文件

默认读取的配置文件为`config.toml`，`toml`类型文件也是默认的、推荐的配置文件格式（语法参考【[TOML格式](os/gcfg/toml.md)】章节），如果想要自定义文件格式，可以通过`SetFileName`方法修改默认读取的配置文件名称（如：`config.json`, `cfg.yaml`, `cfg.xml`, `cfg.ini`等等）。

例如，我们可以通过以下方式读取`config.json`配置文件中的数据库`database`配置项。

```go
// 设置默认配置文件，默认的 config.toml 将会被覆盖
g.Cfg().SetFileName("config.json")
// 后续读取时将会读取到 config.json 配置文件内容，
g.Cfg().Get("database")
```

我们有两种方式进行配置文件的管理，使用全局的`g.Cfg()`获取单例对象（推荐），或者单独使用`gcfg`模块进行管理。

示例配置文件 `config.toml`：
```toml
# 模板引擎目录
viewpath = "/home/www/templates/"
# MySQL数据库配置
[database]
    [[database.default]]
        host     = "127.0.0.1"
        port     = "3306"
        user     = "root"
        pass     = "123456"
        name     = "test1"
        type     = "mysql"
        role     = "master"
        charset  = "utf8"
        priority = "1"
    [[database.default]]
        host     = "127.0.0.1"
        port     = "3306"
        user     = "root"
        pass     = "123456"
        name     = "test2"
        type     = "mysql"
        role     = "master"
        charset  = "utf8"
        priority = "1"
# Redis数据库配置
[redis]
    disk  = "127.0.0.1:6379,0"
    cache = "127.0.0.1:6379,1"
```
> 注意：以上`toml`配置文件中的密码字段值`123456`使用了双引号进行包含，用以标识该密码字段为字符串类型，防止配置文件读取时自动转换为整型引起歧义。


## 配置读取

`gcfg`包最大的特点是支持按层级获取配置数据，层级访问默认通过英文"`.`"号指定，其中`pattern`参数和【[gjson包](encoding/gjson/index.md)】的`pattern`参数一致。

例如针对以上`config.toml`配置文件内容的层级读取：
```go
// /home/www/templates/
g.Cfg().Get("viewpath")

// 127.0.0.1:6379,1
g.Cfg().Get("redis.cache")

// test2
g.Cfg().Get("database.default.1.name")
```

## 配置目录

### 目录配置方法
`gcfg`配置管理器支持非常灵活的多目录自动搜索功能，通过`SetPath`可以修改目录管理目录为**唯一**的一个目录地址，同时，我们推荐通过`AddPath`方法添加多个搜索目录，配置管理器底层将会按照添加目录的顺序作为优先级进行自动检索。直到检索到一个匹配的文件路径为止，如果在所有搜索目录下查找不到配置文件，那么会返回失败。

### 默认目录配置
`gcfg`配置管理对象初始化时，默认会自动添加以下配置文件搜索目录：
1. **当前工作目录及其下的`config`目录**：例如当前的工作目录为`/home/www`时，将会添加`/home/www`及`/home/www/config`；
1. **当前可执行文件所在目录及其下的`config`目录**：例如二进制文件所在目录为`/tmp`时，将会添加`/tmp`及`/tmp/config`；
1. **当前`main`源代码包所在目录及其下的`config`目录**(仅对源码开发环境有效)：例如`main`包所在目录为`/home/john/workspace/gf-app`时，将会添加`/home/john/workspace/gf-app`及`/home/john/workspace/gf-app/config`；

### 默认目录修改

我们可以通过以下方式修改配置管理器的配置文件搜索目录，配置管理对象将会只在该指定目录执行配置文件检索：
1. 通过配置管理器的`SetPath`方法手动修改；
2. 修改命令行启动参数 - `gf.gcfg.path`；
3. 修改指定的环境变量 - `GF_GCFG_PATH`；

假如我们的执行程序文件为`main`，那么可以通过以下方式修改配置管理器的配置文件目录(Linux下)：

1. **(推荐)通过单例模式**
	```go
    g.Cfg().SetPath("/opt/config")
    ```
3. **通过命令行启动参数**
    ```shell
    ./main --gf.gcfg.path=/opt/config/
    ```
1. **通过环境变量（常用在容器中）**
    * 启动时修改环境变量：
        ```shell
        GF_GCFG_PATH=/opt/config/; ./main
        ```
    * 使用`genv`模块来修改环境变量：
        ```go
        genv.Set("GF_GCFG_PATH", "/opt/config")
        ```


# 注意事项

大家都知道，在`Golang`里面，`map/slice`类型其实是一个"引用类型"（也叫"指针类型"），因此当你对这种类型的变量 键值对/索引项 进行修改时，会同时修改到其对应的底层数据。

从效率上考虑，`gcfg`包某些获取方法返回的数据类型为`map/slice`时，没有对齐做值拷贝，因此当你对返回的数据进行修改时，会同时修改`gcfg`对应的底层数据。

例如：
```go
// For testing/example only.
content := `{"map":{"key":"value"}, "slice":[59,90]}`
gcfg.SetContent(content)
defer gcfg.RemoveContent()

m := g.Cfg().GetMap("map")
fmt.Println(m)

// Change the key-value pair.
m["key"] = "john"

// It changes the underlying key-value pair.
fmt.Println(g.Cfg().GetMap("map"))

s := g.Cfg().GetArray("slice")
fmt.Println(s)

// Change the value of specified index.
s[0] = 100

// It changes the underlying slice.
fmt.Println(g.Cfg().GetArray("slice"))

// output:
// map[key:value]
// map[key:john]
// [59 90]
// [100 90]
```









# 自动检测更新

配置管理器使用了缓存机制，当配置文件第一次被读取后会被缓存到内存中，下一次读取时将会直接从缓存中获取，以提高性能。同时，配置管理器提供了对配置文件的**自动检测更新机制**，当配置文件在外部被修改后，配置管理器能够即时地刷新配置文件的缓存内容。

配置管理器的自动检测更新机制是`gf`框架特有的一大特色。




