### 1.转换成其他类型（Parse类函数）

#### 1.1 转换成整型(Atoi)

##### 1.语法

```go
func Atoi(s string) (int, error)
```

##### 2.使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	// 将字符串转成整型
	s := "666"
	sInt,err := strconv.Atoi(s)
	if err == nil {
		fmt.Printf("将字符串:%s 转成整型:%d 类型: %T \n",s,sInt,sInt)
	}
}
//将字符串:666 转成整型:666 类型: int
```

#### 1.2 解析成整型(ParseInt)

##### 1.语法

```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
```

`base`:表示进制(2到36)。如果base=0,则根据字符串前缀判断,`0x`:表示十六进制、`0`:表示八进制、其他代表十进制。

`bitSize`: 指定结果必须是不会溢出的整数类型，其值`0、8、16、32、64 分别代表int、int8、int16、int32、int64`

##### 2.使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	// 将字符串转成int
	s := "666"
	i,err := strconv.ParseInt(s,10,0)
	if err != nil {
		panic("类型转换失败")
	}
	fmt.Printf("字符串%s转成int,%d 类型:%T \n", s,i,i)
  // 这里会报错: 将字符串转成int8 int8的范围(-128~127)
	ii,err := strconv.ParseInt(s,10,8)
	if err != nil {
		panic("类型转换int8失败,值溢出")
	}
	fmt.Printf("字符串%s转成int,%d 类型:%T ", s,ii,ii)
}
/**输出
字符串666转成int,666 类型:int64 
panic: 类型转换int8失败,值溢出

goroutine 1 [running]:
main.main()
        /Users/hui/Project/Go/src/go-basic/main.go:20 +0x2dd
*/
```

> `ParseUint`与`ParseInt`类似，但是只用于无符号数字



#### 1.3 解析成浮点型(ParseFloat)

##### 1.语法

```go
func ParseFloat(s string, bitSize int) (float64, error)
```

将一个字符串s解析成浮点数返回。如果字符串s符合语法规则，会返回一个最为接近s值的浮点数。精度由`bitSize`指定，其值`32表示float32、64表示float32`。

##### 2.使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	// 将字符串转成int
	s := "666"
	i,err := strconv.ParseFloat(s,32)
	if err != nil {
		panic("类型转换失败")
	}
	fmt.Printf("字符串%s转成float,%f 类型:%T \n", s,i,i)
}
// 输出:字符串666转成float,666.000000 类型:float64 
```

#### 1.4 解析成布尔型(ParseBool)

##### 1.语法

```go
func ParseBool(str string) (bool, error)
```

返回字符串表示的布尔值。其值如果是:`1,t,T,True,TRUE,true,0,f,F,FALSE,false,F`

| str值                   | 返回结果 |
| ----------------------- | -------- |
| `1,t,T,True,TRUE,true,` | True     |
| `0,f,F,FALSE,false,F`   | False    |
| 其他值                  | false    |

##### 2.使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	strSlice := []string{
		"1","t","T","True","TRUE","true", // 都为true
		"0","f","F","False","FALSE","false", // 都为false
		"9","a","你","+",// 其他值都是false
	}
	for _,v :=  range strSlice {
		b,_ := strconv.ParseBool(v)
		fmt.Printf("当x = %s,返回: %t\n", v,b)
	}
}
/** 输出
当x = 1,返回: true
当x = t,返回: true
当x = T,返回: true
当x = True,返回: true
当x = TRUE,返回: true
当x = true,返回: true
当x = 0,返回: false
当x = f,返回: false
当x = F,返回: false
当x = False,返回: false
当x = FALSE,返回: false
当x = false,返回: false
当x = 9,返回: false
当x = a,返回: false
当x = 你,返回: false
当x = +,返回: false
*/
```



### 2.转换成字符串(Format类函数)

`Format`类函数主要的功能是将其他类型格式化成字符串。

#### 2.1 int转string(Itoa)

##### 1.Itoa源码

```go
// 由源码可知，Itoa是FormatInt(int64(i), 10)的缩写。
func Itoa(i int) string {
	return FormatInt(int64(i), 10)
}
```

##### 2. 使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	i :=100
	s := strconv.Itoa(i)
	fmt.Printf("转换结果: 值: %s 类型: %T \n", s,s)
  // 输出: 转换结果: 值: 100 类型: string
}
```

#### 2.2  int转string(ForamtInt)

```go
// 返回指定基数，i的字符串表示。base取值范围 2 <= base <= 36
func FormatInt(i int64, base int) string
```
##### 1. 使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	var i  int64 =123456
	s2 := strconv.FormatInt(i,2)
	fmt.Printf("base=2(二进制) 转换结果: 值: %s 类型: %T \n", s2,s2)
	s8 := strconv.FormatInt(i,8)
	fmt.Printf("base=8(八进制) 转换结果: 值: %s 类型: %T \n", s8,s8)
	s10 := strconv.FormatInt(i,10)
	fmt.Printf("base=10(十进制) 转换结果: 值: %s 类型: %T \n", s10,s10)
	s16 := strconv.FormatInt(i,16)
	fmt.Printf("base=16(十六进制) 转换结果: 值: %s 类型: %T \n", s16,s16)
}
/**输出
  base=2(二进制) 转换结果: 值: 11110001001000000 类型: string 
  base=8(八进制) 转换结果: 值: 361100 类型: string 
  base=10(十进制) 转换结果: 值: 123456 类型: string 
  base=16(十六进制) 转换结果: 值: 1e240 类型: string 
*/
```
> `ForamtUint`和`ForamtInt`使用方法一样，区别是`ForamtUint`:无符号，`ForamtInt`:有符号。

#### 2.3 Float转string(ForamtFloat)

```go
func FormatFloat(f float64, fmt byte, prec, bitSize int) string
// bitSize: 表示f的来源类型（32:float32 64:float64）,会根据此进行舍入
// fmt: 表示格式标记,（b、e、E、f、g、G）;
  // 格式标记：
    // 'b' (-ddddp±ddd，二进制指数)
    // 'e' (-d.dddde±dd，十进制指数)
    // 'E' (-d.ddddE±dd，十进制指数)
    // 'f' (-ddd.dddd，没有指数)
    // 'g' ('e':大指数，'f':其它情况)
    // 'G' ('E':大指数，'f':其它情况)
  // 如果格式标记为 'e'，'E'和'f'，则 prec 表示小数点后的数字位数
  // 如果格式标记为 'g'，'G'，则 prec 表示总的数字位数（整数部分+小数部分）
// prec：精度
```
##### 1. 使用示例

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	i := 30.1237
	// 如果格式标记为 'e'，'E'和'f'，则 prec 表示小数点后的数字位数
	s1 := strconv.FormatFloat(i,'f',4,32)
	fmt.Printf("保留4位小数-> 值: %s 类型: %T \n", s1,s1)
	// 如果格式标记为 'g'，'G'，则 prec 表示总的数字位数（整数部分+小数部分）
	s2 := strconv.FormatFloat(i,'g',4,32)
	fmt.Printf("总长度返回4位-> 值: %s 类型: %T \n", s2,s2)
}
/**输出
 保留4位小数-> 值: 30.1237 类型: string 
 总长度返回4位-> 值: 30.12 类型: string 
*/
```
#### 2.4   转布尔型(FormatBool)

##### 1.使用示例

```go
package main

import (
	"fmt"
	"strconv"
)
func main() {
	// 返回true字符串
	s1 := strconv.FormatBool(true)
	fmt.Printf("返回true字符串-> 值: %s 类型: %T \n", s1,s1)
	// 返回false字符串
	s2 := strconv.FormatBool(false)
	fmt.Printf("返回false字符串-> 值: %s 类型: %T \n", s2,s2)
}
/*输出
  返回true字符串-> 值: true 类型: string 
  返回false字符串-> 值: false 类型: string 
*/
```

