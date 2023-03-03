## 1. 介绍

### 1. 1 什么是`Excelize`

`Excelize` 是 `Go` 语言编写的用于操作 `Office Excel `文档基础库，基于 `ECMA-376，ISO/IEC 29500` 国际标准。可以使用它来读取、写入由 `Microsoft Excel™ 2007` 及以上版本创建的电子表格文档。支持` XLSX / XLSM / XLTM / XLTX` 等多种文档格式，高度兼容带有样式、图片(表)、透视表、切片器等复杂组件的文档，并提供流式读写 `API`，用于处理包含大规模数据的工作簿。可应用于各类报表平台、云计算、边缘计算等系统。使用本类库要求使用的 `Go 语言为 1.15 或更高版本`。

[**源代码: https://github.com/qax-os/excelize **](https://github.com/qax-os/excelize)

[文档:https://xuri.me/excelize/zh-hans/](https://xuri.me/excelize/zh-hans/)

### 1.2 为什么用`Excelize`

在一些情况下我们需要通过程序操作 `Excel` 文档，例如：打开读取已有 `Excel`文档内容、创建新的 `Excel` 文档、基于已有文档（模版）生成新的 `Excel` 文档、向 `Excel `文档中插入图片、图表和表格等元素，有时还需要跨平台实现这些操作。使用 `Excelize` 可以方便的满足上述需求。

### 1.3 性能数据

下面的性能数据展示了创建 `N` 行 `50` 列工作表所执行时间和内存使用情况，其中单元格的值均为 6 个文本字符。测试环境基于普通个人计算机 (OS: macOS Big Sur version 11.2, CPU: 3.4 GHz Intel Core i5, RAM: 16 GB 2400 MHz DDR4, HDD: 1 TB, Go Version: `go1.15.7 darwin/amd64`, Commit: [`23c73ab`](https://github.com/360EntSecGroup-Skylar/excelize/tree/23c73ab527731f9d414e81f7ea15e2ae1a72a290))。具体数据会因机器而异，但趋势应该是一样的。

![写入性能](./img/20210720164600.png)



[更多性能数据查看: https://xuri.me/excelize/zh-hans/performance.html](https://xuri.me/excelize/zh-hans/performance.html)

## 2. 安装

**使用最新版本 Excelize 要求您使用的 Go 语言为 1.15 或更高版本。**

### 2.1 安装命令

```go
go get github.com/xuri/excelize/v2
```

### 2.2 Go Modules 安装

```go
// go get github.com/360EntSecGroup-Skylar/excelize/v2
// 原地址已变更
go get github.com/xuri/excelize/v2
```

## 3. 表格写入

### 3.1 单赋值写入(`SetCellValue`)

```go
package excelize
import (
	"fmt"
	"github.com/360EntSecGroup-Skylar/excelize/v2"
	"testing"
)
// 表格写入(单个赋值)
func TestWriteExcelBySetCellValue(t *testing.T) {
	excel := excelize.NewFile()
	// 创建一个工作表
	sheet := excel.NewSheet("学校")
	// 设置新建工作表(学校)的内容
	_ = excel.SetCellValue("学校", "A2", "北京大学")
	_ = excel.SetCellValue("学校", "A3", "南京大学")
	// 设置sheet1的内容(默认创建)
	excel.SetCellValue("Sheet1","A1","张三")
	excel.SetCellValue("Sheet1","A2","小明")
	// 设置默认工作表
	excel.SetActiveSheet(sheet)
	// 保存表格
	if err := excel.SaveAs("../../tmp/test.xlsx");err != nil {
		t.Error(err)
		return
	}
	fmt.Println("执行完成")
}
```

<img src="./img/20210720114541.png" alt="运行结果" />

### 3.2 按行赋值写入(`SetSheetRow`)

```go
// 表格写入(按行写入)
func TestWriteByLine(t *testing.T) {
	excel := excelize.NewFile()
	// 写入标题
	titleSlice := []interface{}{"序号", "姓名", "年龄", "性别"}
	_ = excel.SetSheetRow("Sheet1", "A1", &titleSlice)
	data := []interface{}{
		[]interface{}{1, "张三", 19, "男"},
		[]interface{}{2, "小丽", 18, "女"},
		[]interface{}{3, "小明", 20, "男"},
	}
	// 遍历写入数据
	for key, datum := range data {
		axis := fmt.Sprintf("A%d", key+2)
		// 利用断言，转换类型
		tmp,_ := datum.([]interface{})
		_ = excel.SetSheetRow("Sheet1", axis, &tmp)
	}
	// 保存表格
	if err := excel.SaveAs("../../tmp/line.xlsx"); err != nil {
		t.Error(err)
		return
	}
	fmt.Println("执行完成")
}
```

![运行结果](./img/20210720153912.png)

### 3.3 按行流式写入(`NewStreamWriter`)

当写入表格的内容很大时，推荐使用流式写入，可以有效防止内存溢出。

```go
// 表格写入(按行流式写入)
func TestWriteByStream(t *testing.T) {
	excel := excelize.NewFile()
	// 获取流式写入器
	streamWriter, err := excel.NewStreamWriter("Sheet1")
	if err != nil {
		t.Error("获取流式写入器失败: " + err.Error())
		return
	}
	// 按行写入
	if err := streamWriter.SetRow("A1", []interface{}{"序号", "商品码", "价格"}); err != nil {
		t.Error("获取流式写入器失败: " + err.Error())
		return
	}
	// 制作数据
	// 设置随机因子
	rand.Seed(time.Now().Unix())
	for i := 2; i < 500000; i++ {
		tmp := []interface{}{
			i,
			fmt.Sprintf("P-%d", rand.Intn(100000000)),
			fmt.Sprintf("%.2f", float64(rand.Intn(10))+rand.Float64()),
		}
		_ = streamWriter.SetRow("A"+strconv.Itoa(i), tmp)
	}
	// 调用 Flush 函数来结束流式写入过程
	if err = streamWriter.Flush(); err != nil {
		t.Error("结束流式写入失败: " + err.Error())
		return
	}
	// 保存表格
	if err := excel.SaveAs("../../tmp/stream.xlsx"); err != nil {
		t.Error(err)
		return
	}
	fmt.Println("执行完成")
}
```

<font color=red>@@注意: 使用流式写入器，必须调用 [`Flush`](https://xuri.me/excelize/zh-hans/stream.html#Flush) 函数来结束流式写入过程，并需要确所保写入的行号是递增的，普通 API 不能与流式 API 混合使用在工作表中写入数据。</font>

**运行耗时:**

```bash
=== RUN   TestWriteByStream
执行完成
--- PASS: TestWriteByStream (2.66s)
PASS
```

## 4. 读取

### 4.1 代码

```go
// 读取表格
func TestRead(t *testing.T) {
	// 打开表格文件
	openFile, err := excelize.OpenFile("../../tmp/line.xlsx")
	if err != nil {
		t.Error("打开表格文件失败: " + err.Error())
		return
	}
	// 读取指定工作表所有数据
	rows, err := openFile.GetRows("Sheet1")
	if err != nil {
		t.Error("读取失败: " + err.Error())
		return
	}
	for _, row := range rows {
		fmt.Printf("%+v\n",row)
	}
	fmt.Println("执行完成!")
}
```

### 4.2 运行结果

```bash
=== RUN   TestRead
[序号 姓名 年龄 性别]
[1 张三 19 男]
[2 小丽 18 女]
[3 小明 20 男]
执行完成!
--- PASS: TestRead (0.00s)
PASS
```

## 5. 更多使用

[更多复杂使用,可查看官方中文文档: https://xuri.me/excelize/zh-hans/](https://xuri.me/excelize/zh-hans/)