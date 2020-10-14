`gudp`模块提供了非常简便易用的`gudp.Conn`链接操作对象。

**使用方式**：
```go
import "github.com/gogf/gf/net/gudp"
```

**接口文档**：
https://pkg.go.dev/github.com/gogf/gf/net/gudp
```go
type Conn
    func NewConn(raddr string, laddr ...string) (*Conn, error)
    func NewConnByNetConn(udp *net.UDPConn) *Conn
    func (c *Conn) Close() error
    func (c *Conn) LocalAddr() net.Addr
    func (c *Conn) Recv(length int, retry ...Retry) ([]byte, error)
    func (c *Conn) RecvPkg(retry ...Retry) (result []byte, err error)
    func (c *Conn) RecvPkgWithTimeout(timeout time.Duration, retry ...Retry) ([]byte, error)
    func (c *Conn) RecvWithTimeout(length int, timeout time.Duration, retry ...Retry) ([]byte, error)
    func (c *Conn) RemoteAddr() net.Addr
    func (c *Conn) Send(data []byte, retry ...Retry) error
    func (c *Conn) SendPkg(data []byte, retry ...Retry) error
    func (c *Conn) SendPkgWithTimeout(data []byte, timeout time.Duration, retry ...Retry) error
    func (c *Conn) SendRecv(data []byte, receive int, retry ...Retry) ([]byte, error)
    func (c *Conn) SendRecvPkg(data []byte, retry ...Retry) ([]byte, error)
    func (c *Conn) SendRecvPkgWithTimeout(data []byte, timeout time.Duration, retry ...Retry) ([]byte, error)
    func (c *Conn) SendRecvWithTimeout(data []byte, receive int, timeout time.Duration, retry ...Retry) ([]byte, error)
    func (c *Conn) SendWithTimeout(data []byte, timeout time.Duration, retry ...Retry) error
    func (c *Conn) SetDeadline(t time.Time) error
    func (c *Conn) SetRecvBufferWait(d time.Duration)
    func (c *Conn) SetRecvDeadline(t time.Time) error
    func (c *Conn) SetSendDeadline(t time.Time) error
```


可以看到，`gudp.Conn`和`gtcp.Conn`的方法非常类似，并且也支持简单协议的消息包方法。

# 基本介绍

`gudp.Conn`的操作绝大部分类似于`gtcp`的操作方式（大部分的方法名称也相同），但由于`UDP`是面向非连接的协议，因此`gudp.Conn`（底层通信端口）也只能完成最多一次数据写入和读取，客户端下一次再与目标服务端进行通信的时候，将需要创建新的`Conn`对象进行通信。

# 使用示例

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/gudp"
	"github.com/gogf/gf/os/gtime"
	"time"
)

func main() {
	// Server
	go gudp.NewServer("127.0.0.1:8999", func(conn *gudp.Conn) {
		defer conn.Close()
		for {
			data, err := conn.Recv(-1)
			if len(data) > 0 {
				if err := conn.Send(append([]byte("> "), data...)); err != nil {
					g.Log().Error(err)
				}
			}
			if err != nil {
				g.Log().Error(err)
			}
		}
	}).Run()

	time.Sleep(time.Second)

	// Client
	for {
		if conn, err := gudp.NewConn("127.0.0.1:8999"); err == nil {
			if b, err := conn.SendRecv([]byte(gtime.Datetime()), -1); err == nil {
				fmt.Println(string(b), conn.LocalAddr(), conn.RemoteAddr())
			} else {
				g.Log().Error(err)
			}
			conn.Close()
		} else {
			g.Log().Error(err)
		}
		time.Sleep(time.Second)
	}
}
```
该示例与`gtcp.Conn`中的通信示例类似，不同的是，客户端与服务端无法保持连接，每次通信都需要创建的新的连接对象进行通信。

执行后，输出结果如下：
```html
> 2018-07-21 23:13:31 127.0.0.1:33271 127.0.0.1:8999
> 2018-07-21 23:13:32 127.0.0.1:45826 127.0.0.1:8999
> 2018-07-21 23:13:33 127.0.0.1:58027 127.0.0.1:8999
> 2018-07-21 23:13:34 127.0.0.1:33056 127.0.0.1:8999
> 2018-07-21 23:13:35 127.0.0.1:39260 127.0.0.1:8999
> 2018-07-21 23:13:36 127.0.0.1:33967 127.0.0.1:8999
> 2018-07-21 23:13:37 127.0.0.1:52359 127.0.0.1:8999
...
```
