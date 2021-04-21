### 1.输出控制台

#### 1.1 `Printf`

`Printf`根据`format`参数生成格式化的字符串并写入标准输出。返回写入的字节数和遇到的任何错误。

##### 1.`format`格式

| 动  词 | 功  能                                   |
| ------ | ---------------------------------------- |
| `%v `    | 按值的本来值输出                         |
| `%+v`    | 在 %v 基础上，对结构体字段名和值进行展开 |
| `%#v`    | 输出 Go 语言语法格式的值                 |
| `%T `    | 输出 Go 语言语法格式的类型和值           |
| `%% `    | 输出 % 本体                           |
| `%b `    | 整型以二进制方式显示                     |
| `%o `    | 整型以八进制方式显示                     |
| `%d `    | 整型以十进制方式显示                     |
| `%x `    | 整型以十六进制方式显示                   |
| `%X `    | 整型以十六进制、字母大写方式显示           |
| `%U `    | Unicode 字符                           |
| `%f `    | 浮点数                                 |
| `%p `    | 指针，十六进制方式显示                    |
| `%c `    | 数值对应的 Unicode 编码字符              |
| `%s `    | 直接输出字符串或者[]byte                 |
| `%t `   | 输出值的 true 或 false |

##### 2.打印字符串

```go
str := "hello word"
fmt.Printf("%%s-> %s \n",str)
fmt.Printf("%%v-> %v \n",str)
/**输出
%s-> hello word 
%v-> hello word 
*/
```

##### 3.打印布尔类型

```go
fmt.Printf("%%t-> %t \n", b)
```

##### 4.打印浮点型

| 格式    | 描述               |
| ------- | ------------------ |
| `%f`    | 默认宽度，默认精度 |
| `%9f`   | 宽度9，默认精度    |
| `%.2f`  | 默认宽度，精度2    |
| `%9.2f` | 宽度9，精度2       |
| `%9.f`  | 宽度9，精度0       |

```go
f := 1.234567890
fmt.Printf("默认宽度，默认精度:%f \n", f)
fmt.Printf("宽度9，默认精度:%9f \n", f)
fmt.Printf("默认宽度，精度2:%.2f \n", f)
fmt.Printf("宽度9，精度2:%9.2f \n", f)
fmt.Printf("宽度9，精度0 :%9.f \n", f)
/**输出
默认宽度，默认精度:1.234568 
宽度9，默认精度: 1.234568 
默认宽度，精度2:1.23 
宽度9，精度2:     1.23 
宽度9，精度0 :        1 
*/
```

##### 5.打印整数

```go
// 定义一个整数变量
a := 54321
fmt.Printf("十进制:%d\n",a)
fmt.Printf("二进制:%b\n",a)
fmt.Printf("八进制:%o\n",a)
fmt.Printf("十六进制:%x\n",a)
fmt.Printf("十六进制,字母大写:%X\n",a)
/**输出
十进制:54321
二进制:1101010000110001
八进制:152061
十六进制:d431
十六进制,字母大写:D431
*/
```

##### 6.打印指针

```go
str := "hello"
fmt.Printf("打印指针:%p\n", &str)
//输出: 打印指针:0xc000098d50
```

##### 7.打印变量类型

```go
// 数组
arr := []int{1, 2, 3, 4, 5, 6}
fmt.Printf("arr类型: %T\n", arr)
// 映射
map1 := map[string]int{
	"张三": 20,
	"李四": 23,
}
fmt.Printf("map1类型: %T\n", map1)
// 匿名结构体
fruit := struct {
	name  string
	price float64
}{"香蕉", 4.99}
fmt.Printf("fruit类型: %T\n", fruit)
/**输出
arr类型: []int
map1类型: map[string]int
fruit类型: struct { name string; price float64 }
*/
```

##### 8.打印结构体

```go
// 定义一个匿名结构体
people := struct {
	name, home, school string
}{"张三", "北京", "北京大学"}
fmt.Printf("%v\n", people)
fmt.Printf("%+v\n", people)
fmt.Printf("%#v\n", people)
/**输出
{张三 北京 北京大学}
{name:张三 home:北京 school:北京大学}
struct { name string; home string; school string }{name:"张三", home:"北京", school:"北京大学"}
*/
```

##### 9.打印`Unicode`

```go
fmt.Printf("%U\n", 'A')
fmt.Printf("%c\n", 65)
/**输出
U+0041
A
*/
```

#### 1.2 `Print`

`Print`: 采用默认格式将其参数格式化并写入标准输出。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。

##### 1.打印相邻的字符串

```go
fmt.Print("hello","word","!")
fmt.Print("hello","word",2,3,"!")
// 输出: helloword2 3!helloword!
```

> 如果两个相邻的参数<font color=red>都不是字符串</font>，会在它们的输出之间添加空格,<font color=red>注意的是,不会自动换行。</font>

#### 1.3 `Println`

`Println`: 采用默认格式将其参数格式化并写入标准输出。<font color=red>总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符。</font>

```go
fmt.Println("hello","word",2,3,"!")
fmt.Println("hello","word","!")
/**输出
hello word 2 3 !
hello word !
*/
```

### 2.输出到文件

| 方法名     | 描述                                 |
| ---------- | ------------------------------------ |
| `Fprintln` | 功能和`Println`一样,但可以输出到文件 |
| `Fprintf`  | 功能和`Printf`一样,但可以输出到文件  |
| `Fprint`   | 功能和`Print`一样,但可以输出到文件   |

<font color=red>上述方法，不但可以输出到文件，也可以输出到控制台。</font>

#### 2.1 输出到控制台

```go
_, _ = fmt.Fprintln(os.Stdout, "hello","go")
_, _ = fmt.Fprintln(os.Stdout,  "hello","php")
/** 输出
hello go
hello php
*/
```

#### 2.2 输出到文件

```go
// 打开文件
file, err := os.OpenFile("./test.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, os.ModePerm)
if err != nil {
	fmt.Println(err.Error())
}
// 写入文件
_, err = fmt.Fprintln(file, "hello", "word")
if err != nil {
	fmt.Println(err.Error())
}
```

### 3.生成字符串

| 方法名     | 描述                                                |
| ---------- | --------------------------------------------------- |
| `Sprintf`  | 同`Printf`一样,但是结果不输出到控制台,而是直接返回  |
| `Sprint`   | 同`Print`一样,但是结果不输出到控制台,而是直接返回   |
| `Sprintln` | 同`Println`一样,但是结果不输出到控制台,而是直接返回 |

```go
sprintf := fmt.Sprintf("水果:%s 价格:%.3f", "香蕉", 1.99)
sprint := fmt.Sprint("I", "LOVE", "YOU")
sprintln := fmt.Sprintln("I", "LOVE", "YOU")
fmt.Printf("Sprintf:%s\n",sprintf)
fmt.Printf("sprint:%s\n",sprint)
fmt.Printf("sprintln:%s\n",sprintln)
/**输出
Sprintf:水果:香蕉 价格:1.990
sprint:ILOVEYOU
sprintln:I LOVE YOU
*/

safeConnect
```

### 4.生成错误类型

```go
// 返回错误类型
err := fmt.Errorf("错误信息:%s", "参数不全")
fmt.Printf("类型:%T   内容: %v",err,err)
//输出: 类型:*errors.errorString   内容: 错误信息:参数不全
```

### 5.接收控制台输入

#### 5.1 `Scan`

接收终端输入，按照参数输入顺序赋值，参数间以空格(或换行)间隔，如果输入的参数少于接收的参数则会报错。类型不一致时也会报错。<font color=red>当参数输入数量不够时，换行时会继续等待输入。</font>

```go
func main()  {
	var (
		name   string
		age    int64
		isBody bool
	)
	_, err := fmt.Scan(&name, &age, &isBody)
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	fmt.Printf("name:%s age: %d isBody:%t", name, age, isBody)
}
//启动后，依次输入：张三 22 true
// 输出: name:张三 age: 22 isBody:true
```

#### 5.2 `Scanf`

接收终端输入,按照`format`参数指定的格式接收参数.<font color=red>当参数输入数量不够时，换行时会直接输出.</font>

```go
func main()  {
	var (
		name   string
		age    int
		isBody bool
	)
	_, err := fmt.Scanf("a:%s b:%d c:%t", &name, &age, &isBody)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("name:%s age:%d isBody:%t", name, age, isBody)
}
/*
当输入-> 张三 23 true 报错: input does not match format 
当输入-> a:张三 b:23 c:true 输出: name:张三 age:23 isBody:true
*/
```

#### 5.3 `Scanln`

`Scanln`类似`Scan`，它在遇到换行时才停止扫描。最后一个数据后面必须有换行或者到达结束位置。

```go
func main()  {
	var (
		name   string
		age    int64
		isBody bool
	)
	_, err := fmt.Scanln(&name, &age, &isBody)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("name:%s age: %d isBody:%t", name, age, isBody)
}
/*
当输入-> 张三 23  报错: unexpected newline
当输入-> 李四 74 true 输出: name:李四 age: 74 isBody:true
*/
```

