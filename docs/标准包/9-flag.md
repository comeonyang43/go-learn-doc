### 1.介绍

在 `Go`中，如果要接收命令行参数，需要使用 `flag`包进行解析。不同的参数类型可以通过不同的方法接收。

### 2.参数接受

#### 2.1 接受方式

使用`flag`接收参数，可以由以下三种方式接受：

- 方式一: `flag.Type(name,defaultVal,desc)`
- 方式二: `flag.TypeVar(&flagVar,name,defaultVal,desc)`
- 方式三: `flag.Var(&flagVar,name,desc)`
- 方式四: `flag.Args()`
- 方式五:`flag.Arg(i)`

| 字段         | 说明                                      |
| ------------ | ----------------------------------------- |
| `flag.Type`  | 这里的`Type`是个抽象的概念,不是具体函数名 |
| `name`       | 代表输入参数名                            |
| `defaultVal` | 默认值                                    |
| `desc`       | 描述信息                                  |

#### 2.2 注意事项

- 上面接收方式一、二、三，都是最后调用`flag.Parse()`,而方式四和方式五是提前调用`flag.Parse()`。
- 方式一、二、三传参时，注意布尔类型，尽量设置成`-flag=val`，否则会影响后面的参数解析。
- <font color=red>方式四、五不能使用`-flag=val`这种格式传参。</font>

### 3.`flag.Type`

####  3.1 代码

```go
package main
import (
	"flag"
	"fmt"
)
var (
	// 接收字符串
	str =  flag.String("name","默认名","用户姓名")
	// 接收整型
	age = flag.Int("age",18,"用户年龄")
	// 接收布尔型
	smoking = flag.Bool("smoking",false,"是否吸烟")
)
func main()  {
	// 解析
	flag.Parse()
	fmt.Printf("str: %v\n",*str)
	fmt.Printf("age: %v\n",*age)
	fmt.Printf("smoking: %v\n",*smoking)
}
```

#### 3.2 编译运行

```bash
# 编译
➜ go build main.go
# 运行(不传参数时)
➜ ./main                                  
str: 默认名
age: 18
smoking: false

# 运行(传参数时)
➜ /main -name 张三 -age 100 -smoking=true
str: 张三
age: 100
smoking: true
```

### 4.`flag.TypeVar`

#### 4.1 代码

```go
package main
import (
	"flag"
	"fmt"
)
// 先定义变量
var (
	name string
	age int
	smoking bool
	weight float64
)
func main()  {
	// 接收字符串
	flag.StringVar(&name,"name","默认名字","用户名")
	// 接收整型
	flag.IntVar(&age,"age",18,"用户年龄")
	// 接收布尔类型
	flag.BoolVar(&smoking,"smoking",false,"是否吸烟？")
	// 接收浮点型
	flag.Float64Var(&weight,"w",60.0,"体重")
	// 解析
	flag.Parse()
	fmt.Printf("姓名: %v\n",name)
	fmt.Printf("年龄: %v\n",age)
	fmt.Printf("是否吸烟？: %v\n",smoking)
	fmt.Printf("体重: %v\n",weight)
}
```

#### 4.2 编译运行

```bash
# 编译
➜ go build main.go

# 运行(不传参数时)
➜  ./main                                  
姓名: 默认名字
年龄: 18
是否吸烟？: false
体重: 60
# 运行(传参数时)
➜ ./main -name 张三 -age 22 -smoking=true  -w=88.9
姓名: 张三
年龄: 22
是否吸烟？: true
体重: 88.9
```

#### 4.3 细节

在传参数过程中，发现布尔型传参如果不设置`=`，会影响后面的参数接收,具体如下:

```bash
# 布尔型不设置 = (发现名字没有被接收)
➜  gin-use git:(dev) ✗ ./main -smoking true -name 哈哈                   
姓名: 默认名字
年龄: 18
是否吸烟？: true
体重: 60

# 布尔型设置 = 
➜  gin-use git:(dev) ✗ ./main -smoking=true -name 哈哈
姓名: 哈哈
年龄: 18
是否吸烟？: true
体重: 60
```

### 5.`flag.Var`

通过 `flag.Var()` 绑定自定义类型，自定义类型需要实现 `Value` 接口(`Receives`必须为指针)

#### 5.1 代码

```go
package main

import (
	"flag"
	"fmt"
	"strings"
)
// 自定义变量，并实现flag.Value接口
type Likes []string
func (l *Likes) String() string {
	return fmt.Sprintf("%v",*l)
}
func (l *Likes)Set(s string) error  {
	// 分割字符串
	split := strings.Split(s, ",")
	*l = split
	return nil
}
func main() {
	var likeList Likes
	// 接收自定义类型
	flag.Var(&likeList,"likes","接收自定义类型")
	// 解析
	flag.Parse()
	fmt.Println(likeList)
}
```

#### 5.2 编译运行

```bash
# 编译
➜  go build main.go 
# 运行
➜  ./main -likes=篮球,足球,游戏
[篮球 足球 游戏]
```

### 6.`flag.Args`

一次打印出全部的入参，注意入参格式不能是`-flag=val`格式。

#### 6.1 代码

```go
package main
import (
	"flag"
	"fmt"
)
func main() {
	// 注意Parse是在Args之前调用
	flag.Parse()
	// 一次接收所有的参数
	args := flag.Args()
	fmt.Println(args)
}
```

#### 6.2 编译运行

```bash
# 编译
➜  go build main.go 
# 错误入参示例
➜  ./main -name=张三 -age=18 -sex=男  
flag provided but not defined: -name
Usage of ./main:
# 正确入参示例
➜ ./main 张三 18 男            
[张三 18 男]
```

### 7.`flag.Arg(i)`

获取指定索引位置的参数，默认索引位置是0;注意入参格式不能是`-flag=val`格式。

#### 7.1 代码

```go
package main
import (
	"flag"
	"fmt"
)
func main() {
	// 注意Parse是在Arg之前调用
	flag.Parse()
	// 获取指定索引位置参数
	p0 := flag.Arg(0)
	p1 := flag.Arg(1)
	p2 := flag.Arg(2)
	fmt.Printf("索引=0，v=%v\n",p0)
	fmt.Printf("索引=1，v=%v\n",p1)
	fmt.Printf("索引=2，v=%v\n",p2)
}
```

#### 7.2 编译运行

```bash
# 编译
➜  go build main.go 
# 传参
➜  ./main 张三 18 男  
索引=0，v=张三
索引=1，v=18
索引=2，v=男
```
