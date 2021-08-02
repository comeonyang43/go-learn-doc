## 1.介绍

`redis`官网推荐使用`redigo`(`https://github.com/gomodule/redigo`)，截止到今天`Github Start是8.2k` 但`go-redis(https://github.com/go-redis/redis)`使用的人更多， 并且`go-redis`封装得更好。截止到今天`Github Start 是12.1k`。

### 1.1 集成流程

![](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210726160804.png)



## 2.安装

```go
go get github.com/go-redis/redis/v8
```

## 3. 配置

### 3.1 编辑主配置

**文件位置：`./config.yaml`**

```yaml
app:
  ...
log:
  ...
mysql:
  ...
jwt:
  ...
redis:
  host: 127.0.0.1
  port: 6379
  password: # 密码(可为空)
  defaultDB: 0 # 默认DB
  dialTimeout: 5s #redis连接超时时间.默认5s
```

### 3.2 新增结构体

**文件位置:  `./config/redis.go`**

```go
/**
 * @Description redis配置
 **/
package config

type redis struct {
	Addr        string        `yaml:"addr"`
	Password    string        `yaml:"password"`
	DefaultDB   int           `yaml:"defaultDB"`
	DialTimeout time.Duration `yaml:"dialTimeout"`
}
```

### 3.3 嵌入主配置

**编辑文件:`./config/app.go`**

```go
// ServerConfig 配置信息
type ServerConfig struct {
	...
	Redis redis `yaml:"redis"`
}
```

### 3.4 定义全局变量

**编辑文件:`./global/global.go`**

```go
// 变量
var (
  ...
	GvaRedis *redis.Client //Redis客户端
)
```

## 4. 集成代码


### 3.1  集成入口

**编辑文件:`./main.go`**

```go
func init() {
	...
	// 初始化redis
	initialize.InitRedis()
}
func main() {
	// 启动服务
	...
}
```


### 4.2 创建客户端

**新建文件:`./initialize/redis.go`**

```go
/**
 * @Description 初始化redis
 **/
package initialize
import (
	"52lu/go-import-template/global"
	"context"
	"github.com/go-redis/redis/v8"
)
// 初始化redis客户端
func InitRedis()  {
	// 创建
	redisClient := redis.NewClient(&redis.Options{
		Addr:     global.GvaConfig.Redis.Addr,
		Password: global.GvaConfig.Redis.Password,
		DB:       global.GvaConfig.Redis.DefaultDB,
	})
	// 使用超时上下文，验证redis
	timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), global.GvaConfig.Redis.DialTimeout)
	defer cancelFunc()
	_, err := redisClient.Ping(timeoutCtx).Result()
	if err != nil {
		panic("redis初始化失败! "+err.Error())
	}
	global.GvaRedis = redisClient
}
```

## 5. 简单使用

### 5.1 注册路由

**新增文件:`./router/test_router.go`**

```go
/**
 * @Description 测试路由
 **/
package router
import (
	v1 "52lu/go-import-template/api/v1"
	"github.com/gin-gonic/gin"
)
// 测试路由
func InitTestRouter(engine *gin.Engine) {
	systemRouter := engine.Group("test")
	{
		// redis测试使用
		systemRouter.GET("redis", v1.RdTest)
	}
}
```

### 5.2 绑定方法

**新增文件:`./api/v1/test_api.go`**

```go
// 验证redis
func RdTest(ctx *gin.Context)  {
	method, _ := ctx.GetQuery("type")
	var result string
	var err error
	switch method {
	case "get":
		result, err = global.GvaRedis.Get(ctx, "test").Result()
	case "set":
		result, err = global.GvaRedis.Set(ctx, "test", "hello word!", time.Hour).Result()
	}
	if err != nil {
		response.Error(ctx,err.Error())
		return
	}
	response.OkWithData(ctx,result)
}
```

### 5.3 请求示例

```bash
➜ curl -X GET http://127.0.0.1:8080/test/redis?type=set
{"code":0,"msg":"请求成功","data":"OK"}
➜ curl -X GET http://127.0.0.1:8080/test/redis?type=get
{"code":0,"msg":"请求成功","data":"hello word!"}
```

