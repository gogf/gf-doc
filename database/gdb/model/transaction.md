# 事务处理

`Model`对象也可以通过`TX`事务对象创建，通过事务对象创建的`Model`对象与通过`DB`数据库对象创建的`Model`对象功能是一样的，只不过前者的所有操作都是基于事务，而当事务提交或者回滚后，对应的`Model`对象不能被继续使用，否则会返回错误。因为该`TX`对象不能被继续使用，一个事务对象仅对应于一个事务流程，`Commit`/`Rollback`后即结束。

使用示例：
```go
func Register() error {
	var r sql.Result
	var err error
	tx, err := g.DB().Begin()
	if err != nil {
		return err
	}
	// 方法退出时检验返回值，
	// 如果结果成功则执行tx.Commit()提交,
	// 否则执行tx.Rollback()回滚操作。
	defer func() {
		if err != nil {
			tx.Rollback()
		} else {
			tx.Commit()
		}
	}()
	// 写入用户基础数据
	r, err = tx.Table("user").Insert(g.Map{
		"name":  "john",
		"score": 100,
		//...
	})
	if err != nil {
		return err
	}
	// 写入用户详情数据，需要用到上一次写入得到的用户uid
	r, err = tx.Table("user_detail").Insert(g.Map{
		"uid":   r.LastInsertId(),
		"phone": "18010576258",
		//...
	})
	if err != nil {
		return err
	}
	return nil
}
```

我们也可以在链式操作中通过`TX`方法切换绑定的事务对象。多次链式操作可以绑定同一个事务对象，在该事务对象中执行对应的链式操作。




