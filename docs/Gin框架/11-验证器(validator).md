### 1.介绍

[validator](https://github.com/go-playground/validator)是一个开源的验证器包，可以快速校验输入信息是否符合自定规则。目前`Star 7.8k`,源码地址: https://github.com/go-playground/validator

#### 1.1 安装

```go
go get github.com/go-playground/validator
```

#### 1.2 引用

```go
import "github.com/go-playground/validator"
```

#### 1.3 示例

**代码:**

```go
package main
import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/go-playground/validator/v10"
)
// 定义一个添加用户参数结构体
type AddUserPost struct {
	Name  string `json:"name" validate:"required"`        //必填
	Email string `json:"email" validate:"required,email"` // 必填，并且格式是email
	Age   uint8    `json:"age" validate:"gte=18,lte=30"` // 年龄范围
}
// 简单示例
func main() {
	engine := gin.Default()
	engine.POST("/valid", func(context *gin.Context) {
		var adduserPost  AddUserPost
		// 接收参数
		err := context.ShouldBindJSON(&adduserPost)
		if err != nil {
			context.JSON(500,gin.H{"msg":err})
			return
		}
		fmt.Printf("adduserPost: %+v\n",adduserPost)
		// 使用Validate验证
		validate := validator.New()
		err = validate.Struct(adduserPost)
		if err != nil {
			fmt.Println(err)
			context.JSON(500,gin.H{"msg":err.Error()})
			return
		}
		context.JSON(200,gin.H{"msg":"success"})
	})
	_ = engine.Run()
}
```

**请求:**

```bash
# email不合法时
➜ curl -X POST http://127.0.0.1:8080/valid -d '{"name":"张三","email":"123","age":21}'
{"msg":"Key: 'AddUserPost.Email' Error:Field validation for 'Email' failed on the 'email' tag"}
# age 不在指定范围时
➜ curl -X POST http://127.0.0.1:8080/valid -d '{"name":"张三","email":"123@163.com","age":17}'
{"msg":"Key: 'AddUserPost.Age' Error:Field validation for 'Age' failed on the 'gte' tag"}
# 姓名不填时
➜ curl -X POST http://127.0.0.1:8080/valid -d '{"name":"","email":"123@163.com","age":20}'
{"msg":"Key: 'AddUserPost.Name' Error:Field validation for 'Name' failed on the 'required' tag"}
```

### 2.改成中文

#### 2.1 代码

```go
package main
import (
	"github.com/gin-gonic/gin"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"
	zhs "github.com/go-playground/validator/v10/translations/zh"
)
var (
	validate = validator.New()          // 实例化验证器
	chinese  = zh.New()                 // 获取中文翻译器
	uni      = ut.New(chinese, chinese) // 设置成中文翻译器
	trans, _ = uni.GetTranslator("zh")  // 获取翻译字典
)
type User struct {
	Name  string `form:"name" validate:"required,min=3,max=5"`
	Email string `form:"email" validate:"email"`
	Age   int8   `form:"age" validate:"gte=18,lte=20"`
}

func main() {
	engine := gin.Default()
	engine.GET("/language", func(context *gin.Context) {
		var user User
		err := context.ShouldBindQuery(&user)
		if err != nil {
			context.JSON(500, gin.H{"msg": err})
			return
		}
		// 注册翻译器
		_ = zhs.RegisterDefaultTranslations(validate, trans)
		// 使用验证器验证
		err = validate.Struct(user)
		if err != nil {
			if errors, ok := err.(validator.ValidationErrors); ok {
				// 翻译，并返回
				context.JSON(500, gin.H{
					"翻译前": errors.Error(),
					"翻译后": errors.Translate(trans),
				})
				return
			}
		}
		context.JSON(200,gin.H{"msg":"success"})
	})
	_ = engine.Run()
}
```

#### 2.2 请求

```bash
# 不传参数
➜ curl -X GET http://127.0.0.1:8080/language
{
    "翻译前":"Key: 'User.Name' Error:Field validation for 'Name' failed on the 'required' tag
Key: 'User.Email' Error:Field validation for 'Email' failed on the 'email' tag
Key: 'User.Age' Error:Field validation for 'Age' failed on the 'gte' tag",
    "翻译后":{
        "User.Age":"Age必须大于或等于18",
        "User.Email":"Email必须是一个有效的邮箱",
        "User.Name":"Name为必填字段"
    }
}
```



### 3.校验规则

整理一些常用的规则

| Tag标签    | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| `required` | 必填                                                         |
| `unique`   | 保证唯一性,不同类型处理不同;<br/>对于`map，unique`约束没有重复的`value`值；<br/>对于数组和切片，没有重复的元素值<br/> |
| `min`      | 最小值,不同类型含义不一样<br>数字类型代表最小值。<br/>字符串型代表最小字符数 |
| `max`      | 最大值,不同类型含义不一样<br/>数字类型代表最大值。<br/>字符串型代表最大字符数 |
| `gte`      | 大于等于参数值                                               |
| `gt`       | 大于参数值                                                   |
| `lt`       | 小于参数值                                                   |
| `lte`      | 小于d等于参数值                                              |
| `email`    | 使用`email`来限制字段必须是邮件形式                          |
| `oneof`    | 必须是指定值的其中一个，如:`oneof=男 女`,只能填男或女        |

[更多规则，则可查看文档:https://github.com/go-playground/validator](https://github.com/go-playground/validator)

### 4.自定义规则

#### 4.1 代码

```go
package main
import (
	"fmt"
	"github.com/go-playground/validator/v10"
)
// 验证pre
type CustomParam struct {
	Pre string `validate:"pre=go_"`
}
func main()  {
	// 实例化验证器
	validate = validator.New()
	// 注册自定义标签
	_ = validate.RegisterValidation("pre", ValidatePre)
	cusParam := CustomParam{
		Pre: "php_",
	}
	err := validate.Struct(cusParam)
	fmt.Println(err)
}
// 自定义验证规则
func ValidatePre(fl validator.FieldLevel) bool {
	return fl.Field().String() == "go_"
}
```

#### 4.2 请求返回

```bash
Key: 'CustomParam.Pre' Error:Field validation for 'Pre' failed on the 'pre' tag
```

### 5.在Gin中使用

#### 5.1 代码

```go
package main
import (
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/validator/v10"
	"net/http"
	"time"
)
// 定义结构体
type User struct {
	Name string `form:"name" binding:"required,min=3,max=5" `
	BirthDate time.Time `form:"date" binding:"required,birth" time_format:"2006-01-02"`
}

// 运行程序
func main()  {
	engine := gin.Default()
  // 注册自定义验证标签:birth
	if validate,ok  := binding.Validator.Engine().(*validator.Validate);ok {
		validate.RegisterValidation("birth",checkBirthDate)
	}
  // 接收请求
	engine.GET("/valid", func(context *gin.Context) {
		var user User
    // 集成验证
		err := context.ShouldBindQuery(&user)
		if err != nil {
			context.JSON(http.StatusBadRequest,gin.H{"error":err.Error()})
			return
		}
		context.JSON(http.StatusOK,gin.H{"msg":"success"})
	})
	_ = engine.Run()
}
// 检测生日
func checkBirthDate(fl validator.FieldLevel) bool {
	t,ok := fl.Field().Interface().(time.Time)
	if ok {
		// 当前时间应该大于生日时间
		if time.Now().After(t) {
			return true
		}
	}
	return false
}
```

#### 5.2 请求返回

```bash
# name错误时
➜  curl -X GET http://127.0.0.1:8080/valid?name=张三&date=2020-01-01
{"error":"Key: 'User.Name' Error:Field validation for 'Name' failed on the 'min' tag"}
# 自定义birth格式错误时
➜  curl -X GET http://127.0.0.1:8080/valid?name=张是三&date=2020
{"error":"parsing time \"2020\" as \"2006-01-02\": cannot parse \"\" as \"-\""}
# 都正确
➜  curl -X GET http://127.0.0.1:8080/valid?name=张是三&date=2020-10-10
{"msg":"success"}
```


