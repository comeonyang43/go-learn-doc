## 1. 介绍

`cobra`是一个用来构建现代CLI工具的库。相比`flag`标准库，它提供更多方便的特性和功能。`Cobra` 由 `Go` 项目成员和 `hugo` 作者 [spf13](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspf13) 创建，已经被许多流行的 `Go` 项目采用，比如 [GitHub CLI](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fcli%2Fcli) 和 [Docker CLI](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdocker%2Fcli)。

> 源码地址: https://github.com/spf13/cobra，截止到现在`Star 23.8K`

### 1.1 特性预览

- 使用`cobra add cmdname`可快速的创建子命令`cli`
- 全局、局部和级联的标志
- 自动生成`commands`和`flags`的帮助信息
- 自动识别 `-h`、`--help` 等帮助标识
- 支持自定义帮助信息，用法等的灵活性。
- 可与[viper](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspf13%2Fviper) 紧密集成

### 1.2 相关概念

`Cobra` 结构由三部分组成：命令 (`commands`)、参数 (`arguments`)、标志 (`flags`)。最好的应用程序在使用时读起来像句子，要遵循的模式：

```bash
# 没有子命令
`app cmd --param=？`: 
# 有子命令
`app cmd subCmd --param=？`
```

> `app`:代表编译后的文件名, `cmd`:代表命令  `subCmd`:代表子命令 `--param`: 代表请求参数。

## 2. 安装

```bash
go get -u github.com/spf13/cobra/cobra 
```

##  3. 快速使用

> 快速创建一个`cli`,效果是`app server --port=?`运行一个服务

### 3.1 创建根命令(`rootCmd`)

**文件位置:**`cmd/root.go`

```go
package cmd

import (
	"github.com/spf13/cobra"
)

func init() {
	// 接受参数
	rootCmd.PersistentFlags().String("version", "", "版本")
}

// rootCmd represents the base command when called without any subcommands
// 根命令
var rootCmd = &cobra.Command{
	Use:   "app",
	Short: "命令行的简要描述....",
	Long: `学习使用Cobra,开发cli项目,
- app: 指的是编译后的文件名。`,
   //// 根命令执行方法，如果有要
	//Run: func(cmd *cobra.Command, args []string) {
	//	fmt.Println("args:",args)
	//},
}

// Execute 将所有子命令添加到root命令并适当设置标志。
// 这由 main.main() 调用。它只需要对 rootCmd 调用一次。
func Execute() {
	cobra.CheckErr(rootCmd.Execute())
}
```

### 3.2 创建子命令

#### 1. 创建文件:`cobra add ?`

```bash
# 创建子命令server
➜ cobra add server   
server created at /Users/liuqh/ProjectItem/GoItem/go-cli
```

#### 2. 查看创建子命令内容

**文件位置:**`cmd/server.go`

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
)
// serverCmd represents the server command
var serverCmd = &cobra.Command{
	Use:   "server",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("server called")
	},
}
func init() {
	rootCmd.AddCommand(serverCmd)
}
```

#### 3. 编辑子命令内容

**文件位置:**`cmd/server.go`

```go
package cmd

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/spf13/cobra"
	"os"
)
// serverCmd represents the server command
var (
	serverCmd = &cobra.Command{
		Use:   "server",
		Short: "启动http服务,使用方法: app server --port=?",
		Run: func(cmd *cobra.Command, args []string) {
			if port == "" {
				fmt.Println("port不能为空!")
				os.Exit(-1)
			}
			engine := gin.Default()
			_ = engine.Run(":" + port)
		},
	}
	//接收端口号
	port string
)
func init() {
	// 添加命令
	rootCmd.AddCommand(serverCmd)
	// 接收参数port
	serverCmd.Flags().StringVar(&port, "port", "", "端口号")
}
```

### 3.3 编译运行

#### 1. 编译

```bash
# 编译(编译后的文件名为: app)
➜ go build -o app .
```

#### 2. 运行(不带参数)

```bash
➜ ./app          
学习使用Cobra,开发cli项目,
- app: 指的是编译后的文件名。

Usage:
  app [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  help        Help about any command
  server      启动http服务,使用方法: app server --port=?

Flags:
  -h, --help             help for app
      --version string   版本

Use "app [command] --help" for more information about a command.
```

#### 3. 查看具体子命令

```bash
# 查看server使用信息
➜ ./app server -h
启动http服务,使用方法: app server --port=?

Usage:
  app server [flags]

Flags:
  -h, --help          help for server
      --port string   端口号

Global Flags:
      --version string   版本
```

#### 4. 运行子命令

```bash
# 不传必填参数时
➜ ./app server       
port不能为空!
# 传参数
➜ ./app server --port 8090
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] Listening and serving HTTP on :8090
```



## 4. 嵌套子命令

#### 4.1 编辑命令

**文件位置:**`cmd/user.go`

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
)

// userCmd 父命令
var userCmd = &cobra.Command{
	Use:   "user",
	Short: "用户操作",
}

// 子命令(添加用户)
var userAddCmd = &cobra.Command{
	Use:   "add",
	Short: "添加用户；user add --name=?",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("添加用户:",name)
	},
}

// 子命令(删除用户)
var userDelCmd = &cobra.Command{
	Use:   "del",
	Short: "删除用户；user del --name=?",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("删除用户:",name)
	},
}

var name string
func init() {
	// 添加子命令到父命令
	userCmd.AddCommand(userDelCmd)
	userCmd.AddCommand(userAddCmd)
  // 添加到根命令
	rootCmd.AddCommand(userCmd)
  // 接收参数
	userCmd.PersistentFlags().StringVarP(&name,"name","n","","用户名")
}
```

#### 4.2 查看命令

```bash
➜ ./app user -h    
用户操作

Usage:
  app user [command]

Available Commands:
  add         添加用户；user add --name=?
  del         删除用户；user del --name=?

Flags:
  -h, --help          help for user
  -n, --name string   用户名
```

#### 4.3 编译运行

```bash
# 编译
➜ go build -o app .

# 运行
➜ ./app user add -n 张三
添加用户: 张三
➜ ./app user del -n 张三
删除用户: 张三
```

## 5. 标志 (`flags`)

`cobra`的标志指的就是参数的名称，但是有**本地标志**和**持久化标志**之分,白话描述:

- 本地标志(`Flags`): 当前命令接收,当前命令使用。
- 持久标志(`PersistentFlags`): 当前命令接收，当前命令行和其所有子命令都可使用。

### 5.1 使用示例

> 修改用户操作命令 `cmd/user.go`

#### 1. 修改后的脚本

**文件位置:**`cmd/user.go`

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
)

// userCmd 父命令
var userCmd = &cobra.Command{
	Use:   "user",
	Short: "用户操作",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("用户列表: ",list)
	},
}

// 子命令(添加用户)
var userAddCmd = &cobra.Command{
	Use:   "add",
	Short: "添加用户；user add --name=?",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("添加用户:",name)
	},
}

// 子命令(删除用户)
var userDelCmd = &cobra.Command{
	Use:   "del",
	Short: "删除用户；user del --name=?",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("删除用户:",name)
	},
}

var (
	name string
	list []string
)
func init() {
	// 添加子命令到父命令
	userCmd.AddCommand(userDelCmd)
	userCmd.AddCommand(userAddCmd)
	rootCmd.AddCommand(userCmd)
	// 父命令接收持久标志
	userCmd.PersistentFlags().StringVarP(&name,"name","n","","用户名")
	// 父命令接收本地标志
	userCmd.Flags().StringSliceVarP(&list,"list","l",[]string{},"用户列表")
}
```

#### 2. 运行测试

```bash
# 传入本地标志
➜  ./app user --list="小明,小三"    
用户列表:  [小明 小三]

➜ ./app user del --list="小明,小三"
Error: unknown flag: --list
Usage:
  app user del [flags]

Flags:
  -h, --help   help for del

Global Flags:
  -n, --name string   用户名

Error: unknown flag: --list
```





## 6.  参数限制

### 6.1 位置参数限制

- `NoArgs` : 如果有任何位置参数，该命令将报告错误。

- `MinimumNArgs(int)` :至少传 N 个位置参数，否则报错。
- `ArbitraryArgs`: 接受任意个位置参数。

- `MaximumNArgs(int)` : 最多传N 个位置参数，否则报错。

- `ExactArgs(int)` : 传入位置参数个数等于N，否则报错。

- `RangeArgs(min, max)` : 传入位置参数个数 `min<= N <= max`，否则报错

#### 1. 代码示例

```go
var (
	// 子命令(添加用户)
	userAddCmd = &cobra.Command{
		Use:   "add",
		Short: "添加用户；user add --name=?",
		Args: cobra.RangeArgs(1,3),
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("位置参数(args):", args)
		},
	}
)
```

> **<font color=red>@注意：上面规则限制的是位置参数，并不是标志，不要混淆。</font>**

#### 2.运行测试

```bash
# 传入3个位置参数
➜  go-cli ./app user add 1 2 3      
位置参数(args): [1 2 3]
# 传入4个位置参数
➜  go-cli ./app user add 1 2 3 4    
Error: accepts between 1 and 3 arg(s), received 4
Usage:
  app user add [flags]

Flags:
  -h, --help   help for add

Error: accepts between 1 and 3 arg(s), received 4
```

### 6.2 标志参数限制

标志默认是可选的。如果你想在缺少标志时命令报错，可使用`MarkFlagRequired`限制

#### 1. 代码示例

```go
func init() {
	// 添加子命令到父命令
	userCmd.AddCommand(userAddCmd)
	rootCmd.AddCommand(userCmd)
	// 标志
	userAddCmd.Flags().StringVar(&name,"name","","用户名")
	// 标志必需
	err := userAddCmd.MarkFlagRequired("name")
	if err != nil {
		fmt.Println("--name 不能为空")
		return
	}
}
```

#### 2.运行测试

```bash
# 不传--name标志
➜ ./app user add 
Error: required flag(s) "name" not set
Usage:
  app user add [flags]

Flags:
  -h, --help          help for add
      --name string   用户名

Error: required flag(s) "name" not set
# 传 --name 标志
➜ ./app user add --name=张三
name: 张三
```

### 6.3 自定义位置限制

#### 1.代码示例

```go
package cmd

import (
	"errors"
	"fmt"
	"github.com/spf13/cobra"
	"unicode/utf8"
)

// userCmd 父命令
var userCmd = &cobra.Command{
	Use:   "user",
	Short: "用户操作",
}

var (
	// 子命令(添加用户)
	userAddCmd = &cobra.Command{
		Use:   "add",
		Short: "添加用户；user add name",
		// 自定义参数限制
		Args: func(cmd *cobra.Command, args []string) error {
			if len(args) != 1 {
				return errors.New("参数数量不对")
			}
			// 判断姓名长度
			count := utf8.RuneCountInString(args[0])
			fmt.Printf("%v %v \n",args[0],count)
			if count > 4 {
				return errors.New("姓名长度过长")
			}
			return nil
		},
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("args:", args)
		},
	}
)
func init() {
	// 添加子命令到父命令
	userCmd.AddCommand(userAddCmd)
	rootCmd.AddCommand(userCmd)
}
```

#### 2. 运行测试

```bash
# 姓名正常时
➜ ./app user add 上官伊人      
上官伊人 4 
args: [上官伊人]
# 姓名超长时
➜ ./app user add 上官伊人家
上官伊人家 5 
Error: 姓名长度过长
Usage:
  app user add [flags]

Flags:
  -h, --help   help for add

Error: 姓名长度过长
# 参数数量不对时
➜ ./app user add 上官伊人家 小三
Error: 参数数量不对
Usage:
  app user add [flags]

Flags:
  -h, --help   help for add

Error: 参数数量不对
```



## 7.集成viper

### 7.1 查看目录结构

```bash
├── app
│   └── config
│       ├── app.go # 配置结构体
│       └── app.yaml # 配置文件
├── cmd
│   ├── root.go # 根命令
│   └── server.go # http服务
├── go.mod
├── go.sum
├── local.yaml #
└── main.go
```

### 7.2 代码实现

#### 1. 解析配置: `cmd/root.go`

```go
package cmd

import (
	"52lu/go-cli/app/config"
	"fmt"
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
	"os"
)

func init() {
	// 初始化配置信息
	cobra.OnInitialize(initConfig)
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is ./app.yaml | ./config/app.yaml )")
}

// rootCmd represents the base command when called without any subcommands
// 根命令
var rootCmd = &cobra.Command{
	Use:   "",
	Short: "命令行的简要描述....",
	Long: `学习使用Cobra,开发cli项目,app: 指的是编译后的文件名。`,
    // 根命令执行方法，如果有要
	//Run: func(cmd *cobra.Command, args []string) {
	//},
}
var (
	cfgFile string // 配置文件
	appConfig *config.AppConfig // 配置对应的结构体
)
// Execute 将所有子命令添加到root命令并适当设置标志。
// 这由 main.main() 调用。它只需要对 rootCmd 调用一次。
func Execute() {
	cobra.CheckErr(rootCmd.Execute())
}

// initConfig reads in config file and ENV variables if set.
func initConfig() {
	// 接收指定的配置文件
	if cfgFile != "" {
		// Use config file from the flag.
		viper.SetConfigFile(cfgFile)
	} else {
		// 设置配置文件目录(可以设置多个,优先级根据添加顺序来)
		viper.AddConfigPath(".")
		viper.AddConfigPath("./config")
		viper.AddConfigPath("./app/config")
		// 设置配置文件
		viper.SetConfigType("yaml")
		viper.SetConfigName("app")
	}
	// 读取环境变量
	viper.AutomaticEnv() // read in environment variables that match
	// 读取配置文件
	if err := viper.ReadInConfig(); err != nil {
		fmt.Printf("viper.ReadInConfig: %v\n",err)
	}
	// 解析配置信息
	err := viper.Unmarshal(&appConfig)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
		return
	}
	fmt.Printf("%+v\n",appConfig)
}
```

#### 2. 使用配置:`cmd/server.go`

```go
package cmd

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/spf13/cobra"
	"os"
)
// serverCmd represents the server command
var (
	serverCmd = &cobra.Command{
		Use:   "server",
		Short: "启动http服务,使用方法: app server?",
		Run: func(cmd *cobra.Command, args []string) {
      // 使用配置
			if appConfig.App.Port == "" {
				fmt.Println("port不能为空!")
				os.Exit(-1)
			}
			engine := gin.Default()
			_ = engine.Run(":" + appConfig.App.Port)
		},
	}
	//接收端口号
	port string
)
func init() {
	// 添加命令
	rootCmd.AddCommand(serverCmd)
	// 接收参数port
	serverCmd.Flags().StringVar(&port, "port", "", "端口号")
}
```

#### 3. 具体配置详情

**`app/config/app.go`**

```go
package config

type AppConfig struct {
	App   app   `yaml:"app"`
	MySql mysql `yaml:"mysql"`
}
type app struct {
	Version string `yaml:"version"`
	Author  string `yaml:"author"`
	Port    string `yaml:"port"`
}
type mysql struct {
	Host     string `yaml:"host"`
	DataBase string `yaml:"data_base"`
	User     string `yaml:"user"`
	Password string `yaml:"password"`
}
```

**`app/config/app.yaml`**

```yaml
app:
  version: v1.0.0
  author: 刘庆辉
  port: 8080
mysql:
  host: 127.0.0.1
  data_base: test
  user: root
  password: root
```

**`./local.yaml`**

```yaml
app:
  version: v1.0.2
  author: 刘庆辉
  port: 8081
mysql:
  host: 192.168.0.10
  data_base: test
  user: root
  password: root
```

### 7.3 编译运行

```bash
# 编译
➜ go build -o cli .

# 默认启动http
➜ ./cli server
&{App:{Version:v1.0.0 Author:刘庆辉 Port:8080} MySql:{Host:127.0.0.1 DataBase: User:root Password:root}}
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] Listening and serving HTTP on :8080

# 指定配置文件启动
➜ ./cli server --config=./local.yaml
&{App:{Version:v1.0.2 Author:刘庆辉 Port:8081} MySql:{Host:192.168.0.10 DataBase: User:root Password:root}}
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] Listening and serving HTTP on :8081
```
