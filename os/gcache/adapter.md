
# 缓存适配

> `v1.14.0`新版本特性。

`gcache`模块采用了适配器设计模式，提供了`Adapter`适配器接口，任何实现了`Adapter`接口的对象均可注册到缓存管理对象中，使得开发者可以对缓存管理对象进行灵活的扩展。

`gcache.Cache`对象结构定义如下：
```go
// Cache struct.
type Cache struct {
	Adapter // Adapter for cache features.
}
```

`Adapter`接口定义如下：

https://godoc.org/github.com/gogf/gf/os/gcache#Adapter

```go
// Adapter is the adapter for cache features implements.
type Adapter interface {
	// Set sets cache with <key>-<value> pair, which is expired after <duration>.
	//
	// It does not expire if <duration> == 0.
	// It deletes the <key> if <duration> < 0.
	Set(key interface{}, value interface{}, duration time.Duration) error

	// Sets batch sets cache with key-value pairs by <data>, which is expired after <duration>.
	//
	// It does not expire if <duration> == 0.
	// It deletes the keys of <data> if <duration> < 0 or given <value> is nil.
	Sets(data map[interface{}]interface{}, duration time.Duration) error

	// SetIfNotExist sets cache with <key>-<value> pair which is expired after <duration>
	// if <key> does not exist in the cache. It returns true the <key> dose not exist in the
	// cache and it sets <value> successfully to the cache, or else it returns false.
	//
	// The parameter <value> can be type of <func() interface{}>, but it dose nothing if its
	// result is nil.
	//
	// It does not expire if <duration> == 0.
	// It deletes the <key> if <duration> < 0 or given <value> is nil.
	SetIfNotExist(key interface{}, value interface{}, duration time.Duration) (bool, error)

	// Get retrieves and returns the associated value of given <key>.
	// It returns nil if it does not exist or its value is nil.
	Get(key interface{}) (interface{}, error)

	// GetOrSet retrieves and returns the value of <key>, or sets <key>-<value> pair and
	// returns <value> if <key> does not exist in the cache. The key-value pair expires
	// after <duration>.
	//
	// It does not expire if <duration> == 0.
	// It deletes the <key> if <duration> < 0 or given <value> is nil, but it does nothing
	// if <value> is a function and the function result is nil.
	GetOrSet(key interface{}, value interface{}, duration time.Duration) (interface{}, error)

	// GetOrSetFunc retrieves and returns the value of <key>, or sets <key> with result of
	// function <f> and returns its result if <key> does not exist in the cache. The key-value
	// pair expires after <duration>.
	//
	// It does not expire if <duration> == 0.
	// It deletes the <key> if <duration> < 0 or given <value> is nil, but it does nothing
	// if <value> is a function and the function result is nil.
	GetOrSetFunc(key interface{}, f func() (interface{}, error), duration time.Duration) (interface{}, error)

	// GetOrSetFuncLock retrieves and returns the value of <key>, or sets <key> with result of
	// function <f> and returns its result if <key> does not exist in the cache. The key-value
	// pair expires after <duration>.
	//
	// It does not expire if <duration> == 0.
	// It does nothing if function <f> returns nil.
	//
	// Note that the function <f> should be executed within writing mutex lock for concurrent
	// safety purpose.
	GetOrSetFuncLock(key interface{}, f func() (interface{}, error), duration time.Duration) (interface{}, error)

	// Contains returns true if <key> exists in the cache, or else returns false.
	Contains(key interface{}) (bool, error)

	// GetExpire retrieves and returns the expiration of <key> in the cache.
	//
	// It returns 0 if the <key> does not expire.
	// It returns -1 if the <key> does not exist in the cache.
	GetExpire(key interface{}) (time.Duration, error)

	// Remove deletes one or more keys from cache, and returns its value.
	// If multiple keys are given, it returns the value of the last deleted item.
	Remove(keys ...interface{}) (value interface{}, err error)

	// Update updates the value of <key> without changing its expiration and returns the old value.
	// The returned value <exist> is false if the <key> does not exist in the cache.
	//
	// It deletes the <key> if given <value> is nil.
	// It does nothing if <key> does not exist in the cache.
	Update(key interface{}, value interface{}) (oldValue interface{}, exist bool, err error)

	// UpdateExpire updates the expiration of <key> and returns the old expiration duration value.
	//
	// It returns -1 and does nothing if the <key> does not exist in the cache.
	// It deletes the <key> if <duration> < 0.
	UpdateExpire(key interface{}, duration time.Duration) (oldDuration time.Duration, err error)

	// Size returns the number of items in the cache.
	Size() (size int, err error)

	// Data returns a copy of all key-value pairs in the cache as map type.
	// Note that this function may leads lots of memory usage, you can implement this function
	// if necessary.
	Data() (map[interface{}]interface{}, error)

	// Keys returns all keys in the cache as slice.
	Keys() ([]interface{}, error)

	// Values returns all values in the cache as slice.
	Values() ([]interface{}, error)

	// Clear clears all data of the cache.
	// Note that this function is sensitive and should be carefully used.
	Clear() error

	// Close closes the cache if necessary.
	Close() error
}
```

适配器的注册方法：
```go
// SetAdapter changes the adapter for this cache.
// Be very note that, this setting function is not concurrent-safe, which means you should not call
// this setting function concurrently in multiple goroutines.
func (c *Cache) SetAdapter(adapter Adapter)
```




具体示例请参考【[Redis缓存](os/gcache/redis.md)】章节。











