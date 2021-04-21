### 1.Map转JSON

```go
package main
import (
	"encoding/json"
	"fmt"
)
func main()  {
	// map里面
	map1 := map[string]string {
		"name":"张三",
		"age":"18",
		"home":"北京",
	}
	// 将map解析成json
	json1, err := json.Marshal(map1)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("json1: %s  类型: %T \n",json1,json1)
  
	map2 := map[string][]string {
		"fruit": {"香蕉","葡萄"},
		"coder":{"PHP","Go","Java"},
		"likes":{"打游戏","看动漫"},
	}
	json2, err := json.Marshal(map2)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("json2: %s  类型: %T \n",json2,json2)
}
/**输出
json1: {"age":"18","home":"北京","name":"张三"}  类型: []uint8 
json2: {"coder":["PHP","Go","Java"],"fruit":["香蕉","葡萄"],"likes":["打游戏","看动漫"]}  类型: []uint8 
*/
```

### 2. Json转Map

```go
package main
import (
	"encoding/json"
	"fmt"
)
func main()  {
	json1 := `{"age":"18","home":"北京","name":"张三"}`
	map1 := make(map[string]string)
	err := json.Unmarshal([]byte(json1), &map1)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("map1: %v T: %T\n",map1,map1)

	json2 := `{"coder":["PHP","Go","Java"],"fruit":["香蕉","葡萄"],"likes":["打游戏","看动漫"]}`
	map2 := make(map[string][]string)
	err = json.Unmarshal([]byte(json2), &map2)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("map2: %v T: %T\n",map2,map2)
}
/**输出
map1: map[age:18 home:北京 name:张三] T: map[string]string
map2: map[coder:[PHP Go Java] fruit:[香蕉 葡萄] likes:[打游戏 看动漫]] T: map[string][]string
*/
```

### 3.结构体转JSON

#### 3.1 无字段标签

结构体转换成`JSON`在开发中经常会用到。`encoding/json`包是通过反射机制来实现编解码的，<font color=red>因此结构体必须导出所转换的字段，没有导出的字段不会被`encoding/json`包解析。</font>

```go
package main
import (
	"encoding/json"
	"fmt"
)
type Student struct {
	Name string
	Age int
	Class string
	phone string //小写字段，则不是会导出
}
func main()  {
	s := Student{
		Name:  "张三",
		Age:   17,
		Class: "三年级",
		phone: "17600112222",
	}
	json1, err := json.Marshal(s)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("json: %s \n",json1)
}
// 输出 json: {"Name":"张三","Age":17,"Class":"三年级"} 
```

#### 3.2 有字段标签

`json`包在解析结构体时，如果遇到`key`为`JSON`的字段标签，则会按照一定规则解析该标签。

```go
package main
import (
	"encoding/json"
	"fmt"
)
// 定义一个学生结构体
type Student struct {
	Name  string `json:"_name"` // 字段标签的语法
	Age   int    `json:"_age"`
	Class string `json:"-"` // 不解析
	Phone string `json:"phone"`
	Score float32 // 没有标签时，则用字段名
}
func main()  {
	s := Student{
		Name:  "张三",
		Age:   17,
		Class: "三年级",
		Phone: "17600112222",
		Score: 88.5,
	}
	json1, err := json.Marshal(s)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("json: %s \n",json1)
}
// 输出-> json: {"_name":"张三","_age":17,"phone":"17600112222","Score":88.5}
```

##### 1.规则总结

- 有标签名时,标签名会作为`key`
- 没有标签时,则会使用字段名。
- 有标签名时，但是标签名的值为`-`，则不会导出

#### 3.3 匿名字段

`json`包在解析匿名字段时，会将匿名字段的字段当成该结构体的字段处理。

```go
package main
import (
	"encoding/json"
	"fmt"
)
type Student struct {
	Name  string `json:"_name"`
	Age   int    `json:"_age"`
    Top // 匿名字段
}
type Top struct {
	Sort int `json:"sort"`
	Score float64
	Class string
}
func main()  {
	s := Student{
		Name: "张三",
		Age:  18,
		Top:  Top{1,99.5,"一年级"},
	}
	json1, err := json.Marshal(s)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("json: %s \n",json1)
}
//输出-> json: {"_name":"张三","_age":18,"sort":1,"Score":99.5,"Class":"一年级"}
```

### 4.JSON转结构体

`JSON`可以转换成结构体。同编码一样，`json`包是通过反射机制来实现解码的，因此结构体必须导出所转换的字段，不导出的字段不会被`json`包解析。另外解析时不区分大小写。

```go
package main
import (
	"encoding/json"
	"fmt"
)
type Student struct {
	Name  string `json:"_name"`
	Age   int    `json:"_age"`
    Top // 匿名字段
}
type Top struct {
	Sort int `json:"sort"`
	Score float64
	Class string
}
func main(){
	student := Student{}
	js := `{"_name":"张三","_age":18,"sort":1,"Score":99.5,"Class":"一年级"}`
	err := json.Unmarshal([]byte(js), &student)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("v: %+v  t: %T \n",student,student)
}
// 输出: v: {Name:张三 Age:18 Top:{Sort:1 Score:99.5 Class:一年级}}  t: main.Student
```

### 5.切片转JSON

```go
package main
import (
	"encoding/json"
	"fmt"
)
func main(){
	student := []string {
		"张三",
		"李四",
		"小米",
	}
	marshal, err := json.Marshal(student)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("v :%s T:%T",marshal,marshal)
}
// 输出 v :["张三","李四","小米"] T:[]uint8
```

### 6.JSON转切片

```go
package main
import (
	"encoding/json"
	"fmt"
)
func main(){
	json1 := `["张三","李四","小米"]`
	var sc []string
	err := json.Unmarshal([]byte(json1), &sc)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("v :%v T:%T",sc,sc)
}
// 输出: v :[张三 李四 小米] T:[]string
```

