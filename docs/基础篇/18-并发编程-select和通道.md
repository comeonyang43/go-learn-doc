### 1.什么是select？

`select`语句的机制有点像`switch`语句，不同的是，`select`会随机挑选一个可通信的`case`来执行，如果所有`case`都没有数据到达，则执行`default`，如果没有`default`语句，`select`就会阻塞，直到有`case`接收到数据。

### 2. select使用(没有default)

如果没有`default`语句时，`select`就会阻塞，直到有`case`接收到数据。

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	// 创建通道
	intChan := make(chan int)
	stringChan := make(chan string)

	// 写入
	go func() {
		for i := 1; i < 4; i++ {
			intChan <- i
			stringChan <- "张xx-" + strconv.Itoa(i)
		}
	}()
	// 使用select 没有default,则会阻塞等待。（会随机运行一次）
	select {
	case n := <-intChan:
		fmt.Printf("接收到intChan中的数据: %v\n", n)
	case s := <-stringChan:
		fmt.Printf("接收到stringChan中的数据: %v\n", s)
	}
	fmt.Printf("运行结束!")
}

/** 输出：
 接收到intChan中的数据: 1
 运行结束!
*/
```



### 3.select使用(有default)

如果有`default`时，所有`case`都没有数据到达，则执行`default`。

```go
package main
import (
	"fmt"
	"strconv"
)
func main() {
	// 创建通道
	intChan := make(chan int)
	stringChan := make(chan string)

	// 写入
	go func() {
		for i := 1; i < 4; i++ {
			intChan <- i
			stringChan <- "张xx-" + strconv.Itoa(i)
		}
	}()

	// 使用select 没有default,则会阻塞等待。（会随机运行一次）
	select {
	case n := <-intChan:
		fmt.Printf("接收到intChan中的数据: %v\n", n)
	case s := <-stringChan:
		fmt.Printf("接收到stringChan中的数据: %v\n", s)
	default:
		fmt.Println("什么数据都没收到!")
	}
	fmt.Printf("运行结束!")
}
/** 输出：
 什么数据都没收到!
 运行结束!
*/
```

