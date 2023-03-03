## 1. trace说明

在`pprof`的分析中，能够知道一段时间内的`CPU`占用、内存分配、协程堆栈信息。这些信息都是一段时间内数据的汇总，但是它们并没有提供整个周期内发生的事件，例如指定的`Goroutines`何时执行、执行了多长时间、什么时候陷入了堵塞、什么时候解除了堵塞、`GC`如何影响单个`Goroutine`的执行、`STW`中断花费的时间是否太长等。这就是在Go1.5之后推出的`trace`工具的强大之处，它提供了指定时间内程序发生的事件的完整信息，这些事件信息包括：

- 协程的创建、开始和结束。
- 协程的堵塞——系统调用、通道、锁。
- 网络I/O相关事件。
- 系统调用事件。
- 垃圾回收相关事件。

## 2. 收集trace样本

收集`trace`文件的方式和收集`pprof`特征文件的方式非常相似，有两种主要的方式

### 2.1 调用`runtime/trace`包

```go
// 创建文件
file, _ := os.OpenFile("./trace.out", os.O_APPEND|os.O_CREATE, os.ModePerm)
// 开启
if err := trace.Start(file); err != nil {
    fmt.Println("trace start err ",err)
    return
}
// 关闭
defer trace.Stop()
```

### 2.2 调用`net/http/pprof`

`net/http/pprof`库中集成了`trace`的接口，下例获取`30s`内的`trace`事件并存储到`trace.out`文件中。

```bash
curl -o trace.out http://127.0.0.1:6060/debug/pprof/trace?seconds=30
```

### 2.3 在单元测试中使用`-trace`

```go
//  trace.out:是生成的文件名  test/trace_test.go：是测试文件
go test -trace trace.out test/trace_test.go
```

## 3. 分析trace样本

> 分析`trace`样本,语法格式: `go tool trace trace文件`

### 3.1 第一步: 分析文件

```bash
# 分析文件trace2.out，会生成一个链接
➜ go tool trace trace2.out
2021/11/18 19:45:53 Parsing trace...
2021/11/18 19:45:53 Splitting trace...
2021/11/18 19:45:54 Opening browser. Trace viewer is listening on http://127.0.0.1:65154
```

### 3.2 第二步:在浏览器中打开

![分析维度](./img/image-20211118195302295.png)

**分析维度说明**

- `View trace`：查看跟踪
- `Goroutine analysis`：`Goroutine` 分析
- `Network blocking profile`：网络阻塞概况
- `Synchronization blocking profile`：同步阻塞概况
- `Syscall blocking profile`：系统调用阻塞概况
- `Scheduler latency profile`：调度延迟概况
- `User defined tasks`：用户自定义任务
- `User defined regions`：用户自定义区域
- `Minimum mutator utilization`：最低 Mutator 利用率

## 4. `View trace`查看跟踪

### 4.1 数据栏总览

![](./img/image-20211129142153290.png)

**含义说明：**

- 时间线: 显示执行的时间，上图可以看出程序整体运行时间在 `0s~20s`之间，可以用过放大查看具体某个时间点，运行情况。
- `Goroutines`: 显示在执行期间的每个`Goroutine `运行阶段有多少个协程在运行，其包含 GC 等待（`GCWaiting`）、可运行（`Runnable`）、运行中（`Running`）这三种状态。
- 堆内存：包含当前堆使用量(`Allocated`)和下一次垃圾回收的目标大小(`NextGC`)统计。
- 系统线程：显示在执行期间有多少个线程在运行，其包含正在调用 `Syscall（InSyscall）`、运行中（`Running`）这两种状态。
- `GC`: 执行垃圾回收的次数和具体时间点，由上图可以看出每次执行GC时，堆内存都会下降。
- 逻辑处理器: 默认显示系统核心数量，可以通过`runtime.GOMAXPROCS(n)`来控制数量。

> 快捷键:  **`w(放大)、s(缩小)、a(左移)、d(右移)`**

### 4.2 某一时刻协程整体运行情况

![](./img/image-20211129191945502.png)

> **由上图可以看出，这个时刻(`946.662607`):  运行中的协程数量为2、可运行的协程数量为1、等待GC的协程为0**

### 4.3 某一时刻堆内存

![](./img/image-20211129221151476.png)

### 4.4 某一时刻系统线程

![](./img/image-20211129222055337.png)

### 4.5 查看垃圾回收

#### 1.单个垃圾回收

![](./img/image-20211129230729540.png)

#### 2.查看多个垃圾回收

> 选中多个垃圾回收，则可查看多个垃圾回收信息

![](./img/image-20211129231221679.png)


#### 3.部分协程功能说明

![](./img/image-20211130.png)

- **`proc start`**: 代表启动新线程或从系统调用恢复。

- **`proc stop`**: 代表线程在系统调用中被阻塞或线程退出。

- **`GC(dedicated、fractional、idle)`**：在标记阶段`GC`分为三种不同的 `mark worker` 模式; 它们代表着不同的专注程度，其中 `dedicated` 模式最专注，是完整的 GC 回收行为，`fractional `只会干部分的 GC 行为，`idle `最轻松。

- **`MARK ASSIST`**： 在分配内存过程中重新标记内存(`mark the memory`)的goroutine

- **`STW (sweep termination)`**: 代表`STW`扫描阶段终止。

- **`STW (mark termination)`**: 代表`STW`标记阶段终止。

-  **`runtime.gcBgMarkWorker`**： 帮助标记内存的专用后台goroutine

- **`runtime.bgsweep`**: 执行垃圾清理的goroutine。

- **`runtime.bgscavenge`**: gc碎片清理的goroutine。

- **`syscall`**: 代表goroutine在进行系统调用。

- **`sysexit`**: 代表`goroutine`在`syscall`中被取消或阻塞。



### 4.6 查看具体协程运行信息

![](./img/image-20211129183333708.png)

**含义说明：**

| 字段名              | 说明             |
| ------------------- | ---------------- |
| `Start`             | 开始时间         |
| `Wall Duration`     | 持续时间         |
| `Self Time`         | 执行时间         |
| `Start Stack Trace` | 开始时的堆栈信息 |
| `End Stack Trace`   | 结束时的堆栈信息 |
| `Incoming flow`     | 输入流           |
| `Outgoing flow`     | 输出流           |
| `Preceding events`  | 之前的事件       |
| `Following events`  | 之后的事件       |
| `All connected`     | 所有连接的事件   |

## 5. 协程分析(`Goroutine analysis`)

### 5.1 详情查看

![](./img/image-20211130103051365.png)

### 5.2 字段说明

| 名称                    | 含义         | 耗时  |
| :---------------------- | :----------- | :---- |
| `Execution Time`        | 执行时间     | 151µs |
| `Network Wait Time`     | 网络等待时间 | 0ns   |
| `Sync Block Time`       | 同步阻塞时间 | 19s   |
| `Blocking Syscall Time` | 调用阻塞时间 | 0ns   |
| `Scheduler Wait Time`   | 调度等待时间 | 24µs  |
| `GC Sweeping`           | GC 清扫      | 0ns   |
| `GC Pause`              | GC 暂停      | 12ms  |

## 6.用户自定义任务

### 6.1 代码详情

```go
package main

import (
	"context"
	"fmt"
	"os"
	"runtime/trace"
	"sync"
)
func main() {
	file, err := os.Create("./mytask.out")
	if err != nil {
		fmt.Printf("%v\n", err)
		return
	}
	defer file.Close()
	err = trace.Start(file)
	if err != nil {
		fmt.Printf("%v\n", err)
		return
	}
	defer trace.Stop()

	// 创建自定义任务
	ctx, task := trace.NewTask(context.Background(), "myTask")
	defer task.End()
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		gn := i
		// 启动协程
		wg.Add(1)
		go func() {
			defer wg.Done()
			trace.WithRegion(ctx, fmt.Sprintf("goroutine-%d",gn), func() {
				sum := 0
				for n := 0; n < 1000000; n++ {
					sum = sum + n
				}
				fmt.Println("sum = ",sum)
			})
		}()
	}
	wg.Wait()
	fmt.Println("run ok!")
}
```



### 6.2 查看详情

![](./img/image-20211130224039381.png)