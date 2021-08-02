## 1. 介绍

`JWT`全称`JSON Web Token`是一种跨域认证解决方案，属于一个开放的标准，它规定了一种`Token`实现方式，目前多用于前后端分离项目和`OAuth2.0`业务场景下。

> [jwt-go](https://github.com/dgrijalva/jwt-go) 是使用`Go`语言实现的`Json web token (JWT)`,目前`GitHub Start 9.8k`,源码地址: https://github.com/dgrijalva/jwt-go，<font color=red>从版本3.2.1开始,源码地址变更为: `github.com/golang-jwt/jwt`,需要下载最新版本时,可以使用这个地址。</font>

### 1.2 集成示意图

![image-20210723174102427](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210723174103.png)

## 2. 配置

### 2.1 编辑主配置

**文件位置：`./config.yaml`**

```yaml
app:
  ...
log:
  ...
mysql:
  ...
jwt:
  secret: liuqh.icu # jwt生成密钥
  issuer: 猿码记 # 签发人
  expire: 3h # 有效时间，值如: 30s|10min|1h
```

### 2.2 新增结构体

**文件位置:  `./config/jwt.go`**

```go
/**
 * @Description JWT配置
 **/
package config

import "time"
// JSON WEB TOKEN 配置
type jwt struct {
	Secret string        `yaml:"secret"`
	Issuer string        `yaml:"issuer"`
	Expire time.Duration `yaml:"expire"`
}
```

## 3. 编辑中间件

**文件位置:`./middleware/jwt.go`**，功能包括有`中间件函数/创建Token/解析Token`

### 3.1 中间件函数

```go
/**
 * @Description JWT中间件
 **/
package middleware

import (
	"52lu/go-import-template/global"
	"52lu/go-import-template/model/dao"
	"52lu/go-import-template/model/request"
	"52lu/go-import-template/model/response"
	"errors"
	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt"
	"go.uber.org/zap"
	"net/http"
	"time"
)
/**
 * @description: JWT中间件
 * @return func(ctx *gin.Context)
 */
func JWTAuthMiddleware() func(ctx *gin.Context) {
	return func(ctx *gin.Context) {
		// 获取参数中的token
		token := getToken(ctx)
		global.GvaLogger.Sugar().Infof("token: %s",token)
		if token == "" {
			response.Error(ctx,"Token不能为空!")
			// 中断请求
			ctx.Abort()
			return
		}
		// 验证Token
		userClaim, err := ParseToken(token)
		if err != nil {
			response.ErrorWithToken(ctx,"Token error :" + err.Error())
			// 中断请求
			ctx.Abort()
			return
		}
		// 设置到上下文中
		setContextData(ctx,userClaim,token)
		// 继续请求后续流程
		ctx.Next()
	}
}
// 设置数据到上下文
func setContextData(ctx *gin.Context,userClaim *request.UserClaims,token string)  {
	userDao := &dao.UserDao{
		Uid: userClaim.Uid,
	}
	user, err := userDao.FindUser()
	if err != nil {
		response.Error(ctx,"用户不存在!")
		// 中断请求
		ctx.Abort()
		return
	}
	user.Token = token
	ctx.Set("userClaim",userClaim)
	ctx.Set("user",user)
}

// 从请求中获取Token
func getToken(ctx *gin.Context) string {
	var token string
	// 从header中获取
	token = ctx.Request.Header.Get("TOKEN")
	if token != "" {
		return token
	}
	// 获取当前请求方法
	if ctx.Request.Method == http.MethodGet {
		// 从Get请求中获取Token
		token, ok := ctx.GetQuery("token")
		if ok {
			return token
		}
	}
	// 从POST中和获取
	if ctx.Request.Method == http.MethodPost {
		postParam := make(map[string]interface{})
		_ = ctx.ShouldBindJSON(&postParam)
		token, ok := postParam["token"]
		if ok {
			return token.(string)
		}
	}
	return ""
}
```

### 3.2 创建Token

```go
// 创建Jwt
func CreateToken(uid uint) (string, error) {
	newWithClaims := jwt.NewWithClaims(jwt.SigningMethodHS256, &request.UserClaims{
		StandardClaims: &jwt.StandardClaims{
			ExpiresAt: time.Now().Add(global.GvaConfig.Jwt.Expire).Unix(), // 有效期
			Issuer:    global.GvaConfig.Jwt.Issuer,                  // 签发人
			IssuedAt:  time.Now().Unix(),                            // 签发时间
		},
		Uid: uid,
	})
	return newWithClaims.SignedString([]byte(global.GvaConfig.Jwt.Secret))
}
```

### 3.3 解析Token

```go
// 验证JWT
func ParseToken(tokenString string) (*request.UserClaims, error) {
	var err error
	var token *jwt.Token
	token, err = jwt.ParseWithClaims(tokenString, &request.UserClaims{}, func(token *jwt.Token) (interface{}, error) {
		return []byte(global.GvaConfig.Jwt.Secret), nil
	})
	if err != nil {
		global.GvaLogger.Error("解析JWT失败", zap.String("error", err.Error()))
		return nil, err
	}
	// 断言
	userClaims, ok := token.Claims.(*request.UserClaims)
	// 验证
	if !ok || !token.Valid {
		return nil, errors.New("JWT验证失败")
	}
	return userClaims, nil
}
```

## 4. 注册路由

![注册路由流程](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210723180519.png)

### 4.1 不需要登录路由

#### 1. 注册路由

**文件位置：`router/user_router.go`**

```go
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
	}
}
```

#### 2. 路由绑定函数

**文件位置:`./api/v1/user_api.go`**

```go
/**
 * @description: TODO 用户账号密码登录
 * @param ctx
 */
func Login(ctx *gin.Context) {
	// 绑定参数
	var loginParam request.LoginParam
	_ = ctx.ShouldBindJSON(&loginParam)
  //...(省略)
	// 生成token
	token, err := middleware.CreateToken(userRecord.ID)
	if err != nil {
		global.GvaLogger.Sugar().Errorf("登录失败,Token生成异常:%s", err)
		response.Error(ctx, "登录失败,账号或者密码错误!")
		return
	}
	userRecord.Token = token
	response.OkWithData(ctx, userRecord)
}
```

#### 3. 请求返回

![](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210723183016.png)

### 4.2 需要登录路由

#### 1. 注册路由

**文件位置：`router/user_router.go`**

```go
/**
 * @description: 用户相关的路由
 * @param engine
 */
func InitUserRouter(engine *gin.Engine) {
	// 不需要登录的路由
	...
	// 需要登录
	tokenGroup := engine.Group("v1/user").Use(middleware.JWTAuthMiddleware())
	{
		tokenGroup.POST("/detail", v1.GetUser)
	}
}
```

#### 2. 路由绑定函数

**文件位置:`./api/v1/user_api.go`**

```go
// 查询用户信息
func GetUser(ctx *gin.Context) {
	// 从上下文中获取用户信息，(经过中间件逻辑时，已经设置到上下文)
	user, _ := ctx.Get("user")
	response.OkWithData(ctx, user)
	return
}
```

#### 3. 请求返回

![](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210723183127.png)

<font color=blue>关注【猿码记】微信公众号，回复【gin集成】获取源码。</font>

