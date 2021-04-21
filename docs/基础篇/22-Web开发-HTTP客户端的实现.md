### 1.怎么实现HTTP协议客户端?

`Go`语言标准库内置了`net/http`包，涵盖了`HTTP`客户端和服务端具体的实现方式。内置的`net/http`包提供了最简洁的`HTTP`客户端实现方式，无须借助第三方网络通信库，就可以直接使用`HTTP`中用得最多的`GET`和`POST`方式请求数据。

实现`HTTP`客户端就是客户端通过网络访问向服务端发送请求，服务端发送响应信息，并将相应信息输出到客户端的过程。实现客户端有以下几种方式。

### 2.Get请求

#### 2.1 使用http.NewRequest

##### 1. 源码

```go
// method：GET | POST
// url:
func NewRequest(method, url string, body io.Reader) (*Request, error) {
	return NewRequestWithContext(context.Background(), method, url, body)
}
```

##### 2. 使用示例

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
)
func main() {
	testHttpNewRequestByGet()
}
func testHttpNewRequestByGet()  {
	// 1.创建一个客户端
	client := http.Client{}
	// 2.创建一个请求,请求方式可以是GET，也可以是POST
	request, err := http.NewRequest("GET", "https://api.apiopen.top/getJoke?page=1&count=2&type=text", nil)
	catchError(err)
	// 3.示范添加cookie
	cookie1 := http.Cookie{Name: "uid", Value: "1001"}
	request.AddCookie(&cookie1)
	// 4.发起请求数据
	response, err := client.Do(request)
	catchError(err)
	// 5.解析返回数据
	if response.StatusCode != 200 {
		fmt.Println("网络请求失败！" )
	}
  // 6.读取后需要关闭response.Body
	defer func() {
		response.Body.Close()
	}()
	// 7.读取response.Body
	body, err := ioutil.ReadAll(response.Body)
	catchError(err)
	fmt.Println("请求结果:  " + string(body))
}

// 捕获错误
func catchError(err error)  {
	// 使用延迟函数捕获错误
	defer func() {
		if er := recover(); er != nil {
			fmt.Println(fmt.Sprintf("系统崩溃:%v",er))
		}
	}()
	// 抛出错误
	if err != nil {
		panic(err)
	}
}
```

#### 2.2 使用clent.Get

##### 1. 源码

```go
func (c *Client) Get(url string) (resp *Response, err error) {
	req, err := NewRequest("GET", url, nil)
	if err != nil {
		return nil, err
	}
	return c.Do(req)
}
```

##### 2.  使用示例

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
)
func main() {
	testClientGet()
}
// 测试client.Get方法
func testClientGet() {
	// 1.创建客户端
	client := http.Client{}
	// 2.调用客户端的Get方法
	response, err := client.Get("https://api.apiopen.top/getJoke?page=1&count=2&type=text")
	catchError(err)
	// 3.解析返回数据
	if response.StatusCode != 200 {
		fmt.Println("网络请求失败！")
	}
	// 4.读取后需要关闭response.Body
	defer func() {
		response.Body.Close()
	}()
    // 5.读取response.Body
	res, err := ioutil.ReadAll(response.Body)
	catchError(err)
	// 6.打印输出
	fmt.Printf("返回结果: %s\n",res)
}

// 捕获错误
func catchError(err error) {
	// 使用延迟函数捕获错误
	defer func() {
		if er := recover(); er != nil {
			fmt.Println(fmt.Sprintf("系统崩溃:%v", er))
		}
	}()
	// 抛出错误
	if err != nil {
		panic(err)
	}
}
```

#### 2.3 使用http.Get

##### 1.源码

```go
func Get(url string) (resp *Response, err error) {
	return DefaultClient.Get(url)
}
```

##### 2.使用示例

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	testClientGet()
}
// 测试client.Get方法
func testClientGet() {
	// 1.直接调用http.Get
	response, err := http.Get("https://api.apiopen.top/getJoke?page=1&count=2&type=text")
	catchError(err)
	// 2.解析返回数据
	if response.StatusCode != 200 {
		fmt.Println("网络请求失败！")
	}
	// 3.读取后需要关闭response.Body
	defer func() {
		response.Body.Close()
	}()
  // 4.读取response.Body
	res, err := ioutil.ReadAll(response.Body)
	catchError(err)
	// 5.打印输出
	fmt.Printf("返回结果: %s\n",res)
}

// 捕获错误
func catchError(err error) {
	// 使用延迟函数捕获错误
	defer func() {
		if er := recover(); er != nil {
			fmt.Println(fmt.Sprintf("系统崩溃:%v", er))
		}
	}()
	// 抛出错误
	if err != nil {
		panic(err)
	}
}
```

### 3.Post请求

#### 3.1 使用http.NewRequest

##### 1. 使用示例

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"
)
func main() {
	testHttpNewRequestByPost()
}

func testHttpNewRequestByPost()  {
	// 1.创建一个客户端
	client := http.Client{}
	// 2.添加参数,并将参数转成io.Reader类型
	paramStr := "userName=admin&passWord=1234"
	// 3.创建一个POST请求
	request, err := http.NewRequest("POST", "http://127.0.0.1:8888/login", strings.NewReader(paramStr))
	catchError(err)
	// 4.添加header头
	request.Header.Add("Content-Type","application/x-www-form-urlencoded")
	// 5.设置cookie示例
	request.Header.Set("Cookie","uid=199")
	// 6.发起请求数据
	response, err := client.Do(request)
	catchError(err)
	// 7.解析返回数据
	if response.StatusCode != 200 {
		fmt.Println("网络请求失败！" )
	}
	// 8.读取后需要关闭response.Body
	defer func() {
		response.Body.Close()
	}()
	// 9.读取response.Body
	body, err := ioutil.ReadAll(response.Body)
	catchError(err)
	fmt.Println("请求结果:  " + string(body))
}

// 捕获错误
func catchError(err error)  {
	// 使用延迟函数捕获错误
	defer func() {
		if er := recover(); er != nil {
			fmt.Println(fmt.Sprintf("系统崩溃:%v",er))
		}
	}()
	// 抛出错误
	if err != nil {
		panic(err)
	}
}
```

#### 3.2 使用client.Post 或client.PostForm

##### 1.源码

```go
//client.Post 
func (c *Client) Post(url, contentType string, body io.Reader) (resp *Response, err error) {
	req, err := NewRequest("POST", url, body)
	if err != nil {
		return nil, err
	}
	req.Header.Set("Content-Type", contentType)
	return c.Do(req)
}
// client.PostForm
func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error) {
	return c.Post(url, "application/x-www-form-urlencoded", strings.NewReader(data.Encode()))
}
```

##### 2.使用示例

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
)
func main()  {
	reqType := 2
	// 1.创建一个客户端
	client := http.Client{}
	var response *http.Response
	var err error
	if reqType  == 1{
		// 2.添加参数,参数类型为io.Reader
		paramStr := "userName=admin&passWord=1234"
		// 3.创建一个请求,请求方式可以是GET，也可以是POST
		response, err = client.Post("http://127.0.0.1:8888/login","application/x-www-form-urlencoded", strings.NewReader(paramStr))
	} else  {
		// 2.添加参数,参数类型为url.Values
		param := url.Values{
			"userName": []string{"admin"},
			"passWord": []string{"1234"},
		}
		// 3.创建一个POST请求
		response, err = client.PostForm("http://127.0.0.1:8888/login", param)
	}
	catchError(err)
	// 4.解析返回数据
	if response.StatusCode != 200 {
		fmt.Println("网络请求失败！" )
	}
	// 5.读取后需要关闭response.Body
	defer func() {
		response.Body.Close()
	}()
	// 6.读取response.Body
	body, err := ioutil.ReadAll(response.Body)
	catchError(err)
	fmt.Println("请求结果:  " + string(body))
}

// 捕获错误
func catchError(err error)  {
	// 使用延迟函数捕获错误
	defer func() {
		if er := recover(); er != nil {
			fmt.Println(fmt.Sprintf("系统崩溃:%v",er))
		}
	}()
	// 抛出错误
	if err != nil {
		panic(err)
	}
}
```

#### 3.3 使用 http.Post或http.PostForm

##### 1.源码

```go
// http.Post
func Post(url, contentType string, body io.Reader) (resp *Response, err error) {
	return DefaultClient.Post(url, contentType, body)
}
// http.PostForm
func PostForm(url string, data url.Values) (resp *Response, err error) {
	return DefaultClient.PostForm(url, data)
}
```



##### 2.使用示例

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
)
func main()  {
	reqType := 1
	var response *http.Response
	var err error
	if reqType  == 1{
		// 1.添加参数,参数类型为io.Reader
		paramStr := "userName=admin&passWord=1234"
		// 2.创建一个请求,请求方式可以是GET，也可以是POST
		response, err = http.Post("http://127.0.0.1:8888/login","application/x-www-form-urlencoded", strings.NewReader(paramStr))
	} else  {
		// 1.添加参数,参数类型为url.Values
		param := url.Values{
			"userName": {"admin"},
			"passWord": []string{"1234"},
		}
		// 2.创建一个POST请求
		response, err = http.PostForm("http://127.0.0.1:8888/login", param)
	}
	catchError(err)
	// 3.解析返回数据
	if response.StatusCode != 200 {
		fmt.Println("网络请求失败！" )
	}
	// 4.读取后需要关闭response.Body
	defer func() {
		response.Body.Close()
	}()
	// 5
	//.读取response.Body
	body, err := ioutil.ReadAll(response.Body)
	catchError(err)
	fmt.Println("请求结果:  " + string(body))
}

// 捕获错误
func catchError(err error)  {
	// 使用延迟函数捕获错误
	defer func() {
		if er := recover(); er != nil {
			fmt.Println(fmt.Sprintf("系统崩溃:%v",er))
		}
	}()
	// 抛出错误
	if err != nil {
		panic(err)
	}
}
```

