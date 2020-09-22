[TOC]

# 自定义校验规则

虽然`gvalid`已经内置了常见的数十种校验规则，但是在部分业务场景下我们需要自定义校验规则，特别是一些可以重复使用的业务相关的校验规则。当然，`gvalid`如此的强大，她已经为您考虑得如此周全。

校验规则注册：
```go
// RegisterRule registers custom validation rule and function for package.
// It returns error if there's already the same rule registered previously.
func RegisterRule(rule string, f RuleFunc) error
```

校验方法定义：
```go
// RuleFunc is the custom function for data validation.
// The parameter <value> specifies the value for this rule to validate.
// The parameter <message> specifies the custom error message or configured i18n message for this rule.
// The parameter <params> specifies all the parameters that needs. You can ignore parameter <params> if
// you do not really need it in your custom validation rule.
type RuleFunc func(value interface{}, message string, params map[string]interface{}) error
```
简要说明：
1. 您需要按照`RuleFunc`类型的方法定义，实现一个您需要的校验方法，随后使用`RegisterRule`注册到`gvalid`模块中全局管理。该注册逻辑往往是在程序初始化时执行。该方法在对数据进行校验时将会被自动调用，方法返回`nil`表示校验通过，否则应当返回一个非空的`error`类型值。
1. 其中，`value`参数表示被校验的数据值，注意类型是一个`interface{}`，因此您可以传递任意类型的参数，并在校验方法内部按照程序使用约定自行进行转换（往往使用`gconv`模块实现转换）。
1. 其中，`message`参数表示在校验失败后返回的校验错误提示信息，往往在`struct`定义时使用`gvalid tag`来通过标签定义。
1. 其中，`params`参数表示校验时传递的所有参数，例如校验的是一个`map`或者`struct`时，往往在联合校验时有用。

> 注意事项：从性能考虑，自定义规则的注册方法不支持并发调用，您需要在程序启动时进行注册（例如在`boot`包中处理），无法在运行时动态注册，否则会产生并发安全问题。

## 示例1，用户唯一性规则

在用户注册时，我们往往需要校验当前用户提交的名称/账号是否唯一，因此我们可以注册一个`unique-name`的全局规则来实现。

```go
rule := "unique-name"
gvalid.RegisterRule(rule, func(value interface{}, message string, params map[string]interface{}) error {
	var (
		id   = gconv.Int(params["Id"])
		name = gconv.String(value)
	)
	n, err := g.Table("user").Where("id != ? and name = ?", id, name).Count()
	if err != nil {
		return err
	}
	if n > 0 {
		return errors.New(message)
	}
	return nil
})
type User struct {
	Id   int
	Name string `v:"required|unique-name # 请输入用户名称|用户名称已被占用"`
	Pass string `v:"required|length:6,18"`
}
user := &User{
	Id:   1,
	Name: "john",
	Pass: "123456",
}
err := gvalid.CheckStruct(user, nil)
fmt.Println(err.Error())
// Output:
// 用户名称已被占用
```

## 示例2，自定义实现`required`规则

默认情况下，`gvalid`的内置规则是不支持`map`、`slice`类型的`required`规则校验，因此我们可以自行实现，然后覆盖原有规则。其他的规则也可以按照此逻辑自定义覆盖。
```go
rule := "required"
gvalid.RegisterRule(rule, func(value interface{}, message string, params map[string]interface{}) error {
    reflectValue := reflect.ValueOf(value)
    if reflectValue.Kind() == reflect.Ptr {
        reflectValue = reflectValue.Elem()
    }
    isEmpty := false
    switch reflectValue.Kind() {
    case reflect.Bool:
        isEmpty = !reflectValue.Bool()
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        isEmpty = reflectValue.Int() == 0
    case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        isEmpty = reflectValue.Uint() == 0
    case reflect.Float32, reflect.Float64:
        isEmpty = math.Float64bits(reflectValue.Float()) == 0
    case reflect.Complex64, reflect.Complex128:
        c := reflectValue.Complex()
        isEmpty = math.Float64bits(real(c)) == 0 && math.Float64bits(imag(c)) == 0
    case reflect.String, reflect.Map, reflect.Array, reflect.Slice:
        isEmpty = reflectValue.Len() == 0
    }
    if isEmpty {
        return errors.New(message)
    }
    return nil
})
fmt.Println(gvalid.Check("", "required", "It's required"))
fmt.Println(gvalid.Check([]string{}, "required", "It's required"))
fmt.Println(gvalid.Check(map[string]int{}, "required", "It's required"))
gvalid.DeleteRule(rule)
fmt.Println("rule deleted")
fmt.Println(gvalid.Check("", "required", "It's required"))
fmt.Println(gvalid.Check([]string{}, "required", "It's required"))
fmt.Println(gvalid.Check(map[string]int{}, "required", "It's required"))
// Output:
// It's required
// It's required
// It's required
// rule deleted
// It's required
```










