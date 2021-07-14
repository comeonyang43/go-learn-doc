### 1.介绍

[Viper](https://github.com/spf13/viper)是适用于Go应用程序（包括[Twelve-Factor App](https://12factor.net/zh_cn/)）的完整配置解决方案。它被设计用于在应用程序中工作，并且可以处理所有类型的配置需求和格式。目前`Star 15.8k`,它支持以下特性：

- 设置默认值
- 从`JSON`、`TOML`、`YAML`、`HCL`、`envfile`和`Java properties`格式的配置文件读取配置信息
- 实时监控和重新读取配置文件（可选）
- 从环境变量中读取
- 从远程配置系统（`etcd`或`Consul`）读取并监控配置变化
- 从命令行参数读取配置
- 从`buffer`读取配置
- 显式配置值

### 2.安装

```go
go get github.com/spf13/viper
```

### 3. 读取`yaml`

`viper`可以解析`JSON`、`TOML`、`YAML`、`HCL`、`envfile`、`properties`多种格式的配置文件,这里只演示`yaml`格式的解析,其他格式与此类似。

#### 3.1 创建yaml文件

新建`./app.yaml`文件，内容如下:

```yaml
app:
  name: Gin框架学习实践
  version: v1.0
mysql:
  host: 127.0.0.1
  port: 3306
  user: root
  password: 123456
```

#### 3.2  解析`yaml`文件

```go
package main
import (
	"fmt"
	"github.com/spf13/viper"
)
type APP struct {
	Name    string `yaml:"name"`
	Version string `yaml:"version"`
}

type MYSQL struct {
	Host     string `yaml:"host"`
	Port     string `yaml:"port"`
	User     string `yaml:"user"`
	Password string `yaml:"password"`
}

type YamlConfig struct {
	App   APP   `yaml:"app"`
	Mysql MYSQL `yaml:"mysql"`
}
// 解析yaml配置文件
func main() {
	viperConfig := viper.New()
	// 设置配置文件名，没有后缀
	viperConfig.SetConfigName("app")
	// 设置读取文件格式为: yaml
	viperConfig.SetConfigType("yaml")
	// 设置配置文件目录(可以设置多个,优先级根据添加顺序来)
	viperConfig.AddConfigPath(".")
	// 读取解析
	if err := viperConfig.ReadInConfig(); err != nil {
		if _, ok := err.(viper.ConfigFileNotFoundError); ok {
			fmt.Printf("配置文件未找到！%v\n", err)
			return
		} else {
			fmt.Printf("找到配置文件,但是解析错误,%v\n", err)
			return
		}
	}
	// 映射到结构体
	var yamlConfig YamlConfig
	if err := viperConfig.Unmarshal(&yamlConfig);err != nil{
		fmt.Printf("配置映射错误,%v\n", err)
	}
	fmt.Printf("App: %+v\n",yamlConfig.App)
	fmt.Printf("Mysql: %+v\n",yamlConfig.Mysql)
}
```

**运行输出:**

```bash
App: {Name:Gin框架学习实践 Version:}
Mysql: {Host:127.0.0.1 Port:3306 User:root Password:123456}
```

#### 3.3 读取优先级

如果存在多个配置文件在不同的目录下,可以通过`viper.AddConfigPath(path)`来设置优先级，先添加的优先级越高。

**修改3.2部分代码**

```go
...
	viperConfig := viper.New()
	// 设置配置文件名，没有后缀
	viperConfig.SetConfigName("app")
	// 设置读取文件格式为: yaml
	viperConfig.SetConfigType("yaml")
  // 设置配置文件目录(可以设置多个,优先级根据添加顺序来)
  viperConfig.AddConfigPath(".")
  viperConfig.AddConfigPath("./config")
...
```

<font color=red>先查找(.)当前目录下是否有app.yaml，如果没有则去(./config/)中查找</font>。

#### 3.4 设置默认值

```go
import (
	"github.com/spf13/viper"
)
// 解析yaml配置文件
func main()  {
	viperConfig := viper.New()
	// 指定配置文件
	viperConfig.SetConfigFile("./app.yaml")
	// 设置默认值
	viperConfig.SetDefault("app.version","v1.2.0")
	viperConfig.SetDefault("app.BuildDate","2020-10-10 12:12:12")
	// 读取解析
	if err := viperConfig.ReadInConfig(); err != nil {
		if _, ok := err.(viper.ConfigFileNotFoundError); ok {
			panic("配置文件未找到！")
		} else {
			panic("找到配置文件,但是解析错误！")
		}
	}
  // 映射到结构体
	var yamlConfig YamlConfig
	if err := viperConfig.Unmarshal(&yamlConfig); err != nil {
		panic("配置映射错误！" + err.Error())
	}
	return &yamlConfig
}
// 新增BuildDate
type APP struct {
	Name      string    `yaml:"name"`
	Version   string    `yaml:"version"`
	BuildDate string     `yaml:"buildDate"`
}
// 同3.1
type MYSQL struct {
  ...
}
// 同3.1
type YamlConfig struct {
	  ...
}
//输出：&{App:{Name:Gin框架学习实践 Version:v1.0 BuildDate:2020-10-10 12:12:12} Mysql:{Host:127.0.0.1 Port:3306 User:root Password:123456}}
```

> `yaml`文件同 `3.1 创建的yaml文件`

### 4.从`io.Reader`读取配置

#### 4.1 代码

```go
func ReadFromByte() {
	// 定义配置变量
	yamVar := []byte(`
desc: viper使用
app:
  name: goItem
  version: v1.3
mysql:
   host: 127.0.0.1
   port: 3306
   user: root
   password: 123456
`)
	v := viper.New()
	// 设置解析格式
	v.SetConfigType("yaml")
	err := v.ReadConfig(bytes.NewBuffer(yamVar))
	if err != nil {
		fmt.Println(err)
		return
	}
	// 输出变量
	fmt.Printf("desc: %v \n",v.Get("desc"))
	fmt.Printf("app.name: %v \n",v.Get("app.name"))
	fmt.Printf("app.version: %v \n",v.Get("app.version"))
	fmt.Printf("mysql: %v \n",v.Get("mysql"))
}
```

#### 4.2 输出

```bash
desc: viper使用 
app.name: goItem 
app.version: v1.3 
mysql: map[host:127.0.0.1 password:123456 port:3306 user:root] 
```

### 5.覆盖和别名

#### 5.1 创建配置文件`test.yaml`

```yaml
name: Gin-use
author: 刘庆辉
```

#### 5.2 代码示例

```go
type TestConfig struct {
	Name   string `yaml:"name"`
	Author string `yaml:"author"`
}

// 测试配置覆盖
func main() {
	v := viper.New()
	v.SetConfigFile("./test.yaml")
	if err2 := v.ReadInConfig(); err2 != nil {
		if _, ok := err2.(viper.ConfigFileNotFoundError); ok {
			panic("配置文件未找到！")
		} else {
			panic("找到配置文件,但是解析错误！")
		}
	}
	// 设置别名
	v.RegisterAlias("author","UserName")
	var testConfig TestConfig
	// 覆盖变量
	v.Set("name", "测试配置覆盖")
	err := v.Unmarshal(&testConfig)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("testConfig:%+v\n", testConfig)
	fmt.Printf("别名UserName:%+v\n", v.GetString("UserName"))
}
/**输出
testConfig:{Name:测试配置覆盖 Author:刘庆辉}
别名UserName:刘庆辉
*/
```

### 6.动态修改

#### 6.1 代码

```go
package main
import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"github.com/gin-gonic/gin"
	"github.com/spf13/viper"
)

type APP struct {
	Name      string `yaml:"name"`
	Version   string `yaml:"version"`
	BuildDate string `yaml:"buildDate"`
}
type MYSQL struct {
	Host     string `yaml:"host"`
	Port     string `yaml:"port"`
	User     string `yaml:"user"`
	Password string `yaml:"password"`
}
type YamlConfig struct {
	App   APP   `yaml:"app"`
	Mysql MYSQL `yaml:"mysql"`
}
var yamlConfig YamlConfig
func main() {
	engine := gin.Default()
	// 读取配置
	WatchConfigYaml()
	engine.GET("/test", func(context *gin.Context) {
		context.JSON(200,gin.H{"config":yamlConfig})
	})
	_ = engine.Run()
}

// 动态监听yaml配置文件
func WatchConfigYaml() {
	v := viper.New()
	v.SetConfigFile("./app.yaml")
	if err := v.ReadInConfig();err != nil {
		if _,ok := err.(viper.ConfigFileNotFoundError);ok{
			fmt.Printf("配置文件不存在:%v\n",err)
		} else {
			fmt.Printf("配置文件存在,解析失败:%v\n",err)
		}
	}
	v.WatchConfig()
	v.OnConfigChange(func(e fsnotify.Event) {
		// 配置动态改变时，回调函数
		fmt.Printf("配置发生改变,重新解析配置: %v \n",e.Name)
		if err := v.Unmarshal(&yamlConfig);err != nil {
			fmt.Println(err)
		}
	})
	if err := v.Unmarshal(&yamlConfig);err != nil {
		fmt.Println(err)
	}
}
```

#### 6.2 请求

启动服务后，访问:http://127.0.0.1:8080/test, 之后修改配置文件，保存后刷新浏览器器，会发现配置已经修改。无需重启,即可生效！



[更多使用方法，可查看文档:https://github.com/spf13/viper/](https://github.com/spf13/viper/)

