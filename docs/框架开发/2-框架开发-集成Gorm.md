### 1.安装

```go
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

### 2.配置相关

#### 2.1 编写配置文件

```ini
[mysql]
host=127.0.01
port=3306
database=test
userName=root
password=root
charset=utf8mb4
# 表前缀
table_pre=app_
# 设置闲置的连接数，默认值为2；
max_idle_conn=10
# 设置最大打开的连接数，默认值为0，表示不限制。
max_open_conn=100
# 设置了连接可复用的最大时间。
max_life_time=1h
# 支持把数据库datetime和date类型转换为golang的time.Time类型
parse_time=true
# 使用系统本地时区
loc=Local
# 连接超时时间
timeout=10s
# 慢sql时间,单位毫秒
slow_sql_time=50ms
# 是否打印SQL
print_sql_log=true
```

#### 2.2 定义Mysql配置结构体

**文件:** `app/config/mysql_config.go`

```go
/**
 * @Author Mr.LiuQH
 * @Description 数据库相关的配置
 **/
package config

/**
 * @description: mysql配置
 * @user: Mr.LiuQH
 */
type MysqlConfig struct {
	Host        string `ini:"host"`
	Port        string `ini:"port"`
	Database    string `ini:"database"`
	UserName    string `ini:"userName"`
	Password    string `ini:"password"`
	Charset     string `ini:"charset"`
	MaxIdleConn int    `ini:"max_idle_conn"`
	MaxOpenConn int    `ini:"max_open_conn"`
	ParseTime   string `ini:"parse_time"`
	Loc         string `ini:"loc"`
	Timeout     string `ini:"timeout"`
	MaxLifeTime string `ini:"max_life_time"`
	TablePre    string `ini:"table_pre"`
	SlowSqlTime string `ini:"slow_sql_time"`
	PrintSqlLog bool `ini:"print_sql_log"`
}
```


### 3.初始化

#### 3.1 定义全局客户端变量

**文件:**`app/common/global.go`

```go
// 全局客户端变量
var (
	GormDBClient *gorm.DB
)
```

#### 3.2 映射配置到结构体

```go
/**
 * @description: 加载配置文件
 * @user: Mr.LiuQH
 * @receiver app
 */
func (app *App) loadConfig() {
	iniPath := ConfigPath + app.Env + ".ini"
	fmt.Println("加载配置文件: " + iniPath)
	cfg, err := ini.Load(iniPath)
	BusErrorInstance.ThrowError(err)
	// 加载mysql配置
	err = cfg.Section("mysql").MapTo(MysqlConfigInstance)
}
```

#### 3.3 连接MySQL

```go
/**
 * @description: 连接mysql
 * @user: Mr.LiuQH
 */
func connectMysql() {
	// 用户名:密码@tcp(IP:port)/数据库?charset=utf8mb4&parseTime=True&loc=Local
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=%s&parseTime=%s&loc=%s",
		MysqlConfigInstance.UserName, MysqlConfigInstance.Password, MysqlConfigInstance.Host, MysqlConfigInstance.Port,
		MysqlConfigInstance.Database, MysqlConfigInstance.Charset, MysqlConfigInstance.ParseTime, MysqlConfigInstance.Loc)

	// 连接额外配置信息
	gormConfig := &gorm.Config{
		NamingStrategy: schema.NamingStrategy{
			TablePrefix:   MysqlConfigInstance.TablePre, //表前缀
			SingularTable: true,//使用单数表名，启用该选项时，`User` 的表名应该是 `user`而不是users
		},
	}
	// 打印SQL设置
	if MysqlConfigInstance.PrintSqlLog {
		slowSqlTime, err := time.ParseDuration(MysqlConfigInstance.SlowSqlTime)
		BusErrorInstance.ThrowError(err)
		loggerNew := logger.New(log.New(os.Stdout, "\r\n", log.LstdFlags), logger.Config{
			SlowThreshold: slowSqlTime, //慢SQL阈值
			LogLevel:      logger.Info,
			Colorful:      true, // 彩色打印开启
		})
		gormConfig.Logger = loggerNew
	}
	// 建立连接
	GormDBClient, err = gorm.Open(mysql.Open(dataSourceName), gormConfig)
	BusErrorInstance.ThrowError(err)
	// 设置连接池信息
	db, err2 := GormDBClient.DB()
	BusErrorInstance.ThrowError(err2)
	// 设置空闲连接池中连接的最大数量
	db.SetMaxIdleConns(MysqlConfigInstance.MaxIdleConn)
	// 设置打开数据库连接的最大数量
	db.SetMaxOpenConns(MysqlConfigInstance.MaxOpenConn)
	// 设置了连接可复用的最大时间
	duration, err := time.ParseDuration(MysqlConfigInstance.MaxLifeTime)
	BusErrorInstance.ThrowError(err)
	db.SetConnMaxLifetime(duration)
	// 打印SQL配置信息
	//marshal, _ := json.Marshal(db.Stats())
	//fmt.Printf("数据库配置: %s \n", marshal)
}
```

### 4.创建模型
#### 4.1 创建父模型

**文件:** `app/common/base_model.go`

```go
/**
 * @Author Mr.LiuQH
 * @Description 模型父类
 **/
package common
import (
	"gorm.io/gorm"
)
type BaseModel struct {
	ID        uint `gorm:"primaryKey;autoincrement;not null" json:"id"`
	CreatedAt int `json:"createdAt"`
	UpdatedAt int `json:"updatedAt"`
	DeletedAt gorm.DeletedAt `gorm:"index" json:"deletedAt"`
}
```

#### 4.2 创建用户模型

**文件:**`app/models/user.go`

```go
/**
 * @Author Mr.LiuQH
 * @Description 用户相关的模型
 * @Date 2021/2/4 5:56 下午
 **/
package models

import (
	"goe/app/common"
)

type User struct {
	common.BaseModel
	NickName string          `json:"nickName"`
	Password string          `json:"password"`
	Email    string          `json:"email"`
	Mobile   string          `json:"mobile"`
	Gender   int8            `json:"gender"`
	Birthday common.DateTime `json:"birthday"`
	Status   int8            `json:"status"`
}

// 根据主键查询
func (um *User) FindById(id int) {
	if result := common.GormDBClient.First(um, id); result.Error != nil {
		common.BusErrorInstance.ThrowError(result.Error)
	}
}

// 添加单条记录
func (um *User) Add() {
	if result := common.GormDBClient.Create(um); result.Error != nil {
		common.BusErrorInstance.ThrowError(result.Error)
	}
}

// 根据条件查询
func (um *User) FindByMobile(mobile string) {
	if result := common.GormDBClient.Where("mobile=?", mobile).Find(um); result.Error != nil {
		common.BusErrorInstance.ThrowError(result.Error)
	}
}

/**
 * @description: 更新用户信息
 * @user: Mr.LiuQH
 * @receiver um
 * @param user
 */
func (um *User) UpdateStatus(user User) {
	if result := common.GormDBClient.Model(um).Updates(user); result.Error != nil {
		common.BusErrorInstance.ThrowError(result.Error)
	}
}
/**
 * @description: 删除用户(软删除)
 * @user: Mr.LiuQH
 * @receiver um
 */
func (um *User) DelUser(id int)  {
	if result := common.GormDBClient.Delete(um,id);result.Error != nil {
		common.BusErrorInstance.ThrowError(result.Error)
	}
}
```

### 5.使用

创建`user`控制器，调用模型中的方法，具体代码如下:

**文件:** `app/cntrollers/v1/user.go`

```go
/**
 * @Author Mr.LiuQH
 * @Description TODO
 * @Date 2021/2/22 9:43 上午
 **/
package v1

import (
	"crypto/md5"
	"fmt"
	"goe/app/common"
	"goe/app/models"
	"strconv"
	"time"
)

type UserController struct {
	common.BaseController
}

func init() {
	common.RouteListInstance.AddRoute("v1", "user", &UserController{})
}

/**
 * @description: 查询
 * @user: Mr.LiuQH
 * @receiver u UserController
 * @return error
 */
func (uc UserController) GetUser() error {
	uid := uc.GetParam("uid")
	phone := uc.GetParam("phone")
	userModel := &models.User{}
	if uid != "" {
		id, _ := strconv.Atoi(uid)
		userModel.FindById(id)
	} else if phone != "" {
		userModel.FindByMobile(phone)
	} else {
		return uc.Error("查询条件不能为空!")
	}
	return uc.Success(userModel)
}

/**
 * @description: 注册
 * @user: Mr.LiuQH
 * @receiver uc
 * @return error
 */
func (uc UserController) Register() error {
	nickName := uc.GetParam("nickName")
	email := uc.GetParam("email")
	mobile := uc.GetParam("mobile")
	birthday := uc.GetParam("birthday")
	notEmptyParam := []string{nickName, email, mobile, birthday}
	for _, v := range notEmptyParam {
		if v == "" {
			return uc.Error(fmt.Sprintf("%s不能为空!", v))
		}
	}
	// 判断用户是否存在
	userExist := &models.User{}
	userExist.FindByMobile(mobile)
	if userExist.ID != 0 {
		return uc.Error(fmt.Sprintf("手机号%s已经存在!", mobile))
	}

	location, _ := time.LoadLocation("Asia/Shanghai")
	birthdayTime, _ := time.ParseInLocation("2006-01-02", birthday, location)

	// 插入新用户
	userOne := &models.User{
		NickName: nickName,
		Email:    email,
		Mobile:   mobile,
		Birthday: common.DateTime(birthdayTime),
		Status:   1,
		Password: fmt.Sprintf("%x", md5.Sum([]byte(mobile))),
	}
	userOne.Add()
	// 入库
	return uc.Success(userOne)
}

/**
 * @description: 更新用户信息
 * @user: Mr.LiuQH
 * @receiver uc UserController
 * @return error
 * @date 2021-02-23 10:08:36
 */
func (uc UserController) Update() error {
	uid := uc.GetParam("uid")
	name := uc.GetParam("name")
	phone := uc.GetParam("phone")
	userModel := &models.User{}
	id, _ := strconv.Atoi(uid)
	userModel.FindById(id)
	userUpdate := models.User{
		NickName: name,
		Mobile:   phone,
	}
	userModel.UpdateStatus(userUpdate)
	return uc.Success(userModel)
}
/**
 * @description: 删除用户
 * @user: Mr.LiuQH
 * @receiver uc UserController
 * @return error
 * @date 2021-02-23 15:29:19
 */
func (uc UserController) Del() error  {
	uid := uc.GetParam("uid")
	if uid == "" {
		return uc.Error("缺少参数!")
	}
	userModel := &models.User{}
	id, _ := strconv.Atoi(uid)
	userModel.FindById(id)
	if userModel.ID == 0 {
		return uc.Error("用户不存在!")
	}
	userModel.DelUser(id)
	return uc.Success(userModel)
}
```

### 6.相关地址

[源代码: https://github.com/52lu/goe](https://github.com/52lu/goe)

[博客: http://liuqh.icu/2021/02/02/go/advanced/2-goe-gorm/ ](http://liuqh.icu/2021/02/02/go/advanced/2-goe-gorm/)

[Gorm文档: https://gorm.io/zh_CN/docs/index.html ](https://gorm.io/zh_CN/docs/index.html)

