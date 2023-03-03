### 1.介绍

`Zap`是`uber`开源的日志库，支持日志级别分级 、结构化记录，对性能和内存分配做了极致的优化。[目前 Star 12.8 源码地址: https://github.com/uber-go/zap](https://github.com/uber-go/zap)

**官方性能测试图**

![](/img/20210618164446.png)

![](/img/20210618164425.png)

### 2.安装

```go
go get -u go.uber.org/zap
```

### 3.日志记录器

`Zap`提供了两种类型的日志记录器: `SugaredLogger`和`Logger`,两者对比如下：

**`SugaredLogger`** : 在性能很好但不是很关键的上下文中使用,它比其他结构化日志记录包快4-10倍，并且支持结构化和`printf`风格的日志记录。与 `log15`和 `go-kit` 一样，`SugaredLogger` 的结构化日志 `api` 类型灵活，并接受可变的键值对的数量。

**`Logger`** : 在每一微秒和每一次内存分配都很重要的上下文中，使用`Logger`。它甚至比`SugaredLogger`更快，内存分配次数也更少，但它只支持强类型的结构化日志记录。



### 4.创建Logger

#### 4.1 创建Logger几种方式

在`Zap`中通过调用`zap.NewProduction()`、`zap.NewDevelopment()`或者`zap.Example()`可创建一个`Logger`。

他们创建的`Logger`，唯一的区别在于它将记录的信息不同。

使用场景如下:

- `zap.NewProduction()`  : 在生产环境中使用
- `zap.NewDevelopment()` : 在开发环境中使用
- `zap.Example()` : 适合用在测试代码中

#### 4.2 使用示例

##### a.代码
```go
func TestCreateLogger(t *testing.T) {
	// 初始化logger
	logger := zap.NewExample()
	// 使用defer logger.Sync()将缓存同步到文件中。
	defer logger.Sync()
	// 记录日志
	logger.Info("NewExample",
		zap.String("name","张三"),
		zap.Int("age",18),
	)
	productionLogger, _ := zap.NewProduction()
	defer productionLogger.Sync()
	productionLogger.Info("NewProduction",
		zap.String("name","张三"),
		zap.Int("age",18),
	)
	devLogger, _ := zap.NewDevelopment()
	defer devLogger.Sync()
	devLogger.Info("NewDevelopment",
		zap.String("name","张三"),
		zap.Int("age",18),
	)
}
```

##### b. 输出

```go
=== RUN   TestCreateLogger
{"level":"info","msg":"NewExample","name":"张三","age":18}
{"level":"info","ts":1624005421.7035909,"caller":"test/zap_test.go:25","msg":"NewProduction","name":"张三","age":18}
2021-06-18T16:37:01.703+0800	INFO	test/zap_test.go:31	NewDevelopment	{"name": "张三", "age": 18}
--- PASS: TestCreateLogger (0.00s)
PASS
```

> `zap`底层 API 可以设置缓存，所以一般使用`defer logger.Sync()`将缓存同步到文件中

#### 4.3 总结

1. 使用`NewProduction()`记录日志，默认会记录调用函数信息、日期和时间。
2. `NewExample`和`NewProduction()` 默认都是使用`json`格式记录日志，而`NewDevelopment`不是。
3. 默认情况下日志都会打印到应用程序的控制台界面。
4. 记录日志时,尽量调用`zap.T(key,val)`对应的类型方法，这也是`zap`高性能原因的一部分。

### 5.记录日志

#### 5.1 使用默认记录器(`Logger`)

##### a.代码示例

```go
// 使用默认记录日志
func TestRecordLogWithDefault(t *testing.T) {
	// 初始化记录器（使用默认记录器）
	logger := zap.NewExample()
	defer logger.Sync()
	// 记录日志
	logger.Debug("这是debug日志")
	logger.Debug("这是debug日志",zap.String("name","张三"))
    logger.Info("这是info日志",zap.Int("age",18))
	logger.Error("这是error日志",zap.Int("line",130),zap.Error(fmt.Errorf("错误示例")))
	logger.Warn("这是Warn日志")
	// 下面两个都会中断程序
	//logger.Fatal("这是Fatal日志")
	//logger.Panic("这是Panic日志")
}
```

##### b.输出

```bash
=== RUN   TestRecordLogWithDefault
{"level":"debug","msg":"这是debug日志"}
{"level":"debug","msg":"这是debug日志","name":"张三"}
{"level":"info","msg":"这是info日志","age":18}
{"level":"error","msg":"这是error日志","line":130,"error":"错误示例"}
{"level":"warn","msg":"这是Warn日志"}
--- PASS: TestRecordLogWithDefault (0.00s)
```

#### 5.2 使用默认记录器(`Sugar`)

##### a.代码示例

```go
// 使用Sugar记录器
func TestRecordLogWithSuage(t *testing.T) {
	// 初始化记录器
	logger := zap.NewExample()
	// 把日志记录器转成Sugar
	sugarLogger := logger.Sugar()
	defer sugarLogger.Sync()
	// 记录日志
	sugarLogger.Debug("这是debug日志 ",zap.String("name","张三"))
	sugarLogger.Debugf("这是Debugf日志 name:%s ","张三")
	sugarLogger.Info("这是info日志",zap.Int("age",18))
	sugarLogger.Infof("这是Infof日志  内容:%v",map[string]string{"爱好":"动漫"})
	sugarLogger.Error("这是error日志",zap.Int("line",130),zap.Error(fmt.Errorf("错误示例")))
	sugarLogger.Errorf("这是Errorf日志，错误信息：%s","错误报告！")
	sugarLogger.Warn("这是Warn日志")
	sugarLogger.Warnf("这是Warnf日志 %v",[]int{1,2,4,5})
	// 下面两个都会中断程序
	//sugarLogger.Fatal("这是Fatal日志")
	//sugarLogger.Panic("这是Panic日志")
}
```

##### b.输出

```bash
=== RUN   TestRecordLogWithSuage
{"level":"debug","msg":"这是debug日志 {name 15 0 张三 <nil>}"}
{"level":"debug","msg":"这是Debugf日志 name:张三 "}
{"level":"info","msg":"这是info日志{age 11 18  <nil>}"}
{"level":"info","msg":"这是Infof日志  内容:map[爱好:动漫]"}
{"level":"error","msg":"这是error日志{line 11 130  <nil>} {error 26 0  错误示例}"}
{"level":"error","msg":"这是Errorf日志，错误信息：错误报告！"}
{"level":"warn","msg":"这是Warn日志"}
{"level":"warn","msg":"这是Warnf日志 [1 2 4 5]"}
--- PASS: TestRecordLogWithSuage (0.00s)
PASS
```

### 6.定制Logger

除了`zap.NewProduction()`、`zap.NewDevelopment()`、`zap.Example()`还可以通过`zap.New(...)`创建一个`Logger`。


#### 6.1 定制一: 输出到文件

##### a.代码示例

```go
func Test2File(t *testing.T) {
	// 指定写入文件
	fileHandle, _ := os.Create("./test.log")
	writeFile := zapcore.AddSync(fileHandle)
	// 设置日志输出格式为JSON (参数复用NewDevelopmentEncoderConfig)
	encoder := zapcore.NewJSONEncoder(zap.NewDevelopmentEncoderConfig())
	// 返回zapcore.Core,并指定记录zap.DebugLevel级别及以上日志
	zcore := zapcore.NewCore(encoder, zapcore.Lock(writeFile), zap.DebugLevel)
	// 创建日志记录器
	logger := zap.New(zcore)
	defer logger.Sync()
	// 记录日志
	logger.Info("输出日志到文件", zap.String("name", "张三"))
}
```

#### 6.2 定制二: 同时输入文件和控制台

```go
// 同时输入到文件和控制台
func TestPrintFileAndStd(t *testing.T) {
	// 指定写入文件
	fileHandle, _ := os.Create("./test.log")
  // 同时写入文件和控制台 (只修改这一行)
	writeFile := zapcore.NewMultiWriteSyncer(fileHandle,os.Stdout)
	// 设置日志输出格式为JSON (参数复用NewDevelopmentEncoderConfig)
	encoder := zapcore.NewJSONEncoder(zap.NewDevelopmentEncoderConfig())
	// 返回zapcore.Core
	zcore := zapcore.NewCore(encoder, zapcore.Lock(writeFile), zap.DebugLevel)
	// 创建日志记录器
	logger := zap.New(zcore)
	defer logger.Sync()
	// 记录日志
	logger.Info("输出日志到文件", zap.String("name", "张三"))
}
```

### 7.切割日志

`Zap`本身不支持文件切割和日志归档,好在开源强大,贡献出[Lumberjack](https://github.com/natefinch/lumberjack),它是一个`Go`包，用于将日志写入滚动文件。

#### 7.1 安装`Lumberjack`

```go
go get -u github.com/natefinch/lumberjack
```

#### 7.2 集成到Zap

##### a. 代码示例

```go
// 获取文件切割和归档配置信息
func getLumberjackConfig() zapcore.WriteSyncer {
	lumberjackLogger := &lumberjack.Logger{
		Filename: "./zap.log",//日志文件
		MaxSize: 1,//单文件最大容量(单位MB)
		MaxBackups: 3,//保留旧文件的最大数量
		MaxAge: 1,// 旧文件最多保存几天
		Compress: false, // 是否压缩/归档旧文件
	}
	return zapcore.AddSync(lumberjackLogger)
}

// 测试日志切割和归档
func TestCutAndArchive(t *testing.T) {
	// 设置日志输出格式为JSON (参数复用NewDevelopmentEncoderConfig)
	encoder := zapcore.NewJSONEncoder(zap.NewDevelopmentEncoderConfig())
	core := zapcore.NewCore(encoder, getLumberjackConfig(), zap.DebugLevel)
	sugarLogger := zap.New(core).Sugar()
	defer sugarLogger.Sync()
	// 记录日志
	sugarLogger.Infof("日志内容:%s",strings.Repeat("日志",90000))
}
```

##### b. 效果

![](/img/20210621181145.png)

> 在代码中设置单文件最大容量为1MB，如上图所示当文件超过`1MB(527+527 > 1024)`时,则分割。

