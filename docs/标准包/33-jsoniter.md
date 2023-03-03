## 1. 介绍

`json-iterator`是一款快且灵活的`JSON`解析器,不但`100%`兼容标准库`encoding/json`,而且比其更快。虽然官网说比标准包`encoding/json`快6倍之多，但是随着`Go`版本的不断迭代，目前平均比`encoding/json`快2倍。

官网给出的性能对比结果如下:



### 1.1 官网性能对比图

![性能图](https://go.liuqh.icu/img/687474703a2f2f6a736f6e697465722e636f6d2f62656e63686d61726b732f676f2d62656e63686d61726b2e706e67.png)

### 1.2 自写程序验证

```go
➜  go-study-example git:(main) ✗ go test -bench=. test/jsoniter_test.go -benchmem
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkUnMarshalIter-12         559245              2126 ns/op            1240 B/op         39 allocs/op
BenchmarkUnMarshalJson-12         218618              5390 ns/op            1448 B/op         33 allocs/op
BenchmarkMarshalIter-12           629608              1988 ns/op            1121 B/op         11 allocs/op
BenchmarkMarshal-12               528918              2295 ns/op            1272 B/op         15 allocs/op
PASS
ok      command-line-arguments  6.027s
```
> **<font color=red>经多次运行，发现反序列化(`Unmarshal`)比标准包快了2.5倍,而序列化(`Marshal`)基本没有什么很高的性能提升。</font>**

### 1.3 为什么比`encoding/json`快？

`jsoniter` 能够比`encoding/json`快的主要原因，

- 减少不必要的内存复制。
- 减少 反射(`reflect`） 的使用，对同一类型的对象，`jsoniter `只反射一次之后即缓存下来。

## 2. 下载

```go
go get github.com/json-iterator/go
```

## 3. 使用

### 3.1 完全兼容`encoding/json`

```go
var json = jsoniter.ConfigCompatibleWithStandardLibrary
```



### 3.2 序列化成`string`

```go
// 序列化成字符串
func TestMarshalToString(t *testing.T) {
	order := Order{
		Id:       10,
		OrderNum: "100200300",
		Money:    99.99,
		PayTime:  time.Now(),
		Extend:   map[string]string{"name": "张三"},
	}
	// 直接转成字符串
	jsonStr, _ := jsoniter.MarshalToString(order)
	fmt.Println("jsonStr:", jsonStr)
}
/**输出
=== RUN   TestMarshalToString
jsonStr: {"id":10,"orderNum":"100200300","money":99.99,"payTime":"2021-12-28T23:44:36.258311+08:00","extend":{"name":"张三"}}
--- PASS: TestMarshalToString (0.00s)
PASS
*/
```

### 3.3 序列化成`[]byte`

```go
// 序列化成[]byte
func TestMarshalToByte(t *testing.T) {
	order := Order{
		Id:       10,
		OrderNum: "100200300",
		Money:    99.99,
		PayTime:  time.Now(),
		Extend:   map[string]string{"name": "张三"},
	}
	// 转成字节[]byte
	marshal, _ := jsoniter.Marshal(order)
	fmt.Println("marshal:", marshal)
}
/**输出
=== RUN   TestMarshalToByte
marshal: [123 34 105 100 34 58 49 48 44 34 111 114 100 101 114 78 117 109 34 58 34 49 48 48 50 48 48 51 48 48 34 44 34 109 111 110 101 121 34 58 57 57 46 57 57 44 34 112 97 121 84 105 109 101 34 58 34 50 48 50 49 45 49 50 45 50 56 84 50 51 58 52 54 58 49 51 46 49 48 53 52 54 54 43 48 56 58 48 48 34 44 34 101 120 116 101 110 100 34 58 123 34 110 97 109 101 34 58 34 229 188 160 228 184 137 34 125 125]
--- PASS: TestMarshalToByte (0.00s)
PASS
*/
```

### 3.4 反序列化

```go
// 反序列化
func TestUnmarshalTmp(t *testing.T) {
	str := `{"id":10,"orderNum":"100200300","money":99.99,"payTime":"2021-12-28T23:44:36.258311+08:00","extend":{"name":"张三"}}`
	var order Order
	// 从字符串反序列化
	_ = jsoniter.UnmarshalFromString(str, &order)
	fmt.Println("order:", order)

	var order2 Order
	// 从[]byte反序列化
	_ = jsoniter.Unmarshal([]byte(str), &order2)
	fmt.Println("order2:", order2)
}
/**输出
=== RUN   TestUnmarshalTmp
order: {10 100200300 99.99 2021-12-28 23:44:36.258311 +0800 CST map[name:张三]}
order2: {10 100200300 99.99 2021-12-28 23:44:36.258311 +0800 CST map[name:张三]}
--- PASS: TestUnmarshalTmp (0.00s)
PASS
*/
```

### 3.5 类型不匹配场景

#### 1. 结构体定义

```go
type Order struct {
	Id       int               `json:"id,omitempty"`
	OrderNum string            `json:"orderNum,omitempty"`
	Money    float64           `json:"money,omitempty"`
	PayTime  time.Time         `json:"payTime"`
	Extend   map[string]string `json:"extend"`
}
```



#### 2. `encoding/json`

```go
// 使用encoding/json反序列化
func TestTypeCovert(t *testing.T) {
	// money 为float64类型，故意设置成字符串
	str := `{"id":10,"orderNum":"100200300","money":"99.99","payTime":"2021-12-28T23:44:36.258311+08:00","extend":{"name":"张三"}}`
	var order Order
	// 使用标准库解析
	err := json.Unmarshal([]byte(str), &order)
	if err != nil {
		fmt.Println("json err:", err)
	}
	fmt.Println("order: ", order)
}
/**输出
=== RUN   TestTypeCovert
json err: json: cannot unmarshal string into Go struct field Order.money of type float64
order:  {10 100200300 0 2021-12-28 23:44:36.258311 +0800 CST map[name:张三]}
--- PASS: TestTypeCovert (0.00s)
PASS
*/
```

#### 3. `json-iterator`

```go
// 使用json-iterator反序列化
func TestTypeCovertWithJsonIter(t *testing.T) {
	// money 为float64类型，故意设置成字符串
	str := `{"id":10,"orderNum":"100200300","money":"99.99","payTime":"2021-12-28T23:44:36.258311+08:00","extend":{"name":"张三"}}`
	var order Order
	// 定义
	var jsonNew = jsoniter.ConfigCompatibleWithStandardLibrary
	// 自适应类型
	extra.RegisterFuzzyDecoders()
	err := jsonNew.Unmarshal([]byte(str), &order)
	if err != nil {
		fmt.Println("jsonNew err:", err)
	}
	fmt.Println("order: ", order)
}
/**输出
=== RUN   TestTypeCovertWithJsonIter
order:  {10 100200300 99.99 2021-12-28 23:44:36.258311 +0800 CST map[name:张三]}
--- PASS: TestTypeCovertWithJsonIter (0.00s)
PASS
*/
```

### 3.6 处理私有属性

```go
// 定义结构体，里面包含私有属性
type Demo struct {
	FirstName string `json:"firstName,omitempty"`
	lastName  string
}

// 解析私有属性
func TestDealPrivate(t *testing.T) {
	d := Demo{
		FirstName: "张",
		lastName:  "三丰",
	}
	// 开启解析私有属性，注：私有属性不能有json标签，否则不能解析
	extra.SupportPrivateFields()
	var jsonNew = jsoniter.ConfigCompatibleWithStandardLibrary
	res, err := jsonNew.MarshalToString(d)
	fmt.Println("序列化-err:", err)
	fmt.Println("序列化-res:", res)
	// 反序列化私有属性
	jsonStr := `{"firstName":"张","lastName":"三丰"}`
	var d2 Demo
	err = jsonNew.UnmarshalFromString(jsonStr, &d2)
	fmt.Println("反序列化-err:", err)
	fmt.Println("反序列化-d2:", d2)
}
/**输出
=== RUN   TestDealPrivate
序列化-err: <nil>
序列化-res: {"firstName":"张","lastName":"三丰"}
反序列化-err: <nil>
反序列化-d2: {张 三丰}
--- PASS: TestDealPrivate (0.00s)
PASS
*/
```

### 3.7 时间类型处理

```go
type timeDemo struct {
	CreateTime time.Time `json:"createTime"`
}
// 解析时间
func TestDealTime(t *testing.T) {
	td := timeDemo{CreateTime: time.Now()}
	var jsonNew = jsoniter.ConfigCompatibleWithStandardLibrary
	// 转成以秒
	extra.RegisterTimeAsInt64Codec(time.Second)
	// 序列化
	res, err := jsonNew.MarshalToString(td)
	fmt.Println("时间序列化-err:", err)
	fmt.Println("时间序列化-res:", res)
	// 反序列化
	str := `{"createTime":1640791445}`
	var tds timeDemo
	err = jsonNew.UnmarshalFromString(str, &tds)
	fmt.Println("时间反序列化-err:", err)
	fmt.Println("时间反序列化-res:", tds)
}
/**输出
type timeDemo struct {
	CreateTime time.Time `json:"createTime"`
}
// 解析时间
=== RUN   TestDealTime
时间序列化-err: <nil>
时间序列化-res: {"createTime":1640792397}
时间反序列化-err: <nil>
时间反序列化-res: {2021-12-29 23:24:05 +0800 CST}
--- PASS: TestDealTime (0.00s)
PASS
*/
```

### 3.8 直接读取`Json`字符串

```go
// 直接读取json 字符串
func TestReadJsonString(t *testing.T) {
	str := `{
    "id":10,
    "extend":{
        "name":"张三"
    },
    "desc":[
        {
            "score":"100"
        },
        {
            "score":"90"
        }
    ]
}`
	fmt.Println("id:", jsoniter.Get([]byte(str), "id").ToInt())
	fmt.Println("extend.name:", jsoniter.Get([]byte(str), "extend", "name").ToString())
	fmt.Println("desc.0.score:", jsoniter.Get([]byte(str), "desc", 0, "score").ToInt())
	fmt.Println("desc.1.score:", jsoniter.Get([]byte(str), "desc", 1, "score").ToInt())
}
/**输出
=== RUN   TestReadJsonString
id: 10
extend.name: 张三
desc.0.score: 100
desc.1.score: 90
--- PASS: TestReadJsonString (0.00s)
PASS
*/
```

### 3.9 大驼峰转下划线

```go
type PayInfo struct {
	OrderId  int
	PayMoney float64 `json:"payMoney"`
}
// 没有json标签的大驼峰属性，会转成下划线变量
func TestSetNamingStrategy(t *testing.T) {
	payInfo := PayInfo{
		OrderId:  100,
		PayMoney: 9.9,
	}
	// -------- 序列化 --------
	// 设置后，没有json标签的属性，会自动转成 xx_xx
	extra.SetNamingStrategy(extra.LowerCaseWithUnderscores)
	res, _ := jsoniter.MarshalToString(payInfo)
	fmt.Println("序列化:", res)
	// -------- 反序列化 --------
	var p PayInfo
	_ = jsoniter.UnmarshalFromString(res, &p)
	fmt.Printf("反序列化:%+v \n", p)
}
/**输出
=== RUN   TestSetNamingStrategy
序列化: {"order_id":100,"payMoney":9.9}
反序列化:{OrderId:100 PayMoney:9.9} 
--- PASS: TestSetNamingStrategy (0.00s)
PASS
*/
```

**<font color=blue>@设置`SetNamingStrategy`后，有`json`标签的使用`json`标签，没有的则会自动转成xx_xx</font>**

## 4. 避坑:`concurrent map writes`

**<font color=red>@设置`extra.RegisterFuzzyDecoders()`时，注意不要在多个协程中重复设置，在init函数执行一次即可，否则会报错：fatal error: concurrent map writes</font>**

### 4.1 错误使用

```go
func TestUseRegisterFuzzyDecodersWithGoroutine(t *testing.T) {
	payInfo := PayInfo{
		OrderId:  100,
		PayMoney: 9.9,
	}
	// 在多个协程中设置extra.RegisterFuzzyDecoders()
	for i := 0; i < 10; i++ {
		go func() {
			jsonNew := jsoniter.ConfigCompatibleWithStandardLibrary
			extra.RegisterFuzzyDecoders()
			_, _ = jsonNew.MarshalToString(payInfo)
		}()
	}
	time.Sleep(time.Second * 1)
	fmt.Println("ok")
}
/**报错
=== RUN   TestUseRegisterFuzzyDecodersWithGoroutine
fatal error: concurrent map writes
fatal error: concurrent map writes
fatal error: concurrent map writes
....
*/
```

### 4.2 正确使用

```go
func init() {
	// 在init中设置一次即可
	extra.RegisterFuzzyDecoders()
}
func TestUseRegisterFuzzyDecodersWithGoroutine(t *testing.T) {
	payInfo := PayInfo{
		OrderId:  100,
		PayMoney: 9.9,
	}
	// 在多个协程中设置extra.RegisterFuzzyDecoders()
	for i := 0; i < 10; i++ {
		go func() {
			jsonNew := jsoniter.ConfigCompatibleWithStandardLibrary
			_, _ = jsonNew.MarshalToString(payInfo)
		}()
	}
	time.Sleep(time.Second * 1)
	fmt.Println("ok")
}
/**输出
=== RUN   TestUseRegisterFuzzyDecodersWithGoroutine
ok
--- PASS: TestUseRegisterFuzzyDecodersWithGoroutine (1.00s)
PASS
*/
```
