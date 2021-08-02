### 1.项目介绍

`Gin`框架学习使用,并实践常用包在`Gin`框架中的集成和使用。[源码地址: https://github.com/52lu/gin-import-template.git]( https://github.com/52lu/gin-import-template.git)

#### 1.1 项目结构

```bash
├── api # 接口
├── config # 配置
├── core # 核心代码
├── global # 全局变量和常量
├── initialize # 初始化相关
├── logs # 日志目录
├── main.go # 启动文件
├── model # 实体
├── router # 路由
    └── middleware #中间件
├── test # 单元测试目录
└── utils # 工具包
```

#### 1.2 集成流程

![集成代码流程](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210706163200.png)

### 2.安装

```bash
# 安装zap
go get -u go.uber.org/zap
# 安装lumberjack(日志切割和打包)
go get -u github.com/natefinch/lumberjack
```

### 3.配置

#### 3.1 编辑`app.yaml`

```yaml
...
log:
  path: ./logs # 日志文件目录
  filePrefix: gin # 日志文件前缀
  fileFormat: 2006-01-02 # 日志文件名格式
  level: debug # 最低记录级别
  writeWay: file # file/console/all
  outFormat: json # json/console
  zap:
    outFormat: json # json/
  lumberJack: # 日志文件切割和压缩
    maxSize: 1 # 单文件最大容量(单位MB)
    maxBackups: 3 # 保留旧文件的最大数量
    maxAge: 30 # 旧文件最多保存几天
    compress: false #是否压缩/归档旧文件
...
```

#### 3.2 新增对应结构体

新建文件:`config/log.go`

```go
package config
// 日志信息
type log struct {
	Path       string     `yaml:"path"`
	Level      string     `yaml:"level"`
	FilePrefix string     `yaml:"filePrefix"`
	FileFormat string     `yaml:"fileFormat"`
	OutFormat  string     `yaml:"outFormat"`
	LumberJack lumberJack `yaml:"lumberJack"`
}
// 日志切割
type lumberJack struct {
	MaxSize    int  `yaml:"maxSize"`    //单文件最大容量(单位MB)
	MaxBackups int  `yaml:"maxBackups"` // 保留旧文件的最大数量
	MaxAge     int  `yaml:"maxAge"`     // 旧文件最多保存几天
	Compress   bool `yaml:"compress"`   // 是否压缩/归档旧文件
}
```

#### 3.3 嵌入主配置

```go
// ServerConfig 配置信息
type ServerConfig struct {
  ....
	Log   log   `yaml:"log"` // 嵌入日志配置
}
```

#### 3.4 定义全局变量`logger`

编辑文件:`global/global.go`

```go
// 变量
var (
	...
	GvaLogger *zap.Logger         // 日志
)
```

### 4. 具体代码

#### 4.1 集成入口(`InitLogger`)

```go
const (
	// 日志输出格式
	outJson = "json"
)
func InitLogger() {
	logConfig := global.GvaConfig.Log
	// 判断日志目录是否存在
	if exist, _ := utils.DirExist(logConfig.Path); !exist {
		_ = utils.CreateDir(logConfig.Path)
	}
	// 设置输出格式
	var encoder zapcore.Encoder
	if logConfig.OutFormat == outJson {
		encoder = zapcore.NewJSONEncoder(getEncoderConfig())
	} else {
		encoder = zapcore.NewConsoleEncoder(getEncoderConfig())
	}
	// 设置日志文件切割
	writeSyncer := zapcore.AddSync(getLumberjackWriteSyncer())
	// 创建NewCore
	zapCore := zapcore.NewCore(encoder, writeSyncer, getLevel())
	// 创建logger
	logger := zap.New(zapCore)
	defer logger.Sync()
	// 赋值给全局变量
	global.GvaLogger = logger
}
```

#### 4.2 获取最低记录级别

```go
// 获取最低记录日志级别
func getLevel() zapcore.Level {
	levelMap := map[string]zapcore.Level{
		"debug":  zapcore.DebugLevel,
		"info":   zapcore.InfoLevel,
		"warn":   zapcore.WarnLevel,
		"error":  zapcore.ErrorLevel,
		"dpanic": zapcore.DPanicLevel,
		"panic":  zapcore.PanicLevel,
		"fatal":  zapcore.FatalLevel,
	}
	if level, ok := levelMap[global.GvaConfig.Log.Level]; ok {
		return level
	}
  // 默认info级别
	return zapcore.InfoLevel
}
```

#### 4.3 自定义日志输出字段

```go
// 自定义日志输出字段
func getEncoderConfig() zapcore.EncoderConfig {
	config := zapcore.EncoderConfig{
		// Keys can be anything except the empty string.
		TimeKey:        "time",
		LevelKey:       "level",
		NameKey:        "logger",
		CallerKey:      "caller",
		FunctionKey:    zapcore.OmitKey,
		MessageKey:     "msg",
		StacktraceKey:  "S",
		LineEnding:     zapcore.DefaultLineEnding,
		EncodeLevel:    zapcore.CapitalLevelEncoder,
		EncodeTime:     getEncodeTime, // 自定义输出时间格式
		EncodeDuration: zapcore.StringDurationEncoder,
		EncodeCaller:   zapcore.ShortCallerEncoder,
	}
	return config
}
// 定义日志输出时间格式
func getEncodeTime(t time.Time, enc zapcore.PrimitiveArrayEncoder) {
	enc.AppendString(t.Format("2006/01/02 - 15:04:05.000"))
}
```

**效果如下:**

```bash
# 默认
{"L":"INFO","T":"2021-07-06T11:59:10.763+0800","M":"Info记录","name":"张三"}
# 自定义后
{"level":"INFO","time":"2021/07/06 - 11:59:29.906","msg":"Info记录","name":"张三"}
```

#### 4.4 日志文件切割

```go
// 获取文件切割和归档配置信息
func getLumberjackWriteSyncer() zapcore.WriteSyncer {
	lumberjackConfig := global.GvaConfig.Log.LumberJack
	lumberjackLogger := &lumberjack.Logger{
		Filename:   getLogFile(),                //日志文件
		MaxSize:    lumberjackConfig.MaxSize,    //单文件最大容量(单位MB)
		MaxBackups: lumberjackConfig.MaxBackups, //保留旧文件的最大数量
		MaxAge:     lumberjackConfig.MaxAge,     // 旧文件最多保存几天
		Compress:   lumberjackConfig.Compress,   // 是否压缩/归档旧文件
	}
	// 设置日志文件切割
	return zapcore.AddSync(lumberjackLogger)
}
// 获取日志文件名
func getLogFile() string {
	fileFormat := time.Now().Format(global.GvaConfig.Log.FileFormat)
	fileName := strings.Join([]string{
		global.GvaConfig.Log.FilePrefix,
		fileFormat,
		"log"}, ".")
	return path.Join(global.GvaConfig.Log.Path, fileName)
}
```

### 5. 测试验证

```go
// Sugar模式
global.GvaLogger.Sugar().Infof("日志写入测试: %v",strings.Repeat("hello",6))
// 默认模式
global.GvaLogger.Info("Info记录",zap.String("name","张三"))
```





