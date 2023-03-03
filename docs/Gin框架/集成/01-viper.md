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



### 2.项目介绍

`Gin`框架学习使用,并实践常用包在`Gin`框架中的集成和使用。[源码地址: https://github.com/52lu/gin-import-template.git]( https://github.com/52lu/gin-import-template.git)

#### 2.1 项目结构

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

#### 2.2 集成流程

![集成流程](https://go.liuqh.icu/img/20210706163517.png)



### 3.安装

```go
go get github.com/spf13/viper
```

### 4.配置

#### 4.1 创建全局配置文件

新建`./config/app.yaml`

```yaml
app:
  desc: Gin框架学习实践
  version: v1.0.0
  addr: 0.0.0.0:8080
  env: dev
mysql:
  host: 127.0.0.1
  port: 3306
  user: root
  password: 123456
redis:
  host: 127.0.0.1
  port: 6379
```

#### 4.2 创建全局配置结构体

新建结构体`./config/app.go`

```go
package config
// 应用信息
type app struct {
	Desc       string `yaml:"desc"`
	Addr       string `yaml:"addr"`
	ConfigFile string `yaml:"configFile"`
	Version    string `yaml:"version"`
	Env        string `yaml:"env"`
}
// MySQL信息
type mysql struct {
	Host     string `yaml:"host"`
	Port     string `yaml:"port"`
	User     string `yaml:"user"`
	Password string `yaml:"password"`
}
// Redis
type redis struct {
	Host string `yaml:"host"`
	Port string `yaml:"port"`
}
// ServerConfig 配置信息
type ServerConfig struct {
	App   app   `yaml:"app"`
	Mysql mysql `yaml:"mysql"`
	Redis redis `yaml:"redis"`
	Log   log   `yaml:"log"`
}
```

#### 4.3 定义全局变量和常量

新建`./global/global.go`

```go
// 常量
const (
	ConfigFile = "./config/app.yaml" // 配置文件
)
// 变量
var (
	GvaConfig config.ServerConfig // 全局配置
)
```

### 5.具体集成

#### 5.1 集成入口(`InitConfig`)

新建`./initialize/viper.go`

```go
// ViperInit 初始化viper配置解析包，函数可接受命令行参数
func InitConfig() {
	var configFile string
	// 读取配置文件优先级: 命令行 > 默认值
	flag.StringVar(&configFile,"c",global.ConfigFile,"配置配置")
	if len(configFile) == 0 {
		// 读取默认配置文件
		panic("配置文件不存在！")
	}
	// 读取配置文件
	v := viper.New()
    v.SetConfigFile(configFile)
	if err := v.ReadInConfig();err != nil {
		panic(fmt.Errorf("配置解析失败:%s\n",err))
	}
	// 动态监测配置文件
	v.WatchConfig()
	v.OnConfigChange(func(in fsnotify.Event) {
		fmt.Println("配置文件发生改变")
		if err := v.Unmarshal(&global.GvaConfig); err != nil {
			panic(fmt.Errorf("配置重载失败:%s\n",err))
		}
	})
	if err := v.Unmarshal(&global.GvaConfig); err != nil {
		panic(fmt.Errorf("配置重载失败:%s\n",err))
	}
	// 设置配置文件
	global.GvaConfig.App.ConfigFile = configFile
}
```

#### 5.2 验证测试

```bash
➜ curl http://127.0.0.1:8080/config
{
    "code": 0,
    "msg": "请求成功",
    "data": {
        "App": {
            "Desc": "Gin框架学习实践",
            "Addr": "0.0.0.0:8080",
            "ConfigFile": "./config/app.yaml",
            "Version": "v1.0.0",
            "Env": "dev"
        },
        "Mysql": {
            "Host": "127.0.0.1",
            "Port": "3306",
            "User": "root",
            "Password": "123456"
        },
        "Redis": {
            "Host": "127.0.0.1",
            "Port": "6379"
        },
        "Log": {
            "Path": "./logs",
            "Level": "debug",
            "FilePrefix": "gin",
            "FileFormat": "2006-01-02",
            "OutFormat": "json",
            "LumberJack": {
                "MaxSize": 1,
                "MaxBackups": 3,
                "MaxAge": 30,
                "Compress": false
            }
        }
    },
    "time": "2021-06-06 17:16:48"
}
```



