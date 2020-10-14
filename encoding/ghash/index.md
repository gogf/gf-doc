# ghash

常用经典哈希函数Go语言实现，提供`uint32`及`uint64`类型的哈希函数。

**使用方式**：
```go
import "github.com/gogf/gf/encoding/ghash"
```
**接口文档**：

https://godoc.org/github.com/gogf/gf/encoding/ghash


性能基准测试：
```html
goos: linux
goarch: amd64
pkg: github.com/gogf/gf/encoding/ghash
BenchmarkBKDRHash-4     	50000000	        30.2 ns/op
BenchmarkBKDRHash64-4   	50000000	        27.2 ns/op
BenchmarkSDBMHash-4     	30000000	        40.5 ns/op
BenchmarkSDBMHash64-4   	50000000	        43.1 ns/op
BenchmarkRSHash-4       	30000000	        37.8 ns/op
BenchmarkSRSHash64-4    	50000000	        33.5 ns/op
BenchmarkJSHash-4       	50000000	        37.1 ns/op
BenchmarkJSHash64-4     	30000000	        38.2 ns/op
BenchmarkPJWHash-4      	50000000	        33.7 ns/op
BenchmarkPJWHash64-4    	50000000	        33.8 ns/op
BenchmarkELFHash-4      	50000000	        35.8 ns/op
BenchmarkELFHash64-4    	50000000	        32.4 ns/op
BenchmarkDJBHash-4      	50000000	        26.9 ns/op
BenchmarkDJBHash64-4    	50000000	        26.8 ns/op
BenchmarkAPHash-4       	30000000	        49.1 ns/op
BenchmarkAPHash64-4     	30000000	        49.8 ns/op
```