## 1. 什么是RPC

`RPC（Remote Procedure Call`，远程过程调用）<font color=blue><b>是一种进程间的通信方式，它是一种技术思想(概念),而不是规范，它允许程序调用另一个地址空间(通常是共享网络的另一台机器)的过程函数，而开发者无需额外地为这个调用过程编写网络通信相关代码细节。</b></font>


> **<font color=red>通俗的解释：客户端在不知道调用细节的情况下，调用存在于远程计算机上的某个对象，就像调用本地应用程序中的对象一样。</font>**

## 2.远程调用和本地调用



![](./img/image-20211213233936190.png)



## 3. HTTP和RPC区别

### 3.1 从概念区分

> **一句话概括:** `RPC`是概念，`HTTP`是协议，`RPC`通信协议，可以基于`HTTP协议`，也可以基于`TCP、UDP`。

![RPC的传输协议&序列化协议包含](./img/image-20211215000307020.png)

<font cololr=red>`RPC`调用协议通常包含`传输协议 和 序列化协议`,常用的传输协议是`TCP`、常用的序列化协议是`Protobuf`，当然也支持其他协议，具体可见上图。</font>

### 3.2 从传输协议上区分

- `HTTP`只能基于`HTTP协议`，而且`HTTP` 属于应用层协议。
- `RPC`可以基于`HTTP、TCP、UDP协议`，而其中`TCP、UDP`属于传输层协议。

`HTTP`和`RPC`本质的区别，还是在传输协议上，虽然`HTTP`和`RPC`底层都是通过`TCP协议`，但是`RPC`走的是自定义`TCP`协议，两者主要的区别是在传输报文上，

**HTTP1.1协议的tcp报文:**

```html
HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 2021 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36

<html>
  <body>我是有效的消息</body>
</html>
```

**<font color=blue>从上面整个报文可以看出，有效消息`Body`在整个消息中,占比低,而无效消息`Header`占比却很大，即使将`Body`使用二进制编码协议，也无法减少整个报文体积，而`RPC`的自定义`TCP`协议，就是为了去掉报文中无效的信息。</font>**


### 3.3 从OSI网络模型区分

`HTTP`位于应用层协议，`RPC`常用的自定义`TCP`位于传输层协议，完整的`OSI`七层网络模型，如下图:

![](https://pica.zhimg.com/80/v2-dae27091708599d4d7f9b24b2fe06eec_1440w.jpg?source=1940ef5c)



## 4. RPC调用流程图

![RPC调用流程图](./img/image-20220102220036498.png)





## 5. 常见的RPC框架

### 5.1 跟语言绑定框架

- `Dubbo`：国内最早开源的 `RPC` 框架，由阿里巴巴公司开发并于 2011 年末对外开源，仅支持 `Java` 语言。
- `Motan`：微博内部使用的 `RPC` 框架，于 2016 年对外开源，仅支持 `Java` 语言。
- `Tars`：腾讯内部使用的 `RPC` 框架，于 2017 年对外开源，仅支持 `C++` 语言。
- `Spring Cloud`：国外 `Pivotal` 公司 2014 年对外开源的 `RPC` 框架，仅支持 `Java` 语言。

### 5.2 跨语言开源框架

- `gRPC`：`Google` 于 2015 年对外开源的跨语言 RPC 框架，支持多种语言。
- `Thrift`：最初是由` Facebook` 开发的内部系统跨语言的 RPC 框架，2007 年贡献给了 Apache 基金，成为 Apache 开源项目之一，支持多种语言。


