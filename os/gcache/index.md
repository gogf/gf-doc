
# gcache

`gcache`是提供统一的缓存管理模块，提供了开发者可自定义灵活接入的缓存适配接口，并提供了高速内存缓存适配实现。

**使用方式**：
```go
import "github.com/gogf/gf/os/gcache"
```

**接口文档**： 

https://godoc.org/github.com/gogf/gf/os/gcache

简要介绍：

1. `gcache`默认提供默认的高速内存缓存对象，可以通过包方法操作内存缓存，也可以通过`New`方法创建内存缓存对象。在通过包方法使用缓存功能时，操作的是`gcache`默认提供的一个`gcache.Cache`对象，具有全局性，因此在使用时注意全局键名的覆盖。

1. `gcache`使用的键名类型是`interface{}`，而不是`string`类型，这意味着我们可以使用任意类型的变量作为键名，但大多数时候建议使用`string`或者`[]byte`作为键名，并且统一键名的数据类型，以便维护。

1. `gcache`存储的键值类型是`interface{}`，也就是说可以存储任意的数据类型，当获取数据时返回的也是`interface{}`类型，若需要转换为其他的类型可以通过`gcache`的`Get*`方法便捷获取常见类型。注意，如果您确定知道自己使用的是内存缓存，那么可以直接使用断言方式对返回的`interface{}`变量进行类型转换，否则建议使用`GetVar`方法后通过泛型的对应方法完成类型转换。

1. 另外需要注意的是，`gcache`的缓存过期时间参数`duration`的类型为`time.Duration`类型，在`Set`缓存变量时，如果缓存时间参数`duration = 0`表示不过期，`duration < 0`表示立即过期，`duration > 0`表示超时过期。


