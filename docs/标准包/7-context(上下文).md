### 1.什么是上下文?

从`Go 1.7`开始,标准库引入`context`(上下文)，他主要用来在`goroutine`之间传递上下文信息(同步信号、超时时间、截止时间、`Key-Val`值对)。

### 2. 为什么要用上下文?

#### 2.1 场景说明

在实际开发过程中，我们往往会在`main.goroutine(主协程)`中开启`N 个goroutine(子协程)`来处理其他逻辑，那么`主协程`和 `子协程`之间信号怎么同步呢？列举以下几个需要信号同步的场景。

- 当`主协程`报错或者因为其他原因需要取消时，需要通知`子协程`取消任务。
- 创建`子协程`时,希望指定超时时间或截止时间后，自动取消任务。

#### 2.2 怎么解决？

`context` 包提供了一些函数，协助用户从现有的 `Context` 对象创建新的 `Context` 对象。
这些 `Context` 对象形成一棵树：当一个 `Context` 对象被取消时，继承自它的所有 `Context` 都会被取消。

> `Background` 是所有 `Context` 对象树的根，它不能被取消。

### 3. 上下文接口

`context`包中定义了一个接口`context.Context`,具体定义代码如下:

```go
type Context interface {
  // deadline: 返回取消的时间点(到点Context会自动取消请求)
  // ok：false代表没有设置截止时间
	Deadline() (deadline time.Time, ok bool)
  
  // 返回一个只读的Chan
  // 这个Channel 会在当前工作完成或者上下文被取消后关闭,
  // 多次调用 Done 方法会返回同一个 Channel；
	Done() <-chan struct{}
  
  //返回 context.Context 结束的原因，
  // 它只会在 Done 方法对应的 Channel 关闭时返回非空的值；
	Err() error
	// 从 context.Context 中获取键对应的值
  // 对于同一个上下文来说，多次调用 Value 并传入相同的 Key 会返回相同的结果
	Value(key interface{}) interface{}
}
```



### 4. 创建父上下文 

#### 4.1 签名及说明

**源码**

```go
func Background() Context {
	return background
}
func TODO() Context {
	return todo
}
```

`context` 包中最常用的方法是` context.Background、context.TODO`，这两个方法都会返回预先初始化好的私有变量` background 和 todo`,使用区别如下:

- `Background`: 主要用于`main`函数、初始化以及测试代码中，作为最顶层的`Context`，也就是根`Context`。
- `TODO`:  当我们不知道该使用什么类型的`Context`的时候，可以使用这个。

#### 4.2 使用

```go
func main() {
	// 基于context.Background()为根，创建新的超时上下文
	fmt.Printf("Begin:%s\n",time.Now())
	timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), 3*time.Second)
	// 3秒后自动取消
	defer cancelFunc()
	// 子协程执行取消
	go func(ctx context.Context) {
		fmt.Println("子协程输出...")
	}(timeoutCtx)
	// 等待上下文处于完成状态
	<-timeoutCtx.Done()
	fmt.Printf("等待超时结束：%s \n",time.Now())
}
/**输出()
Begin:2021-04-19 17:10:26.625493 +0800 CST m=+0.000780950
子协程输出...
等待超时结束：2021-04-19 17:10:29.627471 +0800 CST m=+3.002727432 
*/
```

### 5. 创建子上下文

#### 5.1 可取消的(`WithCancel`)

##### 1.签名

```go
// parent 父上下文
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

`context.WithCancel` 函数能够从 `context.Context` 中衍生出一个新的子上下文并返回用于取消该上下文的函数。<font color=red>一旦我们执行返回的取消函数，当前上下文以及它的子上下文都会被取消，所有的 Goroutine 都会同步收到这一取消信号。</font>

##### 2.使用

```go
func main() {
	fmt.Printf("Begin:%s\n",time.Now())
	// 基于context.Background()为根，创建一个可取消的子Context
	cancelCtx, cancelFunc := context.WithCancel(context.Background())
	// 子协程执行取消
	go func(ctx context.Context) {
		for  {
			select {
			case <-ctx.Done():
				fmt.Println("子协程终止,上下文已被取消")
				return
			default:
				time.Sleep(1 * time.Second)
				fmt.Printf("%s: 子协程输出\n",time.Now())
			}
		}
	}(cancelCtx)
	// 5秒后执行取消操作
	time.Sleep(3 * time.Second)
	// 取消上下文，子协程也会停止输出
	cancelFunc()
	// 看看子协程是否停止
	time.Sleep(1 * time.Second)
	fmt.Printf("End：%s \n",time.Now())
}
/*输出
Begin:2021-04-20 11:08:29.77314 +0800 CST m=+0.000635148
2021-04-20 11:08:30.776474 +0800 CST m=+1.003963915: 子协程输出
2021-04-20 11:08:31.780469 +0800 CST m=+2.007953399: 子协程输出
2021-04-20 11:08:32.781569 +0800 CST m=+3.009048079: 子协程输出
子协程终止,上下文已被取消
End：2021-04-20 11:08:33.778543 +0800 CST m=+4.006016052 
*/
```

#### 5.2 指定超时时间(`WithTimeout`)

##### 1.签名

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

`context.WithTimeout` 函数接收父上下文(`parent`)和一个超时时间，能够从 `context.Context` 中衍生出一个新的子上下文并返回用于取消该上下文的函数。执行取消函数时功能和`WithCancel`一样。

##### 2.使用

```go
func main() {
	// 等待超时，自动取消
	waitTimeOut()
	// 子协程主动发送取消信号
	subSendCancel()
}
// 等待超时，自动取消
func waitTimeOut() {
	fmt.Printf("等待超时——> Begin:%s\n", time.Now())
	// 基于context.Background()为根，创建一个3秒后自动取消的子Context
	ctx, cancelFunc := context.WithTimeout(context.Background(), 3*time.Second)
	// 延迟关闭
	defer cancelFunc()
	// 创建子协程
	go func(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("子协程收到取消信号，终止")
				return
			default:
				time.Sleep(1 * time.Second)
				fmt.Printf("子协程输出:%s\n", time.Now())
			}
		}
	}(ctx)
	// 等待超时，自动取消
	<-ctx.Done()
	// 超时后，查看子协程输出
	time.Sleep(2 * time.Second)
	fmt.Printf("End：%s \n", time.Now())
}

// 子协程主动发送取消信号
func subSendCancel() {
	fmt.Printf("子协程主动发送取消信号——> Begin:%s\n", time.Now())
	// 基于context.Background()为根，创建一个可取消的子Context
	cancelCtx, cancelFunc := context.WithTimeout(context.Background(), 5*time.Second)
	// 创建子协程
	go func(cancelFunc context.CancelFunc) {
		fmt.Println("子协程发送->取消信号")
		cancelFunc()
	}(cancelFunc)
	// 同步阻塞
	<-cancelCtx.Done()
	fmt.Printf("End：%s \n", time.Now())
}
/**输出
等待超时——> Begin:2021-04-20 11:43:59.232983 +0800 CST m=+0.000634095
子协程输出:2021-04-20 11:44:00.237187 +0800 CST m=+1.004832384
子协程输出:2021-04-20 11:44:01.240217 +0800 CST m=+2.007856775
子协程输出:2021-04-20 11:44:02.24042 +0800 CST m=+3.008054083
子协程收到取消信号，终止
End：2021-04-20 11:44:04.239543 +0800 CST m=+5.007166176 
子协程主动发送取消信号——> Begin:2021-04-20 11:44:04.239701 +0800 CST m=+5.007323972
子协程发送->取消信号
End：2021-04-20 11:44:04.239809 +0800 CST m=+5.007432189 
*/
```

#### 5.3 指定截止时间(`WithDeadline`)

##### 1.签名

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) 
```

`context.WithDeadline` 函数接收父上下文(`parent`)和一个截止时间，能够从 `context.Context` 中衍生出一个新的子上下文并返回用于取消该上下文的函数。执行取消函数时功能和`WithCancel`一样。

##### 2.使用

```go
func main() {
	fmt.Printf("Begin:%s\n", time.Now())
	// 基于context.Background()为根，创建一个5秒后自动取消的子Context
	deadlineCtx, cancelFunc := context.WithDeadline(context.Background(), time.Now().Add(3*time.Second))
	defer cancelFunc()
	go func(ctx context.Context) {
		for  {
			select {
			case <-ctx.Done():
				fmt.Println("子协程终止,上下文已被取消")
				return
			default:
				time.Sleep(1 * time.Second)
				fmt.Printf("子协程输出: %s\n",time.Now())
			}
		}
	}(deadlineCtx)
	// 同步阻塞，直到上下文被完成或取消
	<-deadlineCtx.Done()
	// 查看子协程是否收到取消信号
	time.Sleep(1 * time.Second)
	fmt.Printf("End：%s \n", time.Now())
}
/**输出
Begin:2021-04-20 13:54:56.224477 +0800 CST m=+0.000826415
子协程输出: 2021-04-20 13:54:57.227495 +0800 CST m=+1.003830376
子协程输出: 2021-04-20 13:54:58.229607 +0800 CST m=+2.005927050
子协程输出: 2021-04-20 13:54:59.230173 +0800 CST m=+3.006503681
子协程输出: 2021-04-20 13:55:00.233766 +0800 CST m=+4.010057104
子协程终止,上下文已被取消
End：2021-04-20 13:55:00.233781 +0800 CST m=+4.010072336 
*/
```

### 6.上下文中传值

#### 6.1 签名

```go
// parent:父上下文
func WithValue(parent Context, key, val interface{}) Context 
```

 `context.WithValue` 能从`父 Context`中创建一个子`子 Context`,并传体一个键值对信息给`子 Context`，在`子 Context`中，通过`context.Value`获取对应的值信息。

#### 6.2 使用

```go
func main() {
	fmt.Printf("Begin:%s\n", time.Now())
	// 基于context.Background()为根，创建一个5秒后自动取消的子Context
	parentCtx, cancelFunc := context.WithCancel(context.Background())
	// 传体信息
	valueCtx := context.WithValue(parentCtx, "name", "张三")
	go func(ctx context.Context,cancelFunc2 context.CancelFunc) {
		// 取出上下文中key=name的值
		fmt.Printf("取出上下文中的name: %v\n",ctx.Value("name"))
		// 子协程发送取消信号
		cancelFunc()
	}(valueCtx,cancelFunc)
	// 等待工作完成或者上下文被取消
	<- parentCtx.Done()
	fmt.Printf("End：%s \n", time.Now())
}
/**输出
Begin:2021-04-20 14:15:57.243469 +0800 CST m=+0.000674192
取出上下文中的name: 张三
End：2021-04-20 14:15:57.243699 +0800 CST m=+0.000904753 
*/
```



