# 路由注册

这里使用了分组路由的注册方式，分组路由也是推荐的路由注册方式。由于`gf-demos`项目包含其他的示例功能，因此该路由中包含了其他的一些路由注册项，仅供参考。

https://github.com/gogf/gf-demos/blob/master/router/router.go
```go
package router

import (
	"github.com/gogf/gf-demos/app/api/chat"
	"github.com/gogf/gf-demos/app/api/curd"
	"github.com/gogf/gf-demos/app/api/user"
	"github.com/gogf/gf-demos/app/service/middleware"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

// 你可以将路由注册放到一个文件中管理，
// 也可以按照模块拆分到不同的文件中管理，
// 但统一都放到router目录下。
func init() {
	s := g.Server()

	// 某些浏览器直接请求favicon.ico文件，特别是产生404时
	s.SetRewrite("/favicon.ico", "/resource/image/favicon.ico")

	// 分组路由注册方式
	s.Group("/", func(group *ghttp.RouterGroup) {
		ctlChat := new(chat.C)
		ctlUser := new(user.C)
		group.Middleware(middleware.CORS)
		group.ALL("/chat", ctlChat)
		group.ALL("/user", ctlUser)
		group.ALL("/curd/:table", new(curd.C))
		group.Group("/", func(group *ghttp.RouterGroup) {
			group.Middleware(middleware.Auth)
			group.ALL("/user/profile", ctlUser, "Profile")
		})
	})
}
```

可以看到，我们的路由注册管理也使用了包初始化方法`init`，这样做的好处是可以在`router`目录中使用不同的`go`文件注册不同的`init`来分别实现不同的路由注册。当项目的路由比较多的时候，可以采用不同的`go`文件管理不同的路由，这在团队协作的项目中也比较方便。

如果您对`HTTP Method`没有特殊的需求，建议统一使用`ALL`方法注册路由，使得该路由可以被任意的`HTTP Method`访问。特别是在跨域场景中，可能会同一个路由接口需要同时满足`POST`和`OPTIONS`请求，这种情况只有使用`ALL`方法注册路由。

该路由的配置中，所有接口均绑定了`middleware.CORS`允许跨域请求的中间件。而只有`/user/profile`路由需要鉴权控制。


