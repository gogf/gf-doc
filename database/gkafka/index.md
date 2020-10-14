# gkafka

`gkafka`模块实现了对`kafka`消息队列系统的客户端功能封装，支持`分组消费`及`指定起始位置`等特性，并提供简便易用的API接口。

## 模块安装
```go
go get -u github.com/gogf/gkafka
```
或者使用`go.mod`:
```
require github.com/gogf/gkafka latest
```

## 使用方式
```go
import "github.com/gogf/gkafka"
```

## 接口文档
[godoc.org/github.com/gogf/gkafka](https://pkg.go.dev/github.com/gogf/gkafka)


## 使用示例

### 生产者

```go
package main

import (
    "fmt"
    "github.com/gogf/gkafka"
    "time"
)

func newKafkaClientProducer(topic string) *gkafka.Client {
    kafkaConfig               := gkafka.NewConfig()
    kafkaConfig.Servers        = "localhost:9092"
    kafkaConfig.AutoMarkOffset = false
    kafkaConfig.Topics         = topic
    return gkafka.NewClient(kafkaConfig)
}

func main()  {
    client := newKafkaClientProducer("test")
    defer client.Close()
    for {
        s := time.Now().String()
        fmt.Println("produce:", s)
        if err := client.SyncSend(&gkafka.Message{Value: []byte(s)}); err != nil {
            fmt.Println(err)
        }
        time.Sleep(time.Second)
    }
}
```

### 消费者

```go
package main

import (
    "fmt"
    "github.com/gogf/gkafka"
)

func newKafkaClientConsumer(topic, group string) *gkafka.Client {
    kafkaConfig               := gkafka.NewConfig()
    kafkaConfig.Servers        = "localhost:9092"
    kafkaConfig.AutoMarkOffset = false
    kafkaConfig.Topics         = topic
    kafkaConfig.GroupId        = group
    return gkafka.NewClient(kafkaConfig)
}

func main()  {
    group  := "test-group"
    topic  := "test"
    client := newKafkaClientConsumer(topic, group)
    defer client.Close()
    for {
        if msg, err := client.Receive(); err != nil {
            fmt.Println(err)
            break
        } else {
            fmt.Println("consume:", msg.Partition, msg.Offset, string(msg.Value))
            msg.MarkOffset()
        }
    }
}
```