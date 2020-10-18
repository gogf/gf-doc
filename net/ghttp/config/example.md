
# 配置示例

所有的配置项请参考`ServerConfig`对象属性：
https://godoc.org/github.com/gogf/gf/net/ghttp#ServerConfig

以下为配置示例文件：
```toml
[server]
    # 基本配置
    Address        = ":80"            # 本地监听地址。默认":80"
	HTTPSAddr      = ":443"           # TLS/HTTPS配置，同时需要配置证书和密钥。默认关闭
	HTTPSCertPath  = ""               # TLS/HTTPS证书文件本地路径，建议使用绝对路径。默认关闭
	HTTPSKeyPath   = ""               # TLS/HTTPS密钥文件本地路径，建议使用绝对路径。默认关闭
	ReadTimeout    = "60s"            # 请求读取超时时间，一般不需要配置。默认为60秒
	WriteTimeout   = "0"              # 数据返回写入超时时间，一般不需要配置。默认不超时（0）
	IdleTimeout    = "60s"            # 仅当Keep-Alive开启时有效，请求闲置时间。默认为60秒
	MaxHeaderBytes = "10240"          # 请求Header大小限制（Byte）。默认为10KB
	KeepAlive      = true             # 是否开启Keep-Alive功能。默认true
	ServerAgent    = "GF HTTP Server" # 服务端Agent信息。默认为"GF HTTP Server"   
    
    # 静态服务配置
	IndexFiles        = ["index.html","index.htm"]   # 自动首页静态文件检索。默认为["index.html", "index.htm"]
	IndexFolder       = false                        # 当访问静态文件目录时，是否展示目录下的文件列表。默认关闭，那么请求将返回403
    ServerRoot        = "/var/www"                   # 静态文件服务的目录根路径，配置时自动开启静态文件服务。默认关闭
	SearchPaths       = ["/home/www","/var/lib/www"] # 提供静态文件服务时额外的文件搜索路径，当根路径找不到时则按照顺序在搜索目录查找。默认关闭
	FileServerEnabled = false                        # 静态文件服务总开关。默认false
	
    # Cookie配置
	CookieMaxAge = "365s" # Cookie有效期。默认为365天
	CookiePath   = "/"    # Cookie有效路径。默认为"/"表示全站所有路径下有效
	CookieDomain = ""     # Cookie有效域名。默认为当前配置Cookie时的域名

	# Sessions配置
	SessionMaxAge       = "24h"            # Session有效期。默认为24小时
	SessionIdName       = "gfsessionid"    # SessionId的键名名称。默认为gfsessionid
	SessionCookieOutput = true             # Session特性开启时，是否将SessionId返回到Cookie中。默认true
	SessionPath         = "/tmp/gsessions" # Session存储的文件目录路径。默认为当前系统临时目录下的gsessions目录
	
    # Logging配置
	LogPath          = ""                 # 日志文件存储目录路径，建议使用绝对路径。默认为空，表示关闭
    LogStdout        = true               # 日志是否输出到终端。默认为true
    ErrorStack       = true               # 当Server捕获到异常时是否记录堆栈信息到日志中。默认为true
    ErrorLogEnabled  = true               # 是否记录异常日志信息到日志中。默认为true
    ErrorLogPattern  = "error-{Ymd}.log"  # 异常错误日志文件格式。默认为"error-{Ymd}.log"
    AccessLogEnabled = false              # 是否记录访问日志。默认为false
    AccessLogPattern = "access-{Ymd}.log" # 访问日志文件格式。默认为"access-{Ymd}.log"
	
    # PProf配置
	PProfEnabled = false # 是否开启PProf性能调试特性。默认为false
	PProfPattern = ""    # 开启PProf时有效，表示PProf特性的页面访问路径，对当前Server绑定的所有域名有效。
    
    # 其他配置
	ClientMaxBodySize = 810241024 # 客户端最大Body上传限制大小，影响文件上传大小(Byte)。默认为8*1024*1024=8MB
	FormParsingMemory = 1048576   # 解析表单时的缓冲区大小(Byte)，一般不需要配置。默认为1024*1024=1MB
	NameToUriType     = 0         # 路由注册中使用对象注册时的路由生成规则。默认为0
	RouteOverWrite    = false     # 当遇到重复路由注册时是否强制覆盖。默认为false，重复路由存在时将会在启动时报错退出
	DumpRouterMap     = true      # 是否在Server启动时打印所有的路由列表。默认为true
	Graceful          = false     # 是否开启平滑重启特性，开启时将会在本地增加10000的本地TCP端口用于进程间通信。默认false
```
