### 1.定义

map是一种集合，可以像遍历数组或切片那样去遍历它。`因为map是由Hash表实现的，所以对map的读取顺序不固定。`


#### 1.1 注意事项

- map是无序的，每次打印出来的map都会不一样，它不能通过index获取，而必须通过key获取。
- `map`的长度是不固定的，和切片一样可以扩展。`内置的len()函数同样适用于map，返回map拥有的键值对的数量。`
- 同一个`map`中`key`必须保证唯一。
- `key`的数据类型必须是可参与比较运算的类型，也就是支持==或!=操作的类型。
- 引用类型则不能作为key的数据类型。
- map的value可以是任何数据类型。
- map和切片一样，也是一种引用类型。



### 2. 声明语法

可以使用`var`关键字来定义map，也可以使用内建函数`make()`。

<font color=red>使用var关键字声明map，未初始化的map的默认值是nil,不能存放键值对。如果要使用map存储键值对，必须在声明时初始化，或者使用make()函数分配到内存空间。</font>

#### 2.1 声明初始化

```go
package main
import "fmt"
func main()  {
	// 声明的同时初始化
	var ageMap = map[string]int {
		"张三": 20,
		"李四": 23,
		"王五": 33,
	}
	fmt.Printf("变量ageMap--> 值: %v 类型: %T \n",ageMap,ageMap)

	// 短变量声明初始化
	ageMap2 := map[string]int{"张三": 20, "李四": 23, "王五": 33,}
	fmt.Printf("变量ageMap2--> 值: %v 类型: %T",ageMap2,ageMap2)
}
/**输出
变量ageMap--> 值: map[张三:20 李四:23 王五:33] 类型: map[string]int 
变量ageMap2--> 值: map[张三:20 李四:23 王五:33] 类型: map[string]int
*/
```

#### 2.2 使用make

```go
package main
import "fmt"
func main()  {
	// 先创建后赋值
	ageMap := make(map[string]int)
	ageMap["张三"] = 20
	ageMap["李四"] = 23
	ageMap["王五"] = 23
	fmt.Printf("变量ageMap-- 值: %v 类型: %T",ageMap,ageMap)
}
// 输出: 变量ageMap--> 值: map[张三:20 李四:23 王五:23] 类型: map[string]int
```

### 3.遍历Map

#### 3.1 遍历基础map

```go
package main
import "fmt"
func main()  {
	// 短变量声明初始化
	ageMap := map[string]int{"张三": 20, "李四": 23, "王五": 33,}
	// 遍历
	for k, v := range ageMap {
		fmt.Printf("k-> 值: %v v--> %v \n",k,v)
	}
}
/**输出
姓名: 张三 年龄: 20 
姓名: 李四 年龄: 23 
姓名: 王五 年龄: 33 
*/
```

#### 3.2 遍历嵌套map

```go
package main

import "fmt"

func main() {
	// 声明嵌套map
	productMap := map[string]map[string]float32{
		"水果": {"香蕉": 3.22, "苹果": 1.88, "葡萄": 2.49},
		"家具": {"桌子": 66.00, "凳子": 12.00, "沙发": 999.00},
		"手机": {"小米10": 3813.00, "华为P40": 5781.00, "iphone12": 7823.00},
	}
	// 遍历
	for class, product := range productMap {
		for name, price := range product {
			fmt.Printf("分类: %s 产品:%s 价格: %0.2f \n",class,name,price)
		}
	}
}
/**输出
分类: 水果 产品:苹果 价格: 1.88 
分类: 水果 产品:葡萄 价格: 2.49 
分类: 水果 产品:香蕉 价格: 3.22 
分类: 家具 产品:桌子 价格: 66.00 
分类: 家具 产品:凳子 价格: 12.00 
分类: 家具 产品:沙发 价格: 999.00 
分类: 手机 产品:小米10 价格: 3813.00 
分类: 手机 产品:华为P40 价格: 5781.00 
分类: 手机 产品:iphone12 价格: 7823.00 
*/
```

### 4.操作map

#### 4.1 判断key是否存在

通过`value, ok := map[key]获知key/value是否存在`。ok是bool型，如果ok是true，则该键值对存在，否则不存在。

```go
package main
import "fmt"
func main() {
	// 声明map
	fruitMap := map[string]float32{"香蕉": 3.22, "苹果": 1.88, "葡萄": 2.49}

	// 存在: ok=true  存在: ok=false
	price,ok := fruitMap["香蕉"]
	if ok {
		fmt.Printf("香蕉存在！价格: %.2f \n",price)
	}
	price2,ok2 := fruitMap["樱桃"]
	if ok2 {
		fmt.Printf("樱桃存在！价格: %.2f \n",price2)
	} else {
		fmt.Printf("樱桃不存在! \n")
	}
	// 简写
	if price,ok := fruitMap["苹果"];ok {
		fmt.Printf("苹果存在！价格: %.2f \n",price)
	}
}
/**输出
  香蕉存在！价格: 3.22 
  樱桃不存在! 
  苹果存在！价格: 1.88 
*/
```



#### 4.2 删除

`delete(map, key) `函数用于删除集合的某个元素，参数为`map`和其对应的`key`。删除函数不返回任何值。

```go
package main
import "fmt"
func main() {
	// 声明map
	fruitMap := map[string]float32{"香蕉": 3.22, "苹果": 1.88, "葡萄": 2.49,"梨":4.13}
	fmt.Printf("删除前-->fruitMap = %v \n",fruitMap)
	// 删除苹果
	delete(fruitMap,"苹果")
	fmt.Printf("删除后-->fruitMap = %v \n",fruitMap)
	// 清空map
	fruitMap = map[string]float32{}
	fmt.Printf("清空后-->fruitMap = %v \n",fruitMap)
}
/**输出
  删除前-->fruitMap = map[梨:4.13 苹果:1.88 葡萄:2.49 香蕉:3.22] 
  删除后-->fruitMap = map[梨:4.13 葡萄:2.49 香蕉:3.22] 
  清空后-->fruitMap = map[] 
*/
```

### 5.map是引用类型

map与切片相似，都是引用类型。将一个map赋值给一个新的变量时，它们指向同一块内存（底层数据结构）。因此，修改两个变量的内容都能够引起它们所指向的数据发生变化。

```go
package main
import "fmt"
func main() {
	// 声明map
	fruitMap := map[string]float32{"香蕉": 3.22, "苹果": 1.88, "葡萄": 2.49,"梨":4.13}
	fmt.Printf("调用函数前-->fruitMap = %v \n",fruitMap)
	// 调用函数
	testMap(fruitMap)
	fmt.Printf("调用函数后-->fruitMap = %v \n",fruitMap)
}
// map是引用类型，函数改变他的值外层变量也会变
func testMap(fruitMap map[string]float32){
	fruitMap["香蕉"] = 5.99
}

/**输出
  调用函数前-->fruitMap = map[梨:4.13 苹果:1.88 葡萄:2.49 香蕉:3.22] 
  调用函数后-->fruitMap = map[梨:4.13 苹果:1.88 葡萄:2.49 香蕉:5.99] 
*/
```

