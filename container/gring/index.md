# gring

支持并发安全开关的环结构，循环双向链表。

**使用场景**：

`ring`这种数据结构在底层开发中用得比较多一些，如：并发锁控制、缓冲区控制。`ring`的特点在于，其必须有固定的大小，当不停地往`ring`中追加写数据时，如果数据大小超过容量大小，新值将会将旧值覆盖。

**使用方式**：
```go
import "github.com/gogf/gf/container/gring"
```

**接口文档**：

https://pkg.go.dev/github.com/gogf/gf/container/gring

> `gring`支持链式操作。



## 使用示例，约瑟夫问题

我们使用`ring`来模拟一下[约瑟夫问题](https://baike.baidu.com/item/%E7%BA%A6%E7%91%9F%E5%A4%AB%E9%97%AE%E9%A2%98/3857719)：

> 著名犹太历史学家 Josephus有过以下的故事：在罗马人占领乔塔帕特后，39 个犹太人与Josephus及他的朋友躲到一个洞中，39个犹太人决定宁愿死也不要被敌人抓到，于是决定了一个自杀方式，41个人排成一个圆圈，由第1个人开始报数，每报数到第3人该人就必须自杀，然后再由下一个重新报数，直到所有人都自杀身亡为止。然而Josephus 和他的朋友并不想遵从。首先从一个人开始，越过k-2个人（因为第一个人已经被越过），并杀掉第k个人。接着，再越过k-1个人，并杀掉第k个人。这个过程沿着圆圈一直进行，直到最终只剩下一个人留下，这个人就可以继续活着。问题是，给定了和，一开始要站在什么地方才能避免被处决？

以下示例为非并发安全场景。

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/container/gring"
)

type Player struct {
    position int  // 位置
    alive    bool // 是否存活
}

const (
    playerCount = 41  // 玩家人数
    startPos    = 1   // 开始报数位置
)

var (
    deadline = 3
)

func main() {
    r := gring.New(playerCount)

    // 设置所有玩家初始值
    for i := 1; i <= playerCount; i++ {
        r.Put(&Player{i, true})
    }

    // 如果开始报数的位置不为1，则设置开始位置
    if startPos > 1 {
        r.Move(startPos - 1)
    }

    counter   := 1  // 报数从1开始，因为下面的循环从第二个开始计算
    deadCount := 0  // 死亡人数，初始值为0

    // 直到所有人都死亡，否则循环一直执行
    for deadCount < playerCount {
        // 跳到下一个人
        r.Next() 

        // 如果是活着的人，则报数
        if r.Val().(*Player).alive {
            counter++
        }

        // 如果报数为deadline，则此人淘汰出局
        if counter == deadline {
            r.Val().(*Player).alive = false
            fmt.Printf("Player %d died!\n", r.Val().(*Player).position)
            deadCount++
            counter = 0
        }
    }
}
```

执行后，输出结果为：
```html
Player 3 died!
Player 6 died!
Player 9 died!
Player 12 died!
Player 15 died!
Player 18 died!
Player 21 died!
Player 24 died!
Player 27 died!
Player 30 died!
Player 33 died!
Player 36 died!
Player 39 died!
Player 1 died!
Player 5 died!
Player 10 died!
Player 14 died!
Player 19 died!
Player 23 died!
Player 28 died!
Player 32 died!
Player 37 died!
Player 41 died!
Player 7 died!
Player 13 died!
Player 20 died!
Player 26 died!
Player 34 died!
Player 40 died!
Player 8 died!
Player 17 died!
Player 29 died!
Player 38 died!
Player 11 died!
Player 25 died!
Player 2 died!
Player 22 died!
Player 4 died!
Player 35 died!
Player 16 died!
Player 31 died!
```

可以看到`16`和`31`是最后两个出队列的，因此Josephus将他的朋友与自己安排在第`16`个与第`31`个位置是安全的。