### 1.介绍

`logrus`是目前 Github 上 star 数量最多的日志库,截止今天`star数量为17.3k `。`logrus`功能强大,性能高效,而且具有高度灵活性,提供了自定义插件的功能。

[logrus源码:https://github.com/sirupsen/logrus](https://github.com/sirupsen/logrus)

#### 1.特性总览

- 完全兼容` golang` 标准库日志模块，`logrus `拥有六种日志级别：`debug、info、warn、error、fatal 和 panic`，这是` golang` 标准库日志模块的` API `的超集.如果您的项目使用标准库日志模块,完全可以以最低的代价迁移到` logrus` 上。
- 可扩展的 Hook 机制：允许使用者通过 hook 的方式将日志分发到任意地方,如本地文件系统、标准输出、`logstash`、`elasticsearch`或者`mq`等,或者通过 hook 定义日志内容和格式等.
- 可选的日志输出格式：logrus 内置了两种日志格式,`JSONFormatter`和`TextFormatter`,如果这两个格式不满足需求,可以自己动手实现接口 Formatter,来定义自己的日志格式.
- `Field`机制：`logrus`鼓励通过 Field 机制进行精细化的、结构化的日志记录,而不是通过冗长的消息来记录日志.
- `logrus`是一个可插拔的、结构化的日志框架.
- 

### 2.安装

```go
go get github.com/sirupsen/logrus
```



### 3.编写配置

#### 3.1 编写`*.ini`

`app/config/dev.ini`

```ini
[log]
#日志目录
path=/Users/hui/Project/Go/self/goe/log
# 日志级别
level=debug
# 日志格式: json,text,customize
formatter=json
# 输出类型: 1:控制台，2:日志文件
output_type=2
# 打开日志记录的行数；true:开启，false:关闭。默认关闭
report_caller=true
# 日志后缀格式
suffix_format=%Y%m%d
```

#### 3.2 编写配置结构体

```go
/**
 * @description: 日志配置
 * @user: Mr.LiuQH
 */
type LogrusConfig struct {
	Path         string `ini:"path"`
	Level        string `ini:"level"`
	Formatter    string `ini:"formatter"`
	OutputType   string `ini:"output_type"`
	ReportCaller bool   `ini:"report_caller"`
	Suffix       string `ini:"suffix_format"`
}
```

#### 3.3 映射配置

`app/app.go`

```go
// 定义全局变量
var LogrusConfigInstance = &config.LogrusConfig{}
// 把ini配置映射到结构体
func (app *App) loadConfig() {
	...
	// 加载日志配置
	err = cfg.Section("log").MapTo(LogrusConfigInstance)
	if err != nil {
		BusErrorInstance.ThrowError(err)
	}
}
```

#### 3.3 初始化全局Logger

`源文件`:https://github.com/52lu/goe/blob/master/app/initialize.go

```go
func setLoggerInstance() {
	// 设置日志级别
	var level logrus.Level
	err := level.UnmarshalText([]byte(LogrusConfigInstance.Level))
	BusErrorInstance.ThrowError(err)
	LoggerClient.SetLevel(level)
	// 设置日志格式
	if LogrusConfigInstance.Formatter == "json" {
		LoggerClient.SetFormatter(&logrus.JSONFormatter{})
	} else if LogrusConfigInstance.Formatter == "text" {
		LoggerClient.SetFormatter(&logrus.TextFormatter{})
	} else if LogrusConfigInstance.Formatter == "customize" {
		LoggerClient.SetFormatter(&CustomizeFormat{})
	} else {
		BusErrorInstance.ThrowError(errors.New("log formatter must json|text|customize"))
	}
	// 打开日志记录的行数；true:开启，false:关闭。默认关闭
	if LogrusConfigInstance.ReportCaller {
		LoggerClient.SetReportCaller(LogrusConfigInstance.ReportCaller)
	}
	// 设置日志输出方式
	switch LogrusConfigInstance.OutputType {
	case "1":
		// 控制台
		LoggerClient.SetOutput(os.Stdout)
	case "2":
    // 文件(日志分类配置)
		Log2FileByClass()
	default:
		// 默认写到控制台
		LoggerClient.SetOutput(os.Stdout)
	}
}
```

#### 3.4 日志分类和自定义格式

`源文件`:https://github.com/52lu/goe/blob/master/app/initialize.go

```go
/**
 * @Author Mr.LiuQH
 * @Description 日志相关
 * @Date 2021/3/8 2:35 下午
 **/
package common

import (
	"fmt"
	goFileRotatelogs "github.com/lestrrat/go-file-rotatelogs"
	"github.com/rifflock/lfshook"
	"github.com/sirupsen/logrus"
	"path"
	"strings"
	"time"
)

type CustomizeFormat struct {
}

/**
 * @description: 自定义格式输出
 * @user: Mr.LiuQH
 * @receiver c CustomizeFormat
 * @param entry
 * @return []byte
 * @return error
 * @date 2021-03-08 15:53:29
 */
func (c CustomizeFormat) Format(entry *logrus.Entry) ([]byte, error) {
	msg := fmt.Sprintf("[%s] [%s] %s \n",
		time.Now().Local().Format("2006-01-02 15:04:05"),
		strings.ToUpper(entry.Level.String()),
		entry.Message,
	)
	return []byte(msg), nil
}

/**
 * @description: 写入日志并分割
 * @user: Mr.LiuQH
 * @param save
 * @date 2021-03-08 15:16:32
 */
func Log2FileByClass() {
	lfhook := lfshook.NewHook(lfshook.WriterMap{
		logrus.DebugLevel: splitConfig("debug"),
		logrus.InfoLevel:  splitConfig("info"),
		logrus.WarnLevel:  splitConfig("warn"),
		logrus.ErrorLevel: splitConfig("error"),
		logrus.FatalLevel: splitConfig("fatal"),
		logrus.PanicLevel: splitConfig("painc"),
	}, LoggerClient.Formatter)
	LoggerClient.AddHook(lfhook)
}

/**
 * @description: 分割文件配置
 * @user: Mr.LiuQH
 * @param level
 * @return *goFileRotatelogs.RotateLogs
 * @date 2021-03-08 15:49:08
 */
func splitConfig(level string) *goFileRotatelogs.RotateLogs {
	// 拼凑日志名
	logFile := path.Join(LogrusConfigInstance.Path, level)
	logs, err := goFileRotatelogs.New(
		// 文件名
		logFile+"-"+LogrusConfigInstance.Suffix,
		// 生成软链，指向最新日志文件
		goFileRotatelogs.WithLinkName(logFile),
	)
	BusErrorInstance.ThrowError(err)
	return logs
}
```

### 4.使用

`源文件`:https://github.com/52lu/goe/blob/master/app/controllers/v1/log.go

```go
/**
 * @Author Mr.LiuQH
 * @Description 日志集成使用
 * @Date 2021/3/8 2:15 下午
 **/
package v1

import (
	"github.com/sirupsen/logrus"
	. "goe/app/common"
)
type LogController struct {
	BaseController
}
func init() {
	RouteListInstance.AddRoute("v1","log",&LogController{})
}
/**
 * @description: 测试logrus使用
 * @user: Mr.LiuQH
 * @receiver l LogController
 * @return error
 * @date 2021-03-08 14:22:45
 */
func (l LogController) Test() error  {
	// 简短消息记录
	LoggerClient.Trace("这是Trace,日志信息")
	LoggerClient.Debug("这是Debug,日志信息")
	LoggerClient.Info("这是Info,日志信息")
	LoggerClient.Error("这是Error,日志信息")
	LoggerClient.Warn("这是Warn,日志信息")
	//记录结构化数据
	LoggerClient.WithFields(logrus.Fields{
		"uerName":"zhangsan",
		"age":28,
		"pice":500.12,
		"likes":[]string{"游戏","旅游"},
	}).Info("记录结构化数据")
	return  l.Success(nil)
}
```

### 5.效果图

![image-20210308164422910](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/image-20210308164422910.png)


### 6.相关地址
[源代码: https://github.com/52lu/goe](https://github.com/52lu/goe)