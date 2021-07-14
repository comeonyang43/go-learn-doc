### 1. 介绍

> `Go`的标准包 `encoding/json`实现了`json对象`的编码和解码。

这篇文章主要讲的是结构体中嵌套匿名字段时的序列化和反序列化，因结构体在参与序列时会有很多细节规则，平时常用的也是结构体，至于`map和slice`可参考历史文章 [Go学习(二十三):JSON编码解析使用](https://mp.weixin.qq.com/s/lmWYxkLU4dV0h-J99Tztzg)

#### 1.1 Go和JSON数据类型

| JSON      | Go                       |
| --------- | ------------------------ |
| `Boolean` | `bool`                   |
| `Number`  | `float64`                |
| `String`  | `string`                 |
| `Array`   | `[]interface{}`          |
| `Object`  | `map[string]interface{}` |
| `NULL`    | `nil`                    |

#### 1.2 结构体序列化规则

**<font color=red>@注意：可导出的字段(首字母大写)，才能参与Json的序列化</font>**

| 标签                          | json的key                                                    |
| ----------------------------- | ------------------------------------------------------------ |
| 有标签,`json:"xx"`            | key=xx                                                       |
| 无标签                        | key=结构体原属性字段                                         |
| 有标签,`json:"-"`             | 会被忽略，不参与序列化                                       |
| 有标签,`json:"xxx,omitempty"` | 代表该字段为空值时,会被忽略。其中`xxx`可以省略，`,`不可以省略。<br>如：`json:",omitempty"` |
| 有标签,`json:"xxx,string"`    | 代表输出类型会转化为字符串。其中`xxx`也可以省略<br><font color=red>它只适用于字符串、浮点数、整数类型的字段</font> |

**a. 代码示例**

```go
type Student struct {
	// 指定json标签时,序列化的key为标签值:name
	Name string `json:"name"`
	// 不指定序列化标签时，key为原属性:Age
	Age  int
	// 当标签值为`json:"-"`,代表改字段会被忽略
	Home string `json:"-"`
	// 标签指定omitempty选项,代表该字段为空值时,会被忽略
	Phone string `json:"phone,omitempty"`
	// 标签指定string选项,代表输出类型会转化为字符串
	// 它只适用于字符串、浮点数、整数类型的字段
	Score float64 `json:"score,string"`
}
```

**b. 运行效果**

```go
func TestMarshal(t *testing.T) {
	// 声明初始化结构体
	student1 := Student{
		Name:  "张三",
		Age:   18,
		Home:  "北京",
		Score:  90.5,
		Phone: "",
	}
	// 序列化
	json1, _ := json.Marshal(student1)
	fmt.Printf("序列化json:%s\n", json1)
}
/**输出
=== RUN   TestMarshal
序列化json: {"name":"张三","Age":18,"score":"90.5"}
--- PASS: TestMarshal (0.00s)
PASS
*/
```


### 2. 匿名字段序列化

#### 2.1 无`JSON`标签

##### a. 字段标签不重复

**<font color=green>School.Name和StudentA.Name，Json标签不一致。</font>**

```go
// 学校
type School struct {
	Name    string `json:"schoolName"`
	Address string `json:"schoolAddress"`
}
// 学生
type StudentA struct {
	Name string `json:"name"`
	// 匿名字段,而且没有json标签
	School 
}
// 序列化-匿名字段 (默认字段不冲突)
func TestAnonymousTagDifferent(t *testing.T) {
	var student = StudentA{
		Name: "小明",
		School:School{
			Name: "北京大学",
			Address: "北京海淀区",
		},
	}
	jsonByte, _ := json.Marshal(student)
	fmt.Printf("json: %s \n",jsonByte)
}
/** 输出
=== RUN   TestAnonymousTagDifferent
json: {"name":"小明","schoolName":"北京大学","schoolAddress":"北京海淀区"} 
--- PASS: TestAnonymousTagDifferent (0.00s)
PASS
*/
```

#####  b. 字段标签重复

**<font color=green>Class.Name和StudentB.Name，Json标签一致，都是 `json:"name"`。</font>**

```go
// 学生
type StudentB struct {
  // 标签名和班级名一样
	Name string `json:"name"` 
	// 匿名字段,而且没有json标签
	Class
}
// 班级
type Class struct {
  // 标签名和学生名一样
	Name string `json:"name"`
	Desc string `json:"desc"`
}
// 序列化-匿名字段 (字段标签冲突)
func TestAnonymousTagSame(t *testing.T) {
	var student = StudentB{
		Name: "小明",
		Class: Class{
			Name: "高二(1)班",
			Desc: "优秀班级",
		},
	}
	jsonByte, _ := json.Marshal(student)
	fmt.Printf("json: %s \n", jsonByte)
}
/** 输出
=== RUN   TestAnonymousTagSame
json: {"name":"小明","desc":"优秀班级"} 
--- PASS: TestAnonymousTagSame (0.00s)
PASS
*/
```

**<font color=red>根据上面代码，得知如果字段标签冲突，冲突的匿名字段会被忽略。</font>**

#### 2.2 有`JSON`标签

当匿名字段设置`json`标签时, 就不会出现冲突的情况，因为序列化后的匿名字段会变成对象。

##### a. 字段标签重复

```go
type Bird struct {
  // json标签一样
	Name string `json:"name"`
	// 匿名字段，有json标签
	Category `json:"category"`
}
type Category struct {
  // json标签一样
	Name string `json:"name"`
}
// 序列化-匿名字段,有json标签
func TestAnonymousWithTag(t *testing.T) {
	b := Bird{
		Name: "喜鹊",
		Category:Category{Name: "鸟类"},
	}
	jsonByte, _ := json.Marshal(b)
	fmt.Printf("json: %s \n", jsonByte)
}
/** 输出
=== RUN   TestAnonymousWithTag
json: {"name":"喜鹊","category":{"name":"鸟类"}} 
--- PASS: TestAnonymousWithTag (0.00s)
PASS
*/
```

##### b. 字段标签不重复

```go
type Bird struct {
	Name string `json:"name"`
	// 匿名字段，有json标签
	Category `json:"category"`
}
type Category struct {
	Name string `json:"categoryName"`
}
// 序列化-匿名字段,有json标签
func TestAnonymousWithTag(t *testing.T) {
	b := Bird{
		Name: "喜鹊",
		Category:Category{Name: "鸟类"},
	}
	jsonByte, _ := json.Marshal(b)
	fmt.Printf("json: %s \n", jsonByte)
}
/** 输出
=== RUN   TestAnonymousWithTag
json: {"name":"喜鹊","category":{"categoryName":"鸟类"}} 
--- PASS: TestAnonymousWithTag (0.00s)
PASS
*/
```

### 3. 匿名字段反序列化

#### 3.1 无`JSON`标签

##### a. 字段标签不重复

```go
type BookClass struct {
	Name  string  `json:"className"`
	Company string `json:"company"`
}
type GoStudy struct {
	Name string `json:"name"`
	Desc string `json:"desc"`
  // 匿名字段无 json标签
	BookClass
}
func TestUnMarshal(t *testing.T) {
	jsonStr := `{"name":"Go语言高级编程","desc":"这是一本Go学习书籍","className":"IT行业书籍","company":"xxx出版社"}`
	var goStudy GoStudy
	err := json.Unmarshal([]byte(jsonStr), &goStudy)
	if err != nil {
		t.Error(err)
		return
	}
	fmt.Printf("反序列化结果: %+v\n",goStudy)
}
/** 输出
=== RUN   TestUnMarshal
反序列化结果: {Name:Go语言高级编程 Desc:这是一本Go学习书籍 BookClass:{Name:IT行业书籍 Company:xxx出版社}}
--- PASS: TestUnMarshal (0.00s)
PASS
*/
```

##### b. 字段标签重复

```go
type BookClass struct {
  // 设置成标签都为 name
	Name  string  `json:"name"`
	Company string `json:"company"`
}
type GoStudy struct {
  //设置成标签都为 name
	Name string `json:"name"`
	Desc string `json:"desc"`
  // 匿名字段无 json标签
	BookClass
}
func TestUnMarshal(t *testing.T) {
	jsonStr := `{"desc":"这是一本Go学习书籍","company":"xxx出版社","name":"IT行业书籍"}`
	var goStudy GoStudy
	err := json.Unmarshal([]byte(jsonStr), &goStudy)
	if err != nil {
		t.Error(err)
		return
	}
	fmt.Printf("反序列化结果: %+v\n",goStudy)
}
/** 输出
=== RUN   TestUnMarshal
反序列化结果: {Name:IT行业书籍 Desc:这是一本Go学习书籍 BookClass:{Name: Company:xxx出版社}}
--- PASS: TestUnMarshal (0.00s)
PASS
*/
```

**<font color=red>@注意: 当字段标签重复时,反序列化会优先给主属性字段赋值。</font>**

#### 3.2 有`JSON`标签

##### a. 字段标签重复

```go
type BookClass struct {
	Name  string  `json:"name"`
	Company string `json:"company"`
}
type GoStudy struct {
	Name string `json:"name"`
	Desc string `json:"desc"`
	// 设置成有json标签
	BookClass `json:"bookClass"`
}
func TestUnMarshal(t *testing.T) {
	jsonStr1 := `{"name":"Go语言高级编程","desc":"这是一本Go学习书籍","company":"xxx出版社"}`
	jsonStr2 := `{"name":"Go语言高级编程","desc":"这是一本Go学习书籍","bookClass":{"name":"IT行业书籍","company":"xxx出版社"}}`
	var res1 GoStudy
	var res2 GoStudy
	_ = json.Unmarshal([]byte(jsonStr1), &res1)
	_ = json.Unmarshal([]byte(jsonStr2), &res2)
	fmt.Printf("res1结果: %+v\n", res1)
	fmt.Printf("res2结果: %+v\n", res2)
}
/** 输出
=== RUN   TestUnMarshal
res1结果: {Name:Go语言高级编程 Desc:这是一本Go学习书籍 BookClass:{Name: Company:}}
res2结果: {Name:Go语言高级编程 Desc:这是一本Go学习书籍 BookClass:{Name:IT行业书籍 Company:xxx出版社}}
--- PASS: TestUnMarshal (0.00s)
PASS
*/
```

##### b. 字段标签不重复

```go
type BookClass struct {
	Name  string  `json:"bookName"`
	Company string `json:"company"`
}
type GoStudy struct {
	Name string `json:"name"`
	Desc string `json:"desc"`
	// 设置成有json标签
	BookClass `json:"bookClass"`
}
func TestUnMarshal(t *testing.T) {
	jsonStr1 := `{"bookName":"Go语言高级编程","desc":"这是一本Go学习书籍","company":"xxx出版社"}`
	jsonStr2 := `{"name":"Go语言高级编程","desc":"这是一本Go学习书籍","bookClass":{"bookName":"IT行业书籍","company":"xxx出版社"}}`
	var res1 GoStudy
	var res2 GoStudy
	_ = json.Unmarshal([]byte(jsonStr1), &res1)
	_ = json.Unmarshal([]byte(jsonStr2), &res2)
	fmt.Printf("res1结果: %+v\n", res1)
	fmt.Printf("res2结果: %+v\n", res2)
}
/** 输出
=== RUN   TestUnMarshal
res1结果: {Name: Desc:这是一本Go学习书籍 BookClass:{Name: Company:}}
res2结果: {Name:Go语言高级编程 Desc:这是一本Go学习书籍 BookClass:{Name:IT行业书籍 Company:xxx出版社}}
--- PASS: TestUnMarshal (0.00s)
PASS
*/
```

### 4. 匿名字段`json`总结

#### 4.1 序列化

##### a. 匿名字段无标签

- 当匿名字段没有指定标签时，序列化后的结构为同级,如`{"..":"..",..}`

- 当匿名属性和主属性的字段标签一样时，序列化会忽略匿名属性的字段。
- 当匿名属性和主属性的字段标签不一样时，序列化不忽略任何字段。

##### b. 匿名字段有标签

- 当匿名字段`a`指定标签时，序列化后的结构为上下级,如`{"..":"..","a":{"xx":"xx"}}`
- 匿名属性和主属性字段，标签无论是否一致，序列化都不会忽略任何字段。

#### 4.2 反序列化

##### a. 匿名字段无标签

- 当匿名字段没有指定标签时，可解析的`JSON`结构,为: `{"..":"..",..}`
- 当匿名属性和主属性的字段标签一样时，会优先将值赋给主属性，匿名属性为类型零值。
- 当匿名属性和主属性的字段标签不一样时，会正常解析。

##### b. 匿名字段有标签

- 当匿名字段指定标签时，可解析的`JSON`结构,为: `{"..":"..","a":{"xx":"xx"}}`
- 匿名属性和主属性字段，标签无论是否一致，反序列化都能正常赋值。

> <font color=red>当结构体中嵌套匿名结构体字段时，在进行序列化和反序列时，推荐为匿名字段加上json标签。</font>









