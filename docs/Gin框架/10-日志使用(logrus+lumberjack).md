### 1.框架自带

`gin`框架自带日志记录功能,通过`fmt.Fprint`和`fmt.Fprintf`来记录日志信息，默认是将日志写到终端,可以通过`gin.DefaultWriter`来设置写入文件。

#### 1.1 终端禁用颜色

```go
 gin.DisableConsoleColor()
```

#### 1.2 只写入文件

```go
// 日志记录使用
func main()  {
  // 禁用控制台颜色，将日志写入文件时不需要控制台颜色
	gin.DisableConsoleColor()
	// 记录到指定文件
	f, _ := os.Create("gin.log")
	gin.DefaultWriter = io.MultiWriter(f)
	// 创建容器
	engine := gin.Default()
	engine.GET("/log", func(context *gin.Context) {
		// 记录日志
		fmt.Fprint(gin.DefaultWriter,"[gin-log] 日志测试使用 \n")
		fmt.Fprintf(gin.DefaultWriter, "[gin-log] Method: %v \n", context.Request.Method)
		fmt.Fprintf(gin.DefaultWriter, "[gin-log] Url: %v \n", context.Request.URL)
		fmt.Fprintf(gin.DefaultWriter, "[gin-log] Header: %v \n", context.Request.Header)
		context.JSON(200,gin.H{"msg":"success"})
	})
	// 启动服务
	_ = engine.Run()
}
```

**请求效果**

```bash
## 查看文件内容
➜ cat gin.log
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /log                      --> go-use/practise.printOutFile.func1 (3 handlers)
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[gin-log] 日志测试使用
[gin-log] Method: GET
[gin-log] Url: /log
[gin-log] Header: map[Accept:[text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9] Accept-Encoding:[gzip, deflate, br] Accept-Language:[zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7] Cache-Control:[max-age=0] Connection:[keep-alive] Sec-Ch-Ua:[" Not A;Brand";v="99", "Chromium";v="90", "Google Chrome";v="90"] Sec-Ch-Ua-Mobile:[?0] Sec-Fetch-Dest:[document] Sec-Fetch-Mode:[navigate] Sec-Fetch-Site:[none] Sec-Fetch-User:[?1] Upgrade-Insecure-Requests:[1] User-Agent:[Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36]]
[GIN] 2021/05/25 - 17:59:53 | 200 |     455.954µs |       127.0.0.1 | GET      "/log"
```



#### 1.3 同时写入文件和终端

如果想将日志同时写入文件和终端,只需要修改上述代码中的`io.MultiWriter(f)`即可，修改成以下内容

```go
func main()  {
  ...
	f, _ := os.Create("gin.log")
  	// 同时将日志写入文件和控制台
	gin.DefaultWriter = io.MultiWriter(f,os.Stdout)
	...
	// 启动服务
	_ = engine.Run()
}
```



### 2.集成logrus

`logrus`可能是` Go` 目前最受欢迎的第三方日志库。目前`Github Star = 17.9k`,源码地址:https://github.com/sirupsen/logrus

#### 2.1 安装

```bash
go get -u github.com/sirupsen/logrus
```

#### 2.2 设置属性

创建`logrus_use.go`文件，编写代码如下:

```go
package main
import (
	"github.com/sirupsen/logrus"
	"os"
	"path"
)
var (
	logPath = "./log"
	logFile = "gin.log"
)
var LogInstance = logrus.New()
func init()  {
	// 打开文件
	logFileName := path.Join(logPath, logFile)
	fileWriter, err := os.OpenFile(logFileName, os.O_APPEND|os.O_WRONLY|os.O_CREATE, os.ModePerm)
	if err != nil {
		panic(err)
	}
	// 设置日志输出到文件
	LogInstance.SetOutput(fileWriter)
	// 设置日志输出格式
	LogInstance.SetFormatter(&logrus.JSONFormatter{})
	// 设置日志记录级别
	LogInstance.SetLevel(logrus.DebugLevel)
}
```

#### 2.3 使用

```go
package main
import (
	"github.com/gin-gonic/gin"
	"github.com/sirupsen/logrus"
)
func main() {
	engine := gin.Default()
	engine.GET("/log", func(context *gin.Context) {
		// Warning级别日志
		LogInstance.WithFields(logrus.Fields{
			"Method": context.Request.Method,
		}).Warning("Warning级别日志")

		// Error级别日志
		LogInstance.WithFields(logrus.Fields{
			"param-name": context.DefaultQuery("name",""),
		}).Error("Error级别日志")

		if context.DefaultQuery("key","") == "1" {
			// Fatal级别日志(此类日志会让服务结束)
			LogInstance.WithFields(logrus.Fields{
				"Host": context.Request.Host,
			}).Fatal("Fatal级别日志")
		}

		// info级别日志
		LogInstance.WithFields(logrus.Fields{
			"code": context.Writer.Status(),
			"url" : context.Request.URL.Path,
			"method" : context.Request.Method,
		}).Info("info级别日志")
		context.JSON(200,gin.H{"msg":"success"})
	})
	_ = engine.Run()
}
```

#### 2.4 日志内容

日志文件:`log/gin.log`

##### JSON格式

```json
{"Method":"GET","level":"warning","msg":"Warning级别日志","time":"2021-05-27T16:06:30+08:00"}
{"level":"error","msg":"Error级别日志","param-name":"","time":"2021-05-27T16:06:30+08:00"}
{"code":200,"level":"info","method":"GET","msg":"info级别日志","time":"2021-05-27T16:06:30+08:00","url":"/log"}
```

##### Text格式

```bash
time="2021-05-27T16:35:16+08:00" level=warning msg="Warning级别日志" Method=GET
time="2021-05-27T16:35:16+08:00" level=error msg="Error级别日志" param-name=
time="2021-05-27T16:35:16+08:00" level=info msg="info级别日志" code=200 method=GET url=/log
```



### 3.压缩滚动

[Lumberjack:https://github.com/natefinch/lumberjack](https://github.com/natefinch/lumberjack)是一个`Go`包，用于将日志写入滚动文件。

#### 3.1 安装

```bash
 go get -u github.com/natefinch/lumberjack
```

#### 3.2 导入

```go
import "github.com/natefinch/lumberjack"
```

#### 3.3 使用

修改`logrus_use.go`文件代码

```go
package main
import (
	"github.com/natefinch/lumberjack"
	"github.com/sirupsen/logrus"
	"path"
)
var (
	logPath = "./log"
	logFile = "gin.log"
)
var LogInstance = logrus.New()
// 日志初始化
func init()  {
	// 打开文件
	logFileName := path.Join(logPath, logFile)
	// 使用滚动压缩方式记录日志
	rolling(logFileName)
	// 设置日志输出JSON格式
	//LogInstance.SetFormatter(&logrus.JSONFormatter{})
	LogInstance.SetFormatter(&logrus.TextFormatter{})
	// 设置日志记录级别
	LogInstance.SetLevel(logrus.DebugLevel)
}
// 日志滚动设置
func rolling(logFile string)  {
	// 设置输出
	LogInstance.SetOutput(&lumberjack.Logger{
		Filename:logFile, //日志文件位置
		MaxSize: 1,// 单文件最大容量,单位是MB
		MaxBackups: 3,// 最大保留过期文件个数
		MaxAge: 1 ,// 保留过期文件的最大时间间隔,单位是天
		Compress: true,// 是否需要压缩滚动日志, 使用的 gzip 压缩
	})
}
```

**main.go 代码**

```go
package main
import (
	"github.com/gin-gonic/gin"
	"github.com/sirupsen/logrus"
	"strings"
)
func main() {
	engine := gin.Default()
	engine.GET("/log", func(context *gin.Context) {
		// Warning级别日志
		LogInstance.WithFields(logrus.Fields{
			"Method": context.Request.Method,
		}).Warning("Warning级别日志")

		// Error级别日志
		LogInstance.WithFields(logrus.Fields{
			"param-name": context.DefaultQuery("name",""),
		}).Error("Error级别日志")

		if context.DefaultQuery("key","") == "1" {
			// Fatal级别日志(此类日志会让服务结束)
			LogInstance.WithFields(logrus.Fields{
				"Host": context.Request.Host,
			}).Fatal("Fatal级别日志")
		}

		// info级别日志
		LogInstance.WithFields(logrus.Fields{
			"code": context.Writer.Status(),
			"url" : context.Request.URL.Path,
			"context" : strings.Repeat("测试",50000),// 重复
		}).Info("info级别日志")
		context.JSON(200,gin.H{"msg":"success"})
	})
	_ = engine.Run()
}

```

#### 3.4 效果

![](./img/20210527185343.png)


