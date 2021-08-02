## 1. 介绍

`testing` 包为`Go` 语言提供自动化测试的支持。通过 `go test` 命令来执行单元测试文件，**单元测试文件命名格式为: `xxx_test.go`**,在单元测试文件中,根据测试类型不同可以分为:**功能测试函数、基准测试函数**,区别如下:

| 类型         | 函数格式                     | 作用                 |
| ------------ | ---------------------------- | -------------------- |
| 功能测试函数 | `TestXXX(t *testing.T)`      | 测试函数功能是否正常 |
| 基准测试函数 | `BenchmarkXXX(b *testing.B)` | 测试函数的性能       |

## 2. 功能测试

### 2.1 编写规范

#### 1. 函数格式

```go
// 导入测试包
import "testing"
// 功能测试函数名
func TestName(t *testing.T) {
	t.Log("附加的日志信息")
	// 功能逻辑代码
	if true {
		// 错误时调用t.Error
		t.Error("报告测试失败")
	}
}
```

**整理规则如下:**

- 每个测试函数必须导入`testing`包
- 函数的名字必须以`Test`开头，**可选的后缀名必须以大写字母开头**
- 参数为`t *testing.T`
- 函数没有返回参数

#### 2. 运行格式

通过在`go test`命令后添加`-run`参数，其值对应的是个正则表达式，只有匹配上的函数才会被`go test`命令执行，如下示例:

```bash
# 执行命令
go test -run=? 文件[目录]
```

**`-run值说明`**

| 值                         | 说明                                          |
| -------------------------- | --------------------------------------------- |
| `go test -run=.`           | 执行当前目录下所有测试文件中的`TestXX`函数    |
| `go test -run=Pass`        | 执行当前目录下所有测试文件中的`TestPass*`函数 |
| `go test -run=. a_test.go` | 执行文件`a_test.go`中的`TestXX`函数           |

### 2.2 测试示例

#### 1. 代码详情

**文件名:`go_test.go `**

```go
package test

import (
	"testing"
)

// 通过测试函数
func TestPass(t *testing.T) {
	t.Log("这个是通过测试函数")
}

// 不通过测试函数
func TestFail(t *testing.T) {
	t.Error("运行测试失败！")
}
```

#### 2. 运行全部函数

```bash
➜  go test go_test.go -v
=== RUN   TestPass
    go_test.go:9: 这个是通过测试函数
--- PASS: TestPass (0.00s)
=== RUN   TestFail
    go_test.go:14: 运行测试失败！
--- FAIL: TestFail (0.00s)
FAIL
FAIL    command-line-arguments  1.635s
FAIL
```

<font color=red><b>`go test`命令添加`-v`参数，可以查看测试函数名称和运行时间</b></font>

#### 3. 运行指定函数

<font color=red>`go test`命令后添加`-run`参数，其值对应的是个正则表达式，只有匹配上的函数才会被`go test`命令执行，如下示例:</font>

```bash
# 只执行函数TestPass
➜ go test go_test.go -v -run="Pass"
=== RUN   TestPass
    more_func_test.go:9: 这个是通过测试函数
--- PASS: TestPass (0.00s)
PASS
ok      command-line-arguments  0.889s
# 只执行TestFail函数
➜  go test go_test.go -v -run="Fail"
=== RUN   TestFail
    more_func_test.go:14: 运行测试失败！
--- FAIL: TestFail (0.00s)
FAIL
FAIL    command-line-arguments  0.198s
FAIL
```

### 2.3 子测试

从`Go1.7+`后新增了子测试，我们可以通过使用`t.Run`来执行子测试,具体使用如下:

#### 1. 代码

```go
package test

import (
	"testing"
)

// 子测试使用
func TestSubtest(t *testing.T) {
	// 子测试A
	t.Run("subA", func(t *testing.T) {
		t.Error("subA测试失败")
	})
	// 子测试B
	t.Run("subB", func(t *testing.T) {
		t.Log("subB测试成功！")
	})
}
```

#### 2. 运行

```bash
➜ go test sub_test.go -v                    
=== RUN   TestSubtest
=== RUN   TestSubtest/subA
    sub_test.go:11: subA测试失败
=== RUN   TestSubtest/subB
    sub_test.go:15: subB测试成功！
--- FAIL: TestSubtest (0.00s)
    --- FAIL: TestSubtest/subA (0.00s)
    --- PASS: TestSubtest/subB (0.00s)
FAIL
FAIL    command-line-arguments  0.851s
FAIL
```

## 3. 基准测试

### 3.1 编写规范

#### 1. 函数格式

基准测试就是在一定的工作负载之下检测程序性能的一种方法。基准测试函数的格式如下：

```go
// 导入测试包
import "testing"
// 功能测试函数名
func BenchmarkName(b *testing.B){
   // 被测试代码放到循环内 
    for i:=0;i<b.N;i++{
     // 具体业务函数
   }
}
```

**整理规则如下:**

- 每个测试函数必须导入`testing`包
- 函数的名字必须以`Benchmark`开头，**可选的后缀名必须以大写字母开头**
- 参数为`t *testing.B`
- `b.N`是基准测试框架提供的，表示循环的次数，
- 函数没有返回参数

#### 2. 运行格式

```bash
# 执行命令
go test -bench=? 文件[目录]
```

在`Go`中我们还是通过`go test`来执行基准测试，区别是需要加上参数`-bench=?`,而其中的`?`代表匹配函数名的正则表达式，匹配规则如下:

| `?=`         | 说明                              |
| ------------ | --------------------------------- |
| `-bench=.`   | 代表执行所有函数                  |
| `-bench=Sub` | 代表执行所有`BenchmarkSub*`的函数 |

### 3.2 测试示例

#### 1. 代码详情

```go
package tmp

import (
	"testing"
)

// 变量赋值
func BenchmarkVar(b *testing.B) {
	strSlice := make([]string,10)
	for i := 0; i < b.N; i++ {
		strSlice = append(strSlice,"go")
	}
}
// 字符串拼接
func BenchmarkMulti(b *testing.B) {
	str := ""
	for i := 0; i < b.N; i++ {
		str = str + strconv.Itoa(i)
	}
}
```

<font color=red><b>@注意: 默认情况下，每个基准测试至少运行1秒。如果在Benchmark函数返回时没有到1秒，则b.N的值会增加，并且函数会再次运行。</b></font>

#### 2. 运行全部函数

```bash
# 执行/test/bench_test.go文件中的所有Benchmark*函数
➜ go test -bench=. ./test/bench_test.go 
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkSum-12         1000000000               0.2574 ns/op
BenchmarkMulti-12       1000000000               0.2529 ns/op
PASS
ok      command-line-arguments  1.187s
```

#### 3. 运行指定函数

```bash
# 只匹配BenchmarkSum*的函数
➜ go test -bench=Sum ./test/bench_test.go
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkSum-12         1000000000               0.2588 ns/op
PASS
ok      command-line-arguments  0.457s
```

#### 4. 测试结果说明

```bash
goos: darwin # 执行系统,常用的值：linux, windows, drawin (macOS)
goarch: amd64 # CPU 架构,常用的值 amd64, arm64, i386, armhf
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz # CPU信息
#-12代表对应的 GOMAXPROCS 的值 
BenchmarkVar-12         23692479(执行次数)   48.77 ns/op #(每次耗时48.77ns)
BenchmarkMulti-12         348254          110975 ns/op
PASS
ok      command-line-arguments  43.349s
```

#### 5. 更多维度数据

可以通过添加参数`-benchmem`来获取更多的性能数据，执行如下:

```bash
➜ go test -bench=. ./test/bench_test.go  -benchmem
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkVar-12    18029002   59.87 ns/op    89 B/op      0 allocs/op
BenchmarkMulti-12  330717     102558 ns/op   900342 B/op  2 allocs/op
PASS
ok      command-line-arguments  36.462s
```

| 指标          | 说明                        |
| ------------- | --------------------------- |
| `x ns/op`     | 每次执行耗时`x ns`          |
| `x B/op`      | 每次操作内存分配了`x`字节   |
| `x allocs/op` | 每次操作进行了`x`次内存分配 |

### 3.3 提高精准度

#### 1. 提高运行时间

默认情况下，每个基准测试至少运行1秒。如果在`Benchmark`函数返回时没有到1秒，则`b.N`的值会自增加，并且函数再次运行。如果想运行更长时间，可以通过参数`-benchtime`设置，如`-benchtime=5s代表最少运行5秒`，下面是两种情况的使用方法

**a. 被测代码详情**

```go
package test

import (
	"fmt"
	"testing"
)
// 测试函数Sprintf性能
func BenchmarkCompute(b *testing.B) {
	b.Logf("b.N=%d",b.N)
	for i := 0; i < b.N; i++ {
		_ = fmt.Sprintf("成绩:%d",80)
	}
}
```
**b. 默认运行时间**

```bash
# 默认运行
➜ go test -bench=Compute ./test/bench_test.go              
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkCompute-12     13177644                81.26 ns/op
--- BENCH: BenchmarkCompute-12
    bench_test.go:10: b.N=1
    bench_test.go:10: b.N=100
    bench_test.go:10: b.N=10000
    bench_test.go:10: b.N=1000000
    bench_test.go:10: b.N=13177644
PASS
ok      command-line-arguments  1.556s
```

**b. 设置至少运行时间**

```bash
# 通过参数-benchtime=5s，设置至少运行5秒
➜ go test -bench=Compute ./test/bench_test.go -benchtime=5s
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkCompute-12     69329104                82.30 ns/op
--- BENCH: BenchmarkCompute-12
    bench_test.go:10: b.N=1
    bench_test.go:10: b.N=100
    bench_test.go:10: b.N=10000
    bench_test.go:10: b.N=1000000
    bench_test.go:10: b.N=69329104
PASS
ok      command-line-arguments  6.208s
```

<font color=red><b>@注意:通过上面代码发现不管是运行1.5秒还是运行6.2秒，每次消耗时间没有太大差异，从侧面说明被测函数性能稳定。</b></font>

#### 2. 设置次数运行结果

默认每次都是运行一次基准测试函数活的一次运行的结果,但是可以通过参数`-count`来设置获取运行多次的结果，具体使用如下:

**执行上面示例:**

```bash
# -count=5
➜ go test -bench=Compute ./test/bench_test.go -count=5
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkCompute-12     13069844                81.35 ns/op
BenchmarkCompute-12     14387593                81.51 ns/op
BenchmarkCompute-12     14359526                82.72 ns/op
BenchmarkCompute-12     14074830                83.99 ns/op
BenchmarkCompute-12     14114455                81.53 ns/op
PASS
ok      command-line-arguments  6.523s
```

### 3.4 计时方法

#### 1. 函数列表

| 方法         | 描述         |
| ------------ | ------------ |
| `ResetTimer` | 重置计时器   |
| `StartTimer` | 控制开始计时 |
| `StopTimer`  | 控制停止计时 |

> 进行基准测试之前可能会做一些准备，比如构建测试数据等，这些准备也需要消耗时间，所以需要把这部分时间排除在外。这时候我们可以使用 `ResetTimer` 方法来重置计时器，避免准备数据的耗时对测试数据造成干扰

#### 2. 使用示例

```go
// 重置时间使用
func BenchmarkTime(b *testing.B) {
	// 准备工作
	time.Sleep(time.Second * 3)
	// 重置时间
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		_ = fmt.Sprintf("hello:%v","word")
	}
}
```

### 3.5 并行测试

在基准测试中可以使用`RunParallel`函数,来运行并行测试，它创建多个`goroutine`，并在其中分配`b.N`个迭代。 `goroutine`的数量默认为`GOMAXPROCS`。要想增加非`CPU`基准测试的并行度，可以在调用`RunParallel`之前调用`SetParallelism`。 也可以通过`-cpu=`来设置使用。函数签名具体如下:

```go
func (b *B) RunParallel(body func(*PB))
```

> `body`将在每个`goroutine`中运行。它应该设置任何`goroutine-loca`l状态，然后迭代直到`pb.Next`返回`false`。它不应使用`StartTimer`，`StopTimer`或`ResetTimer`函数，因为它们具有全局作用。它也不应调用运行。

#### 1. 函数格式

```go
func BenchmarkXXX(b *testing.B) {
	// b.SetParallelism(1) // 设置使用的CPU数
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			// 调用具体函数
		}
	})
}
```

#### 2. 代码示例

````go
// 并行测试
func BenchmarkParallel(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			_ = fmt.Sprintf("成绩:%d",80)
		}
	})
}
````

#### 4. 运行测试

```bash
# 不带-cpu
➜ go test -bench=Parallel ./test/bench_test.go
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkParallel-12            50117173                22.84 ns/op
PASS
ok      command-line-arguments  1.788s
# 设置CPU=4
➜ go test -bench=Parallel ./test/bench_test.go -cpu=4
goos: darwin
goarch: amd64
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkParallel-4     35953536                32.66 ns/op
PASS
ok      command-line-arguments  4.240s
```




## 4. 报告函数

 `类型testing.T和testing.B`都继承了类型`testing.common`,`testing.common`常用的报告函数，整理如下:

| 函数名    | 作用                                         |
| --------- | -------------------------------------------- |
| `Fail`    | 测试失败，但是后续代码依然会执行             |
| `FailNow` | 测试失败，并中断代码执行                     |
| `SkipNow` | 跳过测试，中断代码执行，并且不会标识测试失败 |
| `Log`     | 输出信息                                     |
| `Logf`    | 输出格式化的信息                             |
| `Skip`    | 相当于`Log + SkipNow`                        |
| `Skipf`   | 相当于 `Logf + SkipNow`                      |
| `Error`   | 相当于 `Log + Fail`                          |
| `Errorf`  | 相当于 `Logf + Fail`                         |
| `Fatal`   | 相当于` Log + FailNow`                       |
| `Fatalf`  | 相当于 `Logf + FailNow`                      |

<font color=red><b>@注意: 上表中说的测试中断，都是指中断其所在的测试函数。</b></font>