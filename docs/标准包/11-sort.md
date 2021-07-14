### 1.介绍

`sort`包实现了四种基本排序算法：`插入排序、归并排序、堆排序、快速排序`。 但是这四种排序方法是不公开的，它们只能在` sort` 包内部使用。`sort` 包会根据实际数据自动选择高效的排序算法，所以在对数据集合排序时不必考虑应当选择哪一种排序方法，只要实现了 `sort.Interface` 定义的三个方法,就可以顺利对数据集合进行排序：

- `Len() int`: 获取数据集合长度。
- `Less() bool`: 比较两个元素大小。
- `Swap()`: 交换两个元素位置。

#### 1.1 支持功能

`sort` 包对`[]int` 、`[]float64` 、`[]string` 切片提供了完整的支持，主要功能有：

- 对基本数据类型切片的排序。
- 基本数据元素查找。
- 判断基本数据类型切片是否已经排序。
- 对排好序的数据集合逆序。

### 2.排序

#### 2.1 `T.Sort`

##### a. 使用方法

```go
// 步骤1.定义相关类型 sort.IntSlice/Float64Slice/StringSlice
sliceExample := T{..,..,..}
// 步骤2.调用排序方法
sliceExample.Sort()
// 步骤3.判断是否已排序
if sorted := sort.IsSorted(sliceExample);sorted{
	fmt.Printf("排序成功:%+v\n",sliceExample)
} else {
	fmt.Println("排序失败！")
}
```

##### b. 代码示例

```go
package test
import (
	"fmt"
	"sort"
	"testing"
)
func TestIntSliceSort(t *testing.T) {
	// 定义整数序列
	nums := sort.IntSlice{10,40,6,9,23,72,89}
	// 排序
	nums.Sort()
	// 判断是否已排序
	if sorted := sort.IsSorted(nums);sorted{
		fmt.Printf("排序结果:%+v\n",nums)
	} else {
		fmt.Println("排序失败！")
	}
}
/** 输出:
=== RUN   TestIntSliceSort
排序结果:[6 9 10 23 40 72 89]
--- PASS: TestIntSliceSort (0.00s)
PASS
*/
```

#### 2.2 `sort.Ints`

将 `[]int` 进行升序排序。

##### a. 代码示例

```go
// 测试使用
func TestInts(t *testing.T) {
	a := []int{23,12, 9, 2, 12,3, 89}
	// 调用排序
	sort.Ints(a)
	// 检测是否已经排序
	if sorted := sort.IntsAreSorted(a); sorted {
		fmt.Printf("排序成功: %v\n",a)
	} else  {
		fmt.Printf("排序失败: %v\n",a)
	}
	// 查找元素
	fmt.Printf("查找元素(存在): %v 索引:%v \n",12,sort.SearchInts(a, 12))
	fmt.Printf("查找元素(不存在): %v 索引:%v \n",22,sort.SearchInts(a, 22))
}
/** 输出
=== RUN   TestInts
排序成功: [2 3 9 12 12 23 89]
查找元素(存在): 12 索引:3 
查找元素(不存在): 22 索引:5 
--- PASS: TestInts (0.00s)
PASS
*/
```
#### 2.3 `sort.Float64s `

将 `[]float64` 进行升序排序。

##### a. 代码示例

```go
// 浮点型切片排序
func TestFloat64s(t *testing.T) {
	f := []float64{20.2,13.8,28.12,5.23}
	// 排序
	sort.Float64s(f)
	// 检测是否已经排序
	if sorted := sort.Float64sAreSorted(f); sorted {
		fmt.Printf("排序成功: %v\n",f)
	} else  {
		fmt.Printf("排序失败: %v\n",f)
	}
	// 查找元素
	searchVal1 := 13.8
	searchVal2 := 29.23
	fmt.Printf("查找元素(存在): %v 索引:%v \n",searchVal1,sort.SearchFloat64s(f, searchVal1))
	fmt.Printf("查找元素(不存在): %v 索引:%v \n",searchVal2,sort.SearchFloat64s(f, searchVal2))
}
/** 输出
=== RUN   TestFloat64s
排序成功: [5.23 13.8 20.2 28.12]
查找元素(存在): 13.8 索引:1 
查找元素(不存在): 29.23 索引:4 
--- PASS: TestFloat64s (0.00s)
PASS
*/
```

#### 2.4 `sort.Strings `

将 `[]string` 进行升序排序。

##### a. 代码示例

```go
// 字符串型切片排序
func TestSlice(t *testing.T) {
	s := []string{"d", "a", "c", "e", "b"}
	// 排序
	sort.Strings(s)
	// 检测是否已经排序
	if sorted := sort.StringsAreSorted(s); sorted {
		fmt.Printf("排序成功: %v\n",s)
	} else  {
		fmt.Printf("排序失败: %v\n",s)
	}
	// 查找元素
	searchVal1 := "e"
	searchVal2 := "f"
	fmt.Printf("查找元素(存在): %v 索引:%v \n",searchVal1,sort.SearchStrings(s, searchVal1))
	fmt.Printf("查找元素(不存在): %v 索引:%v \n",searchVal2,sort.SearchStrings(s, searchVal2))
}
/** 输出
=== RUN   TestSlice
排序成功: [a b c d e]
查找元素(存在): e 索引:4 
查找元素(不存在): f 索引:5 
--- PASS: TestSlice (0.00s)
PASS
*/
```

#### 2.5 降序

##### a.降序原理

前面介绍的都是排序规则都是升序(`sort`包默认都是升序),如果想要降序，可以使用`sort.Reverse`来调换`Less()`参数，从而实现降序的目的。

**`sort.Reverse 源码`**

```go
// 接收一个Interface类型参数
func Reverse(data Interface) Interface {
	return &reverse{data}
}
// 可以看到这里是把参数 i,j 互换成 j,i
func (r reverse) Less(i, j int) bool {
	return r.Interface.Less(j, i)
}
```

##### c.代码示例

```go
// 测试降序使用
func TestReverseSort(t *testing.T) {
	// 使用sort包中的数据类型
	intSlice := sort.IntSlice{23, 15, 9, 2, 12, 3, 89}
	// 互换j,i
	reverse := sort.Reverse(intSlice)
	// 排序
	sort.Sort(reverse)
	fmt.Printf("排序结果: %v\n", intSlice)
}
/** 输出
=== RUN   TestReverseSort
排序结果: [89 23 15 12 9 3 2]
--- PASS: TestReverseSort (0.00s)
PASS
*/
```



### 3. 查找

#### 3.1 查找步骤

- 步骤一：定义类型变量
- 步骤二：先排序后查找(<font color=red>查找是基于排序后的结果</font>)
- 步骤三：查询可能存在的索引
- 步骤四：判断是否相等

#### 3.2 代码示例

```go
// 查找
func TestSearch(t *testing.T) {
	// 1.定义类型
	intSlice := sort.IntSlice{10, 40, 6, 9, 23, 72, 89}
	// 2.排序后查找
	intSlice.Sort()
	// 3.查询可能存在的索引
	searchVal := 40
	searchIndex := intSlice.Search(searchVal)
	// 4.判断是否相等
	if searchIndex > len(intSlice) {
		fmt.Printf("未找到：%v\n", searchVal)
	} else {
		if searchVal == intSlice[searchIndex] {
			fmt.Printf("找到: %v 索引为: %v \n", searchVal, searchIndex)
		} else {
			fmt.Printf("未找到：%v\n", searchVal)
		}
	}
	// 查找不存在情况
	searchVal2 := 90
	searchIndex2 := intSlice.Search(searchVal2)
	if searchIndex2 >= len(intSlice) {
		fmt.Printf("未找到：%v\n", searchVal2)
	} else {
		if searchVal2 == intSlice[searchIndex2] {
			fmt.Printf("找到: %v 索引为: %v \n", searchVal2, searchIndex2)
		} else {
			fmt.Printf("未找到：%v\n", searchVal2)
		}
	}
}

/** 输出
=== RUN   TestSearch
找到: 40 索引为: 4 
未找到：90
--- PASS: TestSearch (0.00s)
PASS
*/
```

> <font color =red>根据查询出的索引,取出对应的值，看是否与查找的值相等。如果相等则认为找到，不相等则认为没有找到。</font>

