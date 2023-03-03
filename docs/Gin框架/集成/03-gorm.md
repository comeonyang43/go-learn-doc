## 1. 介绍

`gorm`是一个使用`Go`语言编写的`ORM`框架。 它文档齐全，对开发者友好，支持主流数据库。具体使用可参考之前的文章[Go常用包(十九):全功能ORM框架(gorm)](https://mp.weixin.qq.com/s/uQ3KOpRwtwNQoeAjNSne9A)

### 1.1 集成流程

![](https://go.liuqh.icu/img/20210717125819.png)



### 1.2 涉及目录

![](https://go.liuqh.icu/img/20210717135522.png)

## 2. 配置

### 2.1 编辑主配置`./config.yaml`

```yaml
mysql:
  host: 127.0.0.1
  port: 3306
  user: root
  password: root
  database: test
  charset: utf8mb4 # 要支持完整的UTF-8编码,需设置成: utf8mb4
  parseTime: true # 解析time.Time类型
  timeZone: Local # 时区,若设置 Asia/Shanghai,需写成: Asia%2fShanghai
  defaultStringSize: 255 # string 类型字段的默认长度
  disableDatetimePrecision: true # 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
  skipInitializeWithVersion: false # 根据当前 MySQL 版本自动配置
  autoMigrate: true # 开启时，每次服务启动都会根据实体创建/更新表结构
  slowSql: 50ms # 慢sql时间。单位毫秒
  logLevel: info # error、info、warn
  ignoreRecordNotFoundError: true # 是否忽略ErrRecordNotFound(未查到记录错误)
  gorm: # gorm配置项disableForeignKeyConstraintWhenMigrating
    skipDefaultTx: false # 是否跳过默认事务
    tablePrefix: "app_" #表前缀
    singularTable: true # 是否使用单数表名(默认复数)，启用后，User结构体表将是user
    coverLogger: true # 是否覆盖默认logger
    prepareStmt: false # 执行任何SQL时都会创建一个prepared statement并将其缓存，以提高后续的效率
    disableForeignKeyConstraintWhenMigrating: true #在AutoMigrate 或 CreateTable 时，GORM 会自动创建外键约束，若要禁用该特性，可将其设置为 true
```

### 2.2 编辑结构体

**新增`./config/mysql.go`文件:**

```go
/**
 * @Description mysql配置信息
 **/
package config

import "time"

// MySQL信息
type mysql struct {
	Host                      string        `yaml:"host"`
	Port                      string        `yaml:"port"`
	User                      string        `yaml:"user"`
	Password                  string        `yaml:"password"`
	Database                  string        `yaml:"database"`
	Charset                   string        `yaml:"charset"`                   //要支持完整的UTF-8编码,需设置成: utf8mb4
	AutoMigrate               bool          `yaml:"autoMigrate"`               // 初始化时调用数据迁移
	ParseTime                 bool          `yaml:"parseTime"`                 //解析time.Time类型
	TimeZone                  string        `yaml:"timeZone"`                  // 时区,若设置 Asia/Shanghai,需写成: Asia%2fShanghai
	DefaultStringSize         uint          `yaml:"defaultStringSize"`         // string 类型字段的默认长度
	DisableDatetimePrecision  bool          `yaml:"disableDatetimePrecision"`  // 禁用 datetime 精度
	SkipInitializeWithVersion bool          `yaml:"skipInitializeWithVersion"` // 根据当前 MySQL 版本自动配置
	Gorm                      gorm          `yaml:"gorm"`
	SlowSql                   time.Duration `yaml:"slowSql"`                   //慢SQL
	LogLevel                  string        `yaml:"logLevel"`                  // 日志记录级别
	IgnoreRecordNotFoundError bool          `yaml:"ignoreRecordNotFoundError"` // 是否忽略ErrRecordNotFound(未查到记录错误)
}
// gorm 配置信息
type gorm struct {
	SkipDefaultTx   bool   `yaml:"skipDefaultTx"`                            //是否跳过默认事务
	CoverLogger     bool   `yaml:"coverLogger"`                              //是否覆盖默认logger
	PreparedStmt    bool   `yaml:"prepareStmt"`                              // 设置SQL缓存
	CloseForeignKey bool   `yaml:"disableForeignKeyConstraintWhenMigrating"` // 禁用外键约束
	TablePrefix     string `yaml:"tablePrefix"`                              // 表前缀
	SingularTable   bool   `yaml:"singularTable"`                            //是否使用单数表名(默认复数)，启用后，User结构体表将是user
}
```

### 2.3 嵌入主配置

**编辑文件:`./config/app.go`**

```go
...
// ServerConfig 配置信息
type ServerConfig struct {
  ....
	Mysql mysql `yaml:"mysql"` // 嵌入MySQL配置
}
```

### 2.4 定义全局变量

**编辑文件:`./global/global.go`**

```go
// 变量
var (
	...
	GvaMysqlClient *gorm.DB            //Mysql客户端
)
```

## 3.代码实现

### 3.1  集成入口

#### 1. 编辑 `main.go`

```go
func init() {
	...
	// 初始化gorm
	initialize.InitGorm()
}
func main() {
	// 启动服务
	...
}
```

#### 2. `initialize.InitGorm()`

**新增文件:`./initialize/gorm.go`**

```go
// 初始化mysql客户端
func InitGorm() {
	mysqlConfig := global.GvaConfig.Mysql
	// user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=%s&parseTime=%t&loc=%s",
		mysqlConfig.User, mysqlConfig.Password, mysqlConfig.Host, mysqlConfig.Port, mysqlConfig.Database, mysqlConfig.Charset,
		mysqlConfig.ParseTime, mysqlConfig.TimeZone)
	// 设置gorm配置
	gormConfig := &gorm.Config{
		SkipDefaultTransaction: mysqlConfig.Gorm.SkipDefaultTx, //是否跳过默认事务
		// 命名策略
		NamingStrategy: schema.NamingStrategy{
			TablePrefix:   mysqlConfig.Gorm.TablePrefix,
			SingularTable: mysqlConfig.Gorm.SingularTable,
		},
		// 执行任何SQL时都会创建一个prepared statement并将其缓存，以提高后续的效率
		PrepareStmt: mysqlConfig.Gorm.PreparedStmt,
		//在AutoMigrate 或 CreateTable 时，GORM 会自动创建外键约束，若要禁用该特性，可将其设置为 true
		DisableForeignKeyConstraintWhenMigrating: mysqlConfig.Gorm.CloseForeignKey,
	}
	// 是否覆盖默认sql配置
	if mysqlConfig.Gorm.CoverLogger {
		setNewLogger(gormConfig)
	}
	client, err := gorm.Open(mysql.New(mysql.Config{
		DSN:                       dsn,
		DefaultStringSize:         mysqlConfig.DefaultStringSize,
		DisableDatetimePrecision:  mysqlConfig.DisableDatetimePrecision,
		SkipInitializeWithVersion: mysqlConfig.SkipInitializeWithVersion,
	}), gormConfig)
	if err != nil {
		panic(fmt.Sprintf("创建mysql客户端失败: %s", err))
	}
	// 赋值给全局变量
	global.GvaMysqlClient = client
	// 是否调用数据迁移
	if mysqlConfig.AutoMigrate {
		core.AutoMigrate()
	}
}
```

### 3.2 重写默认`Logger`

`Gorm` 有一个 [默认 logger 实现](https://github.com/go-gorm/gorm/blob/master/logger/logger.go)，默认情况下，它会打印慢 `SQL` 和错误到控制台，也可以重写覆盖，实现写入到单独文件。

**编辑文件:`./initialize/gorm.go`**

```go
// 设置新的Logger
func setNewLogger(gConfig *gorm.Config) {
	logPath := global.GvaConfig.Log.Path
	file, _ := os.OpenFile(logPath+"/sql.log", os.O_RDWR|os.O_CREATE|os.O_APPEND, os.ModePerm)
	// 日志级别映射 error、info、warn
	logLevelMap := map[string]logger.LogLevel{
		"error": logger.Error,
		"info":  logger.Info,
		"warn":  logger.Warn,
	}
	var logLevel logger.LogLevel
	var ok bool
	if logLevel, ok = logLevelMap[global.GvaConfig.Mysql.LogLevel]; !ok {
		logLevel = logger.Error
	}
	newLogger := logger.New(log.New(file, "\r\n", log.LstdFlags), logger.Config{
		SlowThreshold:             global.GvaConfig.Mysql.SlowSql,                   //慢SQL时间
		LogLevel:                  logLevel,                                         // 记录日志级别
		IgnoreRecordNotFoundError: global.GvaConfig.Mysql.IgnoreRecordNotFoundError, // 是否忽略ErrRecordNotFound(未查到记录错误)
		Colorful:                  false,                                            // 开关颜色
	})
	gConfig.Logger = newLogger
}
```

### 3.3 数据迁移`AutoMigrate `

#### 1. 新增实体

**新增文件:`./model/entity/user.go`**

```go
/**
 * @Description 用户相关的实体
 **/
package entity

import (
	"52lu/go-import-template/global"
)

// 用户表
type User struct {
	global.BaseModel
	NickName string   `json:"nickName" gorm:"type:varchar(20);not null;default:'';comment:昵称"`
	Phone    string   `json:"phone" gorm:"type:char(11);unique:un_phone;comment:手机号"`
	Password string   `json:"password" gorm:"type:varchar(20);comment:密码"`
	Status   int      `json:"status" gorm:"size:4;default:1;comment:状态 1:正常 2:白名单 3:黑名单"`
	UserInfo UserInfo `json:"userInfo" gorm:"-"`
}

// 用户信息表
type UserInfo struct {
	global.BaseModel
	Uid      uint   `json:"uid" gorm:"comment:用户id"`
	Birthday string `json:"birthday" gorm:"type:varchar(10);comment:生日"`
	Address  string `json:"address" gorm:"type:text;comment:地址"`
}
```

#### 2. 迁移代码

**新增文件:`./core/gorm_migrate.go`**

```go
/**
 * @Description mysql迁移
 **/
package core

import (
	"52lu/go-import-template/global"
	"52lu/go-import-template/model/entity"
	"fmt"
	"gorm.io/gorm"
)
// 设置表信息
func setTableOption(tableComment string) *gorm.DB {
	value := fmt.Sprintf("ENGINE=InnoDB COMMENT='%s'", tableComment)
	return global.GvaMysqlClient.Set("gorm:table_options", value)
}
// 用户相关表
func userTable() {
	// 用户账号表
	_ = setTableOption("用户表").AutoMigrate(&entity.User{})
	// 用户信息表
	_ = setTableOption("用户信息表").AutoMigrate(&entity.UserInfo{})
}
// 数据表迁移
func AutoMigrate() {
	// 创建用户相关表
	userTable()
}
```

## 4. 场景示例

下面以登录和注册场景，演示使用和请求流程。

### 4.1 调用流程

![](https://go.liuqh.icu/img/20210719112514.png)

### 4.2 代码实现

#### 1. 编辑api

**新建`./api/v1/user_api.go`**

```go
/**
 * @Description 用户相关接口
 **/
package v1

import (
	"52lu/go-import-template/global"
	"52lu/go-import-template/model/entity"
	"52lu/go-import-template/model/request"
	"52lu/go-import-template/model/response"
	"52lu/go-import-template/service"
	"fmt"
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
)

/**
 * @description:  用户注册
 * @param ctx
 */
func Register(ctx *gin.Context) {
	// 绑定参数
	var registerParam request.RegisterParam
	_ = ctx.ShouldBindJSON(&registerParam)
	// todo 参数校验
	// 调用注册
	register, err := service.Register(registerParam)
	if err != nil {
		response.Fail(ctx,"注册失败!")
		return
	}
	response.OkWithData(ctx,register)
}

/**
 * @description:  用户账号密码登录
 * @param ctx
 */
func Login(ctx *gin.Context) {
	// 绑定参数
	var loginParam request.LoginParam
	_ = ctx.ShouldBindJSON(&loginParam)
	fmt.Println("参数:", loginParam)
	if loginParam.Password == "" || loginParam.Phone == "" {
		response.Fail(ctx, "手机号和密码不能为空！")
		return
	}
	// 调用登录服务
	userRecord := entity.User{Phone: loginParam.Phone, Password: loginParam.Password}
	if err := service.LoginPwd(&userRecord);err != nil {
		global.GvaLogger.Error("登录失败:",zap.Any("user",userRecord))
		response.Fail(ctx,"登录失败,账号或者密码错误!")
		return
	}
	response.OkWithData(ctx, userRecord)
}

```

#### 2. 注册路由

```go
package router
import (
	v1 "52lu/go-import-template/api/v1"
	"github.com/gin-gonic/gin"
)
/**
 * @description: 用户相关的路由
 * @param engine
 */
func InitUserRouter(engine *gin.Engine) {
	// 不需要登录的路由
	noLoginGroup := engine.Group("v1/user")
	{
		// 登录
		noLoginGroup.POST("login", v1.Login)
		// 注册
		noLoginGroup.POST("register", v1.Register)
	}
}
```

#### 3.业务处理

**新建:`./service/user.go`**

```go
/**
 * @Description TODO
 **/
package service

import (
	"52lu/go-import-template/global"
	"52lu/go-import-template/model/entity"
	"52lu/go-import-template/model/request"
	"gorm.io/gorm"
)

/**
 * @description: 账户密码登录
 * @param user
 */
func LoginPwd(user *entity.User) error {
	//校验账户和密码
	result := global.GvaMysqlClient.Where("phone=? and password=?", user.Phone, user.Password).
		First(user)
	return result.Error
}

// 注册用户
func Register(param request.RegisterParam) (*entity.User, error) {
	user := entity.User{
		NickName: param.NickName,
		Phone:    param.Phone,
		Password: param.Password,
	}
	_ = global.GvaMysqlClient.Transaction(func(tx *gorm.DB) error {
		if err := tx.Create(&user).Error; err != nil {
			global.GvaLogger.Sugar().Errorf("新增用户失败: %s", err)
			return err
		}
		userInfo := entity.UserInfo{
			Uid:      user.ID,
			Birthday: param.Birthday,
			Address:  param.Address,
		}
		if err := tx.Create(&userInfo).Error; err != nil {
			global.GvaLogger.Sugar().Errorf("新增用户信息失败: %s", err)
			return err
		}
		user.UserInfo = userInfo
		return nil
	})
	return &user, nil
}
```









