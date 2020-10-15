[TOC]

# 查询缓存

`gdb`支持对查询结果的缓存处理，常用于多读少写的查询缓存场景，并支持手动的缓存清理。需要注意的是，查询缓存仅支持链式操作，且在事务操作下不可用。

相关方法：
```go
// 查询缓存/清除缓存操作，需要注意的是，事务查询不支持缓存。
// 当duration < 0时表示清除缓存， duration=0时表示不过期, duration > 0时表示过期时间，duration过期时间单位：秒；
// name表示自定义的缓存名称，便于业务层精准定位缓存项(如果业务层需要手动清理时，必须指定缓存名称)，
// 例如：查询缓存时设置名称，清理缓存时可以给定清理的缓存名称进行精准清理。
func (m *Model) Cache(duration time.Duration, name ... string) *Model
```

# 缓存对象

`ORM`对象默认情况下提供了缓存管理对象，该缓存对象类型为`*gcache.Cache`，也就是说同时也支持`*gcache.Cache`的所有特性。可以通过`GetCache() *gcache.Cache` 接口方法获得该缓存对象，并通过返回的对象实现自定义的各种缓存操作，例如：`g.DB().GetCache().Keys()`。

# 缓存适配（`Redis`缓存）

默认情况下`ORM`的`*gcache.Cache`缓存对象提供的是单进程内存缓存，虽然性能非常高效，但是只能在单进程内使用。如果服务如果采用多节点部署，多节点之间的缓存可能会产生数据不一致的情况，因此大多数场景下我们都是通过`Redis`服务器来实现对数据库查询数据的缓存。`*gcache.Cache`对象采用了适配器设计模式，可以轻松实现从单进程内存缓存切换为分布式的`Redis`缓存，具体请参考：
1. `gcache`模块【[缓存适配](os/gcache/adapter.md)】章节。
1. `Redis`适配器: https://github.com/gogf/gcache-adapter 使用方式请参考仓库`README`介绍。

# 使用示例

## 数据表结构
```sql
CREATE TABLE `user` (
  `uid` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(30) NOT NULL DEFAULT '' COMMENT '昵称',
  `site` varchar(255) NOT NULL DEFAULT '' COMMENT '主页',
  PRIMARY KEY (`uid`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

## 示例代码
```go
package main

import (
	"github.com/gogf/gf/database/gdb"
	"github.com/gogf/gf/frame/g"
	"time"
)

func main() {
	db := g.DB()
	// 开启调试模式，以便于记录所有执行的SQL
	db.SetDebug(true)

	// 执行2次查询并将查询结果缓存1小时，并可执行缓存名称(可选)
	for i := 0; i < 2; i++ {
		r, _ := db.Table("user").Cache(time.Hour, "vip-user").Where("uid", 1).One()
		g.Log().Print(r.Map())
	}

	// 执行更新操作，并清理指定名称的查询缓存
	_, err := db.Table("user").Cache(-1, "vip-user").Data(gdb.Map{"name": "smith"}).Where("uid", 1).Update()
	if err != nil {
		g.Log().Fatal(err)
	}

	// 再次执行查询，启用查询缓存特性
	r, _ := db.Table("user").Cache(time.Hour, "vip-user").Where("uid", 1).One()
	g.Log().Print(r.Map())
}
```
执行后输出结果为（测试表数据结构仅供示例参考）：
```shell
2019-12-28 12:19:57.228 [DEBU] [1 ms] SELECT * FROM `user` WHERE uid=1 LIMIT 1
2019-12-28 12:19:57.228 {"name":"john","site":"https://goframe.org","uid":1}
2019-12-28 12:19:57.228 {"name":"john","site":"https://goframe.org","uid":1}
2019-12-28 12:19:57.299 [DEBU] [1 ms] UPDATE `user` SET `name`='smith' WHERE uid=1
2019-12-28 12:19:57.300 [DEBU] [1 ms] SELECT * FROM `user` WHERE uid=1 LIMIT 1
2019-12-28 12:19:57.300 {"name":"smith","site":"https://goframe.org","uid":1}
```
可以看到：
1. 为了方便展示缓存效果，这里开启了数据`debug`特性，当有任何的SQL操作时将会输出到终端。
1. 执行两次`One`方法数据查询，第一次走了SQL查询，第二次直接使用到了缓存，SQL没有提交到数据库执行，因此这里只打印了一条查询SQL，并且两次查询的结果也是一致的。
1. 注意这里为该查询的缓存设置了一个自定义的名称`vip-user`，以便于后续清空更新缓存。如果缓存不需要清理，那么可以不用设置缓存名称。
1. 当执行`Update`更新操作时，同时根据名称清空指定的缓存。
1. 随后再执行`One`方法数据查询，这时重新缓存新的数据。