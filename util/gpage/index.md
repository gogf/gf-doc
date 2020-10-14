
[TOC]

分页管理由`gpage`模块实现，`gpage`提供了强大的动态分页及静态分页功能，并且为开发者自定义分页样式提供了极高的灵活度。

**使用方式**：
```go
import "github.com/gogf/gf/util/gpage"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/util/gpage

**分页对象**：
```go
// Page is the pagination implementer.
// All the attributes are public, you can change them when necessary.
type Page struct {
	TotalSize      int    // Total size.
	TotalPage      int    // Total page, which is automatically calculated.
	CurrentPage    int    // Current page number >= 1.
	UrlTemplate    string // Custom url template for page url producing.
	LinkStyle      string // CSS style name for HTML link tag <a>.
	SpanStyle      string // CSS style name for HTML span tag <span>, which is used for first, current and last page tag.
	SelectStyle    string // CSS style name for HTML select tag <select>.
	NextPageTag    string // Tag name for next p.
	PrevPageTag    string // Tag name for prev p.
	FirstPageTag   string // Tag name for first p.
	LastPageTag    string // Tag name for last p.
	PrevBarTag     string // Tag string for prev bar.
	NextBarTag     string // Tag string for next bar.
	PageBarNum     int    // Page bar number for displaying.
	AjaxActionName string // Ajax function name. Ajax is enabled if this attribute is not empty.
}
```

# 创建分页对象

由于分页对象往往是在`Web`服务中使用，因此从框架`v1.12`版本开始，我们提供更加便捷的分页对象创建方式，分页对象集成到了`ghttp.Request`对象上，可以非常方便地通过`Request.GetPage`方法获取分页对象。该方法定义如下：
```go
func (r *Request) GetPage(totalSize, pageSize int) *gpage.Page
```

可以看到，获取分页对象仅需要传递总数量及分页数量即可。当然，分页对象也可以独立使用，由于篇幅有限，我们这里只介绍最常用且最简便的使用方式。

具体使用示例请查看后续章节。

# 预定义分页样式

方法`GetContent`提供了预定义的常见的分页样式，以便于开发者快速使用。当预定义的样式无法满足开发者需求时，开发者可以使用公开的方法来自定义分页样式（或者进行方法重载来实现自定义），也可以使用正则替换指定预定义的分页样式中的部分内容来实现自定义。

具体使用示例请查看后续章节。

# 使用Ajax分页功能

分页对象的`AjaxActionName`属性用于给定一个`Ajax`方法名，用于实现`Ajax`分页，但是需要注意的是，该`Ajax`方法名称需要前后端约定统一，并且该`Ajax`方法只有一个URL参数。以下是一个`Ajax`方法的客户端定义示例：
```javascript
function DoAjax(url) {
	// 这里读取URL的内容并根据业务逻辑进行内容展示
}
```

具体使用示例请查看后续章节。