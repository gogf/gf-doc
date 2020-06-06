
`gdb`链式操作使用方式简单灵活，是`GF`框架官方推荐的数据库操作方式。

# 链式操作

链式操作可以通过数据库对象的`db.Table`/`db.Model`方法或者事务对象的`tx.Table`/`tx.Model`方法，基于指定的数据表返回一个链式操作对象`*Model`，该对象可以执行以下方法。

接口文档：
https://godoc.org/github.com/gogf/gf/database/gdb#Model


```go
func (m *Model) Insert(data ...interface{}) (result sql.Result, err error)
func (m *Model) Replace(data ...interface{}) (result sql.Result, err error)
func (m *Model) Save(data ...interface{}) (result sql.Result, err error)
func (m *Model) Update(dataAndWhere ...interface{}) (result sql.Result, err error)
func (m *Model) Delete(where ...interface{}) (result sql.Result, err error)

func (m *Model) All(where ...interface{} (Result, error)
func (m *Model) One(where ...interface{}) (Record, error)
func (m *Model) Array(fieldsAndWhere ...interface{}) ([]Value, error)
func (m *Model) Value(fieldsAndWhere ...interface{}) (Value, error)
func (m *Model) Count(where ...interface{}) (int, error)

func (m *Model) FindAll(where ...interface{}) (Result, error)
func (m *Model) FindOne(where ...interface{}) (Record, error)
func (m *Model) FindArray(fieldsAndWhere ...interface{}) (Value, error)
func (m *Model) FindValue(fieldsAndWhere ...interface{}) (Value, error)
func (m *Model) FindCount(where ...interface{}) (int, error)

func (m *Model) Struct(pointer interface{}) error
func (m *Model) Structs(pointer interface{}) error
func (m *Model) Scan(pointer interface{}) error

func (m *Model) LeftJoin(table ...string) *Model
func (m *Model) RightJoin(table ...string) *Model
func (m *Model) InnerJoin(table ...string) *Model

func (m *Model) Where(where interface{}, args...interface{}) *Model
func (m *Model) WherePri(where interface{}, args ...interface{}) *Model
func (m *Model) And(where interface{}, args ...interface{}) *Model
func (m *Model) Or(where interface{}, args ...interface{}) *Model

func (m *Model) Group(group string) *Model
func (m *Model) Order(order string) *Model

func (m *Model) Fields(fields string) *Model
func (m *Model) FieldsEx(fields string) *Model
func (m *Model) Data(data...interface{}) *Model
func (m *Model) Batch(batch int) *Model
func (m *Model) Filter() *Model
func (m *Model) Safe(safe...bool) *Model

func (m *Model) DB(db DB) *Model
func (m *Model) TX(tx *TX) *Model

func (m *Model) Master() *Model
func (m *Model) Slave() *Model

func (m *Model) As(as string) *Model
func (m *Model) Chunk(limit int, callback func(result Result, err error) bool)
func (m *Model) Schema(schema string) *Model
func (m *Model) Option(option int) *Model
func (m *Model) OmitEmpty() *Model
func (m *Model) Page(page, limit int) (*Model)
func (m *Model) Offset(offset int) *Model
func (m *Model) Limit(start int, limit int) *Model
```
















