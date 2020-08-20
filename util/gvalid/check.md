[TOC]

# 单数据校验 - Check

接口文档：https://godoc.org/github.com/gogf/gf/util/gvalid
```go
func Check(value interface{}, rules string, msgs interface{}, params ...interface{}) *Error
```

单数据校验比较简单，我们来看几个示例。

## 示例1，校验数据长度，使用默认的错误提示
```go
rule := "length:6,16"
if e := gvalid.Check("123456", rule, nil);  e != nil {
    fmt.Println(e.String())
}
if e := gvalid.Check("12345", rule, nil);  e != nil {
    fmt.Println(e.String())
}

// 输出： 字段长度为6到16个字符
```

## 示例2，校验数据类型及大小，并且使用自定义的错误提示
```go
rule := "integer|between:6,16"
msgs := "请输入一个整数|参数大小不对啊老铁"
if e := gvalid.Check(5.66, rule, msgs); e != nil {
    fmt.Println(e.Map())
}

// 输出： map[integer:请输入一个整数 between:参数大小不对啊老铁]
```

可以看到，多个规则以及多个自定义错误提示之间使用英文“|”号进行分割，注意自定义错误提示的顺序和多规则的顺序一一对应。msgs参数除了支持string类型以外，还支持**map[string]string类型**，请看以下例子：
```go
rule := "url|min-length:11"
msgs := map[string]string{
    "url"        : "请输入正确的URL地址",
    "min-length" : "地址长度至少为:min位"
}
if e := gvalid.Check("https://goframeorg", rule, msgs); e != nil {
    fmt.Println(e.Map())
}

// 输出： map[url:请输入正确的URL地址]
```

## 示例3，使用自定义正则校验数据格式，使用默认错误提示
```go
// 参数长度至少为6个数字或者6个字母，但是总长度不能超过16个字符
rule := `regex:\d{6,}|\D{6,}|max-length:16`
if e := gvalid.Check("123456", rule, nil);  e != nil {
    fmt.Println(e.Map())
}
if e := gvalid.Check("abcde6", rule, nil); e != nil {
    fmt.Println(e.Map())
}

// 输出： map[regex:字段值不合法]
```





