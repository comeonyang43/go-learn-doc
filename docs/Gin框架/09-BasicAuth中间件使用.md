### 1.什么是`BasicAuth`?

`Basic Auth`是一种开放平台认证方式，简单的说就是需要你输入用户名和密码才能继续访问。

### 2.在单路由中使用

​	如果需要针对单个路由使用，在要在单路由中注册`BasicAuth`中间件即可。

#### 2.1 **代码**

```go
// 使用BasicAuth中间件
func main(){
	engine := gin.Default()
  // 设置账号和密码，key:代表账号,value:代表密码
	ginAccounts := gin.Accounts{
		"user":"password",
		"abc":"123",
	}
  // 注册路由和中间件
	engine.GET("/test",gin.BasicAuth(ginAccounts), func(context *gin.Context) {
		// 获取中间件BasicAuth
		user := context.MustGet(gin.AuthUserKey).(string)
		fmt.Println(user)
		context.JSON(200,gin.H{"msg":"success"})
	}) 
	_ = engine.Run()
}
```

#### 2.2 访问效果

![image-20210524113630422](/img/20210524113630.png)

### 3.在路由组中使用

绝大部分情况下,我们都是在路由组中使用`BasicAuth`中间件。

#### 3.1 代码

```go
func RunUseBasicAuthWithGroup() {
	engine := gin.Default()
	// 注册路由组和中间件
	userGroup := engine.Group("/user", gin.BasicAuth(gin.Accounts{
		"abc": "123",
	}))
	userGroup.GET("info", func(context *gin.Context) {
		context.JSON(200, gin.H{"msg": "user.info"})
	})
}
```

 **访问效果同上。**

