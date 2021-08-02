## 1. 介绍

`JWT`全称`JSON Web Token`是一种跨域认证解决方案，属于一个开放的标准，它规定了一种`Token`实现方式，目前多用于前后端分离项目和`OAuth2.0`业务场景下。

> [jwt-go](https://github.com/dgrijalva/jwt-go) 是使用`Go`语言实现的`Json web token (JWT)`,目前`GitHub Start 9.8k`,源码地址: https://github.com/dgrijalva/jwt-go，<font color=red>从版本3.2.1开始,源码地址变更为: `github.com/golang-jwt/jwt`,需要下载最新版本时,可以使用这个地址。</font>

## 2. 安装

```go
// 下载最新版本
go get -u github.com/golang-jwt/jwt
```

## 3.生成JWT

```go
// 设置声明
type UserClaims struct {
	*jwt.StandardClaims
	Name string
	Uid  uint
}
// 创建Jwt
func TestCreateJWTByNewWithClaims(t *testing.T) {
	nowTime := time.Now()
	// 使用NewWithClaims创建声明
	claims := jwt.NewWithClaims(jwt.SigningMethodHS256,&UserClaims{
		StandardClaims: &jwt.StandardClaims{
			// 设置有效期为5分钟
			ExpiresAt: nowTime.Add(time.Minute * 5).Unix(),
			Issuer: "猿码记", // 签发人
			IssuedAt: nowTime.Unix(), //签发时间
		},
		Name: "张三",
		Uid: 100,
	})
	// 使用指定的secret签名,获取字符串token
	signedString, err := claims.SignedString([]byte("这是我定义的secret"))
	if err != nil {
		t.Error(err)
		return
	}
	fmt.Println("生成JWT:",signedString)
}

/** 输出
=== RUN   TestCreateJWTByNewWithClaims
生成JWT: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjY4NTM5OTcsImlhdCI6MTYyNjg1MzY5NywiaXNzIjoi54y_56CB6K6wIiwiTmFtZSI6IuW8oOS4iSIsIlVpZCI6MTAwfQ.GFu9ixPIMFauXUgu3GC35UD7ZuXLDN4l9M_8-k3yseE
--- PASS: TestCreateJWTByNewWithClaims (0.00s)
PASS
*/
```

## 4. 解析JWT

```go
// 解析JWT
func TestValidJWT(t *testing.T) {
	jwtStr := "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjY5MDk0MTYsImlhdCI6MTYyNjg1NTQxNiwiaXNzIjoi54y_56CB6K6wIiwiTmFtZSI6IuW8oOS4iSIsIlVpZCI6MTAwfQ.NG0qvleaLBVLrpOr89Z83c4EaYaVFbCnqY4POdIMuks"
	// 解析jwt,第三个参数是个函数，返回生成jwt设置的secret
	token, err := jwt.ParseWithClaims(jwtStr, &UserClaims{}, func(token *jwt.Token) (interface{}, error) {
		return []byte("这是我定义的secret"), nil
	})
	if err != nil {
		t.Error("解析失败:", err.Error())
		return
	}
	// 断言类型
	claim, ok := token.Claims.(*UserClaims)
	// 验证
	if !ok || !token.Valid {
		t.Error("解析失败,Token不合法!")
		return
	}
	fmt.Printf("解析结果: %+v\n", claim)
}

/** 输出
=== RUN   TestValidJWT
解析结果: &{StandardClaims:0xc0000a4360 Name:张三 Uid:100}
--- PASS: TestValidJWT (0.00s)
PASS
*/
```



[更多使用可见文档: https://pkg.go.dev/github.com/dgrijalva/jwt-go](https://pkg.go.dev/github.com/dgrijalva/jwt-go)

