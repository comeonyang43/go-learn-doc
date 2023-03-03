### 1.静态文件服务

#### 1.1 代码

```go
package main
import "github.com/gin-gonic/gin"
func main() {
	engine := gin.Default()
	// 静态图片
	engine.Static("/img","./public/img")
	// 静态文件
	engine.StaticFile("/favicon.ico","./public/favicon.ico")
	// 启动服务
	_ = engine.Run()
}
```

#### 1.2 目录结构

![](https://go.liuqh.icu/img/20210530160003.png)

#### 1.3 请求示例

```bash
# 请求图片
http://127.0.0.1:8080https://go.liuqh.icu/img/b.jpg
http://127.0.0.1:8080https://go.liuqh.icu/img/c.jpg
# 请求文件
http://127.0.0.1:8080/favicon.ico
```

### 2.重定向

```go
package main
import (
	"github.com/gin-gonic/gin"
	"net/http"
)
func main() {
	engine := gin.Default()
	// 重定向到外部(百度)
	engine.GET("/baidu", func(context *gin.Context) {
		context.Redirect(http.StatusMovedPermanently,"http://www.baidu.com")
	})
	// 路由重定向(第一种)
	engine.GET("/route", func(context *gin.Context) {
		context.Request.URL.Path = "/baidu"
		engine.HandleContext(context)
	})

	// 路由重定向(第二种)
	engine.GET("/pos", func(context *gin.Context) {
		context.Redirect(http.StatusTemporaryRedirect,"/get")
	})
	engine.GET("/get", func(context *gin.Context) {
		context.JSON(http.StatusOK,gin.H{"msg":"success"})
	})
	// 启动服务
	_ = engine.Run()
}
```



### 3.修改HTTP配置

#### 3.1 默认启动HTTP服务

```go
func main() {
	engine := gin.Default()
	// 注册路由
	engine.GET("/", func(context *gin.Context) {
		// todo
	})
	// 启动服务
	_ = engine.Run()
}
```

#### 3.2 启动自定义HTTP服务

可以直接使用 `http.ListenAndServe()`启动服务，具体配置如下:

```go
package main
import (
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)
func main() {
	engine := gin.Default()
	// 自定义配置HTTP服务
	serverConfig := &http.Server{
		Addr: ":8080", //ip和端口号
		Handler: engine,//调用的处理器，如为nil会调用http.DefaultServeMux
		ReadTimeout: time.Second,//计算从成功建立连接到request body(或header)完全被读取的时间
		WriteTimeout: time.Second,//计算从request body(或header)读取结束到 response write结束的时间
		MaxHeaderBytes: 1 << 20,//请求头的最大长度，如为0则用DefaultMaxHeaderBytes
	}
	// 注册路由
	engine.GET("/test", func(context *gin.Context) {
		context.JSON(200,gin.H{"msg":"success"})
	})
	// 使用http.ListenAndServe启动服务
	_ = serverConfig.ListenAndServe()
}
```

### 4.运行多个服务

```go
package main

import (
	"golang.org/x/sync/errgroup"
	"log"
	"net/http"
	"github.com/gin-gonic/gin"
	"time"
)
// 通过sync/errgroup来管理多个服务
var (
	g errgroup.Group
)
// 服务1
func serverOne() http.Handler {
	engine := gin.Default()
	engine.GET("/server1", func(context *gin.Context) {
		context.JSON(200,gin.H{"msg":"server1"})
	})
	return engine
}
// 服务2
func serverTwo() http.Handler {
	engine := gin.Default()
	engine.GET("/server2", func(context *gin.Context) {
		context.JSON(200,gin.H{"msg":"server2"})
	})
	return engine
}
func main() {
	// 服务1的配置信息
	s1 := &http.Server{
		Addr: ":8080",
		Handler: serverOne(),
		ReadTimeout: 5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}
	// 服务2的配置信息
	s2 := &http.Server{
		Addr: ":8081",
		Handler: serverTwo(),
		ReadTimeout: 5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}
	// 启动服务
	g.Go(func() error {
		return s1.ListenAndServe()
	})
	g.Go(func() error {
		return s2.ListenAndServe()
	})
	if err := g.Wait();err != nil {
		log.Fatal(err)
	}
}
```

