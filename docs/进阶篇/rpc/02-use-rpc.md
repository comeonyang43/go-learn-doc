## 1. 介绍

`Go`语言标准包(`net/rpc`)已经提供了对`RPC`的支持，而且支持三个级别的RPC：`TCP、HTTP和JSONRPC`。但`Go`语言的`RPC`包是独一无二的`RPC`，它和传统的RPC系统不同，它只支持Go语言开发的服务器与客户端之间的交互，因为在内部，它们采用了`Gob`来编码。



废话不多说,下面会分别实现: `TCP版、HTTP版、JSONRPC版`的RPC。开始撸代码~~

## 2.TCP版

> <font color=red><b>Go语言的RPC规则：方法只能有两个可序列化的参数，其中第二个参数是指针类型，并且返回一个error类型，同时必须是公开的方法。</b></font>

### 2.1 服务端实现

```go
package main

import (
	"log"
	"net"
	"net/rpc"
	"time"
)

type HelloService struct{}

func (h *HelloService) Say(request string, response *string) error {
	format := time.Now().Format("2006-01-02 15:04:05")
	*response = request + " -- " + format
	return nil
}

func main() {
	// 注册服务名称
	_ = rpc.RegisterName("HelloService", new(HelloService))
	// 监听端口
	listen, err := net.Listen("tcp", ":1234")
	if err != nil {
		return
	}
	for {
		// 监听请求
		accept, err := listen.Accept()
		if err != nil {
			log.Fatalf("Accept Error: %s", err)
		}
		go rpc.ServeConn(accept)
	}
}
```

`rpc.RegisterName()`函数调用会将对象类型中所有满足RPC规则的对象方法注册为RPC函数，所有注册的方法会放在`HelloService`服务的空间之下。然后建立一个唯一的TCP链接，并且通过`rpc.ServeConn()`函数在该TCP链接上为对方提供RPC服务。



### 2.2 客户端实现

```go
package main

import (
	"fmt"
	"log"
	"net/rpc"
	"time"
)

func main() {
	// 建立链接
	dial, err := rpc.Dial("tcp", "127.0.0.1:1234")
	if err != nil {
		log.Fatal("Dial error ", err)
	}
	var result string
	for i := 0; i < 5; i++ {
		// 发起请求
		_ = dial.Call("HelloService.Say", "go", &result)
		fmt.Println("result:", result)
		time.Sleep(time.Second * 2)
	}
}
```

首先是通过`rpc.Dial`拨号`RPC`服务，然后通过`client.Call()`调用具体的`RPC`方法。在调用`client.Call()`时，第一个参数是用点号链接的`RPC`服务名字和方法名字，第二个和第三个参数分别是定义RPC方法的两个参数。

### 2.3 启动 & 请求

```bash
# 启动服务
➜ go run  rpc/server/main.go
# 启动客户端
➜ go run rpc/client/main.go 
result: go -- 2022-01-08 21:42:18
result: go -- 2022-01-08 21:42:21
result: go -- 2022-01-08 21:42:23
result: go -- 2022-01-08 21:42:25
result: go -- 2022-01-08 21:42:27
```

## 3. HTTP版

### 3.1 服务端实现

```go
package main

import (
	"fmt"
	"net/http"
	"net/rpc"
)

type MathService struct {
}

// 相乘方法
func (u *MathService) Multi(a int, sum *int) error {
	*sum = a * a
	return nil
}

func main() {
	userService := new(MathService)
	// 注册服务
	err := rpc.Register(userService)
	if err != nil {
		return
	}
	rpc.HandleHTTP()
	err = http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println(err)
	}
}
```




### 3.2 客户端实现

```go
package main

import (
	"fmt"
	"net/rpc"
)

func main() {
	// 建立链接
	client, err := rpc.DialHTTP("tcp", ":8080")
	if err != nil {
		fmt.Println("err ", err)
		return
	}
	// 返回
	var result int
	//  请求方法
	for i := 1; i < 10; i++ {
		err = client.Call("MathService.Multi", i, &result)
		fmt.Printf("i:%v result:%v \n", i, result)
	}
}
```



### 3.3 启动 & 请求

```bash
# 启动服务端
➜ go run rpc/http/server.go

 # 启动客户端
➜ go run rpc/http/client.go
i:1 result:1 
i:2 result:4 
i:3 result:9 
i:4 result:16 
i:5 result:25 
i:6 result:36 
i:7 result:49 
i:8 result:64 
i:9 result:81 
```

## 4. JSON RPC

上面`TCP版`和`HTTP版`,数据编码采用的都是默认的`gob`编码，而`gob`编码是`Go`特有的编码和解码的专用序列化方法，这也就意味着`Gob`无法跨语言使用。而`JSON RPC`则可以跨语言使用。

### 4.1 服务端实现

```go
package main

import (
	"52lu/go-study-example/rpc/jsonrpc/dto"
	"fmt"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
	"time"
)

type MathService struct {
}

// 求和
func (m MathService) Sum(arg *dto.SumParam, res *dto.SumRes) error {
	fmt.Printf("arg: %#v\n", arg)
	*res = dto.SumRes{
		Sum:  arg.A + arg.B,
		Time: time.Now(),
	}
	return nil
}

func main() {
	// 注册服务
	err := rpc.RegisterName("MathService", new(MathService))
	if err != nil {
		fmt.Println("rpc RegisterName err ", err)
		return
	}
	// 监听端口
	listen, err := net.Listen("tcp", ":9090")
	if err != nil {
		fmt.Println("Listen err ", err)
		return
	}
	// 监听
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("conn err ", err)
			return
		}
		// 使用json编码
		go rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
	}
}
```

### 4.2 客户端实现

```go
package main

import (
	"52lu/go-study-example/rpc/jsonrpc/dto"
	"fmt"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

func main() {
	// 建立链接
	conn, err := net.Dial("tcp", ":9090")
	if err != nil {
		fmt.Println(" rpc.Dial err ", err)
		return
	}
	// 使用json编码
	client := rpc.NewClientWithCodec(jsonrpc.NewClientCodec(conn))
	// 参数
	arg := dto.SumParam{A: 20, B: 80}
	// 结果
	var sum dto.SumRes
	err = client.Call("MathService.Sum", &arg, &sum)
	if err != nil {
		fmt.Println("client.Call err ", err)
		return
	}
	fmt.Printf("res:%+v \n", sum)
}
```



### 4.3 启动 & 请求

```bash
# 启动服务端
➜ go run rpc/jsonrpc/server.go
arg: &dto.SumParam{A:20, B:80} # 请求时打印

# 启动客户端
➜ go run rpc/jsonrpc/client.go
res:{Sum:100 Time:2022-01-10 23:12:48.880364 +0800 CST} 
```




