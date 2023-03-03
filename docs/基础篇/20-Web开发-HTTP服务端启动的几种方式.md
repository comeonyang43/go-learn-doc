### 1. 怎么启动Web服务？

`Go`语言标准库内置的`net/http`包，可以实现`HTTP`服务端。实现`HTTP`服务端就是能够启动`Web`服务，相当于搭建起了一个`Web`服务器。
`http.ListenAndServer()`函数用来启动`Web`服务，绑定并监听`http`端口。

```go
func ListenAndServe(addr string, handler Handler)
// addr：监听地址；如 :8080 或者0.0.0.0:8080
// handler：HTTP处理器Handler
```


### 2.启动Web服务的几种方式

根据不同服务返回的`handler`,常见启动`Web`服务有以下几种方式。

#### 2.1 http.FileServer: 静态文件服务

`http.FileServer()`搭建的服务器只提供静态文件的访问。因为这种`web`服务只支持静态文件访问，所以称之为静态文件服务。

##### 1.使用示例

**文件目录如下:**

![](https://s2.loli.net/2023/02/14/eViarYEfubd5QXj.png)

```go
package main
import (
	"net/http"
)
func main() {
	runFileServer()
}
// 启动一个文件服务器
func runFileServer()  {
	// 如果改路径下有index.html,则会优先显示index.html,否则会看到文件目录
	http.ListenAndServe(":3000",http.FileServer(http.Dir("./public/")))
}
```

<font color=red>注意: 启动的时候，需求使用go run main.go。否则会报404</font>

#### 2.2 http.HandleFunc: 默认的多路由分发服务

`http.HandleFunc()`的作用是注册网络访问的路由。因为它采用的是默认的路由分发任务方式，所以称之为默认的多路由分发服务。

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
// pattern:请求路径的匹配模式
// handler:函数类型，表示这个请求需要处理的事情，其实就是Handler接口中的ServeHTTP()方法。
```

> `ServeHTTP()`方法有两个参数，其中第一个参数是`ResponseWriter`类型，包含了服务器端给客户端的响应数据。服务器端往`ResponseWriter`写入了什么内容，浏览器的网页源码就是什么内容。第二个参数是一个 `*Request`指针，包含了客户端发送给服务器端的请求信息（路径、浏览器类型等）




##### 1.使用示例

```go
package main
import (
	"fmt"
	"net/http"
)
func main() {
	// 绑定路由/hello
	http.HandleFunc("/hello",helloHandle)
	// 绑定路由到/test
	http.HandleFunc("/test",testHandle)
	// 启动服务
	err := http.ListenAndServe(":5000",nil)
	fmt.Println(err)
}
// 处理路由hello
func helloHandle(w http.ResponseWriter, r *http.Request)  {
	fmt.Println("访问路由hello")
	// 解析url参数
	fmt.Println(r.URL.Query())
	w.Write([]byte("hello word!"))
}
// 处理路由test
func testHandle(w http.ResponseWriter, r *http.Request)  {
	fmt.Println("访问路由test")
	// 解析url参数,并输出
	fmt.Println(r.URL.Query())
	w.Write([]byte("test doing!"))
}
// 访问: http://127.0.0.1:5000/test?a=1001
// 访问: http://127.0.0.1:5000/hello?b=990
```

> 通过`http. HandleFunc()`注册网络路由时，`http.ListenAndServer()`的第二个参数通常为`nil`，这意味着服务端采用默认的`http.DefaultServeMux`进行分发处理。

#### 2.3 http.NewServeMux(): 自定义多路由分发服务

`http.NewServeMux()`的作用是注册网络访问的多路路由。因为它采用的是自定义的多路由分发任务方式，所以称之为自定义多路由分发服务。

注册网络路由时，如果`http.ListenAndServer()`的第二个参数为`nil`，那么表示服务端采用默认的`http.DefaultServeMux`进行分发处理。也可以自定义`ServeMux`。`ServeMux`结构体如下

```go
// ServeMux结构体源码
type ServeMux struct {
	mu    sync.RWMutex // 锁,由于请求涉及到并发处理，因此这里有个锁机制 
	m     map[string]muxEntry // 存放具体的路由信息 
	es    []muxEntry // 按照路由长度从大到小的存放处理函数
	hosts bool       // 标记路由中是否带有主机名
}
// muxEntry是路由的具体条目
type muxEntry struct {
	h       Handler // 处理函数
	pattern string  // 路由路径
}
```



### 3. 自定义多路由实践

```go
package main
import (
	"fmt"
	"net/http"
)
// 定义一个接口体，用来实现http.Handler
type MyRoute struct {
}
// 实现http.Handler接口中的ServeHTTP方法
func (m *MyRoute)ServeHTTP(w http.ResponseWriter,r *http.Request)  {
	path := r.URL.Path
	fmt.Println(path)
	switch path {
	case  "/":
		w.Write([]byte("首页"))
	case "/hello":
		w.Write([]byte("say hello"))
	case "/test":
		w.Write([]byte("test doing"))
	default:
		http.NotFound(w,r)
	}
	return
}
func main() {
	myRoute := &MyRoute{}
	http.ListenAndServe(":10000",myRoute)
}
```

#### 3.1 代码执行流程

使用`http.ListenAndServe(":10000",myRoute)`启动服务之后，会发生以下操作

##### 1.实例化`http.Server`,并调用`ListenAndServe()`

```go
func ListenAndServe(addr string, handler Handler) error {
  // 实例化 Server
	server := &Server{Addr: addr, Handler: handler}
  // 调用 ListenAndServe()
	return server.ListenAndServe()
}
```

##### 2.监听端口

```go
   func (srv *Server) ListenAndServe() error {
   	if srv.shuttingDown() {
   		return ErrServerClosed
   	}
   	addr := srv.Addr
   	if addr == "" {
   		addr = ":http"
   	}
     // 监听端口
   	ln, err := net.Listen("tcp", addr)
   	if err != nil {
   		return err
   	}
     // 启动服务
   	return srv.Serve(ln)
   }
```

##### 3.启动for无限循环,在循环体中`Accept`请求，并开启` goroutine`为这个请求服务

```go
func (srv *Server) Serve(l net.Listener) error {
  	//...省略N行代码
	for {
		rw, err := l.Accept()
		if err != nil {
		 	//...省略N行代码
		}
		connCtx := ctx
		if cc := srv.ConnContext; cc != nil {
			connCtx = cc(connCtx, rw)
			if connCtx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew)
    // 开启goroutine为这个请求服务
		go c.serve(connCtx)
	}
}
```

##### 4.读取每个请求内容,并调用`ServeHTTP`

```go
func (c *conn) serve(ctx context.Context) {
	//...省略N行代码
	for {
    // 读取每个请求内容
		w, err := c.readRequest(ctx)
    //...省略N行代码
    
    // 调用ServeHTTP
		serverHandler{c.server}.ServeHTTP(w, w.req)
		//...省略N行代码
} 
```

##### 5. 判断`handler`是否为空,如果为空则把`handler`设置成`DefaultServeMux`。
```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
    // 如果为空则把handler设置成DefaultServeMux。
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
  // 上述示例中，传的是&MyRoute，所以会调用MyRoute.ServeHTTP
	handler.ServeHTTP(rw, req)
}
```