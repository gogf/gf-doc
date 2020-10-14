# gparser

`gf`框架针对常用的数据格式编码解析，提供了非常强大灵活的功能，由`gparser`模块提供，**支持Go变量(interface{})、Struct、JSON、XML、YAML/YML、TOML数据格式之间的相互转换**，支持按照层级进行数据检索访问、支持运行时动态新增/修改/删除层级变量(并发安全)等特性。`gparser`包使得对于未知数据结构、多维数组结构的访问、操作变得异常的简便。

> 需要说明的是，`gparser`模块底层基于`gjson`模块实现，继承了`gjson`包所有的特性。可以将`gparser`模块当做`gjson`模块的一个别名模块，防止`json`这个名称对开发者造成的混淆。具体请参考【[`gjson`模块文档](/encoding/gjson/index.md)】。

**使用方式**：

```go
import "github.com/gogf/gf/encoding/gparser"
```


## 接口文档

https://godoc.org/github.com/gogf/gf/encoding/gparser

简要说明，
1. ```Load```与```LoadContent```方法支持根据文件及内容，生成```gparser.Parser```对象；
2. ```New```方法支持生成一个空的```gparser.Parser```对象，常用用于动态数据生成；
3. ```New```方法同时也支持按照给定的任意Go变量生成一个```gparser.Parser```对象；
3. ```Get*```相关方法支持按照层级检索数据，```pattern```参数中使用英文"```.```"号区分层级关系；
4. ```Set```方法支持按照层级新增/修改，给定的**变量类型支持任意类型**；
5. ```Remove```方法支持按照层级删除变量，只需要给定```pattern```层级检索参数即可；
5. ```To*```相关方法支持将```gparser.Parser```对象生成为支持的数据格式字符串；
6. ```VarTo*```相关方法支持将**任意的Go变量**直接转换为支持的数据格式字符串；


## 使用示例

1. **数据层级检索**

	示例1，读取JSON：
    ```go
    data :=
        `{
            "users" : {
                    "count" : 100,
                    "list"  : [
                        {"name" : "Ming", "score" : 60},
                        {"name" : "John", "score" : 99.5}
                    ]
            }
        }`

    if p, e := gparser.LoadContent([]byte(data), "json"); e != nil {
        glog.Error(e)
    } else {
        fmt.Println("John Score:", p.GetFloat32("users.list.1.score"))
    }
    ```
    可以看到我们可以通过英文"."号实现非常方便的层级访问，针对于数组列表，索引从0开始，我们也可以通过"."号访问其对应的索引项数据。
    
	示例2，读取XML：
    ```go
    data :=
        `<?xml version="1.0" encoding="UTF-8"?>
         <note>
             <to>Tove</to>
             <from>Jani</from>
             <heading>Reminder</heading>
             <body>Don't forget me this weekend!</body>
         </note>`

    if p, e := gparser.LoadContent([]byte(data), "xml"); e != nil {
        glog.Error(e)
    } else {
        fmt.Println("Heading:", p.GetString("note.heading"))
    }
    ```
    LoadContent方法的第二个参数指定内容的数据类型，可选值为(json,xml,yaml/yml,toml)。其他两种数据类型可自行测试，这里不再赘述。
    
2. **处理键名本身带有层级符号"."的情况**

	当键名和层级在访问时存在pattern同名的情况，当然这并不是什么问题，以下是一个示例。
    ```go
    data :=
        `{
            "users" : {
                "count" : 100
            },
            "users.count" : 101
        }`
    if p, e := gparser.LoadContent([]byte(data), "json"); e != nil {
        glog.Error(e)
    } else {
        p.SetViolenceCheck(true)
        fmt.Println("Users Count:", p.Get("users.count"))
    }
    ```
    运行之后打印出的结果为```101```。当键名存在"."号时，检索优先级：键名->层级，因此并不会引起歧义。
    
    再来看一个例子：
    ```go
    data :=
        `{
            "users" : {
                "count" : {
                    "type1" : 1,
                    "type2" : 2
                },
                "count.type1" : 100
            }
        }`
    if p, e := gparser.LoadContent([]byte(data), "json"); e != nil {
        glog.Error(e)
    } else {
        p.SetViolenceCheck(true)
        fmt.Println("Users Count:", p.Get("users.count.type1"))
        fmt.Println("Users Count:", p.Get("users.count.type2"))
    }
    ```
    执行后，输出结果为：
    ```
    100
    2
    ```
    看到了么，gparser会按照给定pattern对层级进行自动探测，检索时按照键名优先的原则进行匹配，并不会出现歧义冲突。
    
    
3. **运行时动态修改数据**

    ```go
    data :=
        `{
            "users" : {
                "count" : 100
            }
        }`
    if p, e := gparser.LoadContent([]byte(data), "json"); e != nil {
        glog.Error(e)
    } else {
        p.Set("users.count",  2)
        p.Set("users.list",  []string{"John", "小明"})
        c, _ := p.ToJson()
        fmt.Println(string(c))
    }
    ```
    修改count为2，并在users节点下新增增加list节点，节点类型为数组。
    执行后输出结果为：
    ```json
    {"users":{"count":2,"list":["John","小明"]}}
    ```
	gparser包的数据运行时修改特性非常强大，在该特性的支持下，各种数据结构的编码/解析显得异常的灵活方便。
    
4. **运行时动态删除变量**

    我们再来看一个删除变量的例子：
    ```go
    data :=
        `<?xml version="1.0" encoding="UTF-8"?>
         <article>
             <count>10</count>
             <list><title>gf article1</title><content>gf content1</content></list>
             <list><title>gf article2</title><content>gf content2</content></list>
             <list><title>gf article3</title><content>gf content3</content></list>
         </article>`
    if p, e := gparser.LoadContent([]byte(data), "xml"); e != nil {
        glog.Error(e)
    } else {
        p.Remove("article.list.0")
        c, _ := p.ToJson()
        fmt.Println(string(c))
    }
    ```
	以上程序输出结果为：
    ```json
	{"article":{"count":"10","list":[{"content":"gf content2","title":"gf article2"},{"content":"gf content3","title":"gf article3"}]}}
    ```
    可以看到，使用Remove方法可以非常方便地根据pattern参数动态删除变量。在该示例中，我们删除了article.list数组的索引0数据项，并将XML转换为JSON数据格式返回。
    
5. **动态生成指定格式的编码数据**

	我们来动态生成一个XML，先来一个简单一点的。
    ```go
    p := gparser.New(nil)
    p.Set("name",   "john")
    p.Set("age",    18)
    p.Set("scores", map[string]int{
        "语文" : 100,
        "数学" : 100,
        "英语" : 100,
    })
    c, _ := p.ToXmlIndent("simple-xml")
    fmt.Println(string(c))
    ```
    执行后，输出结果为：
    ```xml
    <simple-xml>
        <age>18</age>
        <name>john</name>
        <scores>
            <数学>100</数学>
            <英语>100</英语>
            <语文>100</语文>
        </scores>
    </simple-xml>
    ```
	可以看到，我们直接使用Set方式便创建了一个XML数据格式，根本就不需要struct有木有？！想要struct？当然也可以，请看下面的示例。
    
    
6. **变量与struct相互转换**

	示例1，struct转换为gparser.Parser对象，并转换输出为JSON格式：
    ```go
    type Order struct {
        Id    int      `json:"id"`
        Price float32  `json:"price"`
    }
    p := gparser.New(nil)
    p.Set("orders.list.0", Order{1, 100})
    p.Set("orders.list.1", Order{2, 666})
    p.Set("orders.list.2", Order{3, 999.99})
    fmt.Println("Order 1 Price:", p.Get("orders.list.1.price"))
    c, _ := p.ToJson()
    fmt.Println(string(c))
    ```
    执行后，输出结果为：
    ```
    Order 1 Price: 666
    {"orders":{"list":{"0":{"id":1,"price":100},"1":{"id":2,"price":666},"2":{"id":3,"price":999.99}}}}
    ```
    怎么样，有没有觉得很6？
    
    但是针对于struct处理，这里有一点需要特别说明的是，如果struct中的变量不对外公开，那么该变量同时也不能被gparser通过层级检索方式访问到。由于gparser的底层数据结构采用了json格式，如果struct同时定义了json tag，那么层级检索将会按照json tag进行检索访问，不支持struct的其他tag类型。
    
    示例2，变量转换为struct对象：
    ```go
    type Info struct {
        Name string
        Url  string
    }
    o := Info{}
    p := gparser.New(map[string]string {
        "Name" : "gf",
        "Url"  : "https://gitee.com/johng",
    })
    p.ToStruct(&o)
    fmt.Println("Name:", o.Name)
    fmt.Println("Url :", o.Url)
    ```
    执行后，输出为：
    ```html
    Name: gf
    Url : https://gitee.com/johng
    ```
    当然，也可以直接使用gparser.VarToStruct方法来进行直接转换。

7. **数据格式相互转换**

	由于只是演示数据格式的转换，咱们来个数据结构简单点的：
    ```go
    p := gparser.New(map[string]string{
        "name" : "gf",
        "site" : "https://gitee.com/johng",
    })
    c, _ := p.ToJson()
    fmt.Println("JSON:")
    fmt.Println(string(c))
    fmt.Println("======================")

    fmt.Println("XML:")
    c, _ = p.ToXmlIndent()
    fmt.Println(string(c))
    fmt.Println("======================")

    fmt.Println("YAML:")
    c, _ = p.ToYaml()
    fmt.Println(string(c))
    fmt.Println("======================")

    fmt.Println("TOML:")
    c, _ = p.ToToml()
    fmt.Println(string(c))
    ```
    执行后，输出结果为：
    ```shell
    JSON:
    {"name":"gf","site":"https://gitee.com/johng"}
    ======================
    XML:
    <doc>
        <name>gf</name>
        <site>https://gitee.com/johng</site>
    </doc>
    ======================
    YAML:
    name: gf
    site: https://gitee.com/johng

    ======================
    TOML:
    name = "gf"
    site = "https://gitee.com/johng"
    ```
    可以看到，gparser包使得数据格式的转换变得异常的方便灵活。
    
    
    