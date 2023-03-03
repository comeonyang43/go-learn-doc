## 1. 介绍

`gRPC`建立在`HTTP/2`协议之上，对`TLS`提供了很好的支持。当不需要证书认证时,可通过`grpc.WithInsecure()`选项跳过了对服务器证书的验证，没有启用证书的`gRPC`服务和客户端进行的是明文通信，信息面临被任何第三方监听的风险。为了保证`gRPC`通信不被第三方监听、篡改或伪造，可以对服务器启动`TLS`加密特性。

## 2. 概念简述

### 2.1 什么是CA

`CA`是`Certificate Authority`的缩写，也叫“证书授权中心”。它是负责管理和签发证书的第三方机构，作用是检查证书持有者身份的合法性，并签发证书，以防证书被伪造或篡改。

> `CA`实际上是一个机构，负责“证件”印制核发。就像负责颁发身份证的公安局、负责发放行驶证、驾驶证的车管所。

### 2.2 什么是CA证书

`CA` 证书就是`CA`颁发的证书。我们常听到的数字证书就是`CA证书`,`CA证书`包含信息有:**证书拥有者的身份信息，CA机构的签名，公钥和私钥**,

- 身份信息: 用于证明证书持有者的身份
- CA机构的签名: 用于保证身份的真实性
- 公钥和私钥: 用于通信过程中加解密，从而保证通讯信息的安全性

### 2.3 什么是`SAN`

`SAN(Subject Alternative Name) `是 `SSL` 标准 `x509` 中定义的一个扩展。使用了 `SAN` 字段的 `SSL` 证书，可以扩展此证书支持的域名，使得一个证书可以支持多个不同域名的解析。

### 2.4 为什么需要`SAN`

先看下不通过`SAN`生成的证书，会报的错误信息。

> `rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed: x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0"`

<font color=red>出现上述错误，原因是因为 从go 1.15 版本开始[废弃 CommonName](https://golang.org/doc/go1.15#commonname)，因此推荐使用 SAN 证书。</font>

## 3. 证书生成流程

### 3.1 生成CA证书

#### 1. 新增`ca.conf`

```ini
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
countryName                 = GB
countryName_default         = BeiJing
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = JiangSu
localityName                = Locality Name (eg, city)
localityName_default        = NanJing
organizationName            = Organization Name (eg, company)
organizationName_default    = Step
commonName                  = liuqh.icu
commonName_max              = 64
commonName_default          = liuqh.icu
```

#### 2. 生成CA私钥

```bash
# 生成CA私钥
$ openssl genrsa -out ca.key 4096
```

#### 3. 生成CA证书

```bash
$ openssl req -new -x509 -days 365 -subj "/C=GB/L=Beijing/O=github/CN=liuqh.icu" \
-key ca.key -out ca.crt -config ca.conf
```

- `C=GB`: `C`代表的是国家名称代码。
- `L=Beijing`: 代表地方名称,例如城市。
- `O=gobook`: 代表组织单位名称。
- `CN=liuqh.icu`: 代表关联的域名，

> [更多参数含义:https://www.digicert.com/kb/ssl-support/openssl-quick-reference-guide.htm#CreatingYourCSR](https://www.digicert.com/kb/ssl-support/openssl-quick-reference-guide.htm#CreatingYourCSR)

### 3.2 生成服务端证书

#### 1. 新增`server.conf`

```ini
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = CN
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = JiangSu
localityName                = Locality Name (eg, city)
localityName_default        = NanJing
organizationName            = Organization Name (eg, company)
organizationName_default    = Step
commonName                  = CommonName (e.g. server FQDN or YOUR name)
commonName_max              = 64
commonName_default          = XXX(自定义,客户端需要此字段做匹配)
[ req_ext ]
subjectAltName = @alt_names
[alt_names]
DNS.1   = liuqh.icu
IP      = 127.0.0.1
```

#### 2.生成公私钥

```bash
$ openssl genrsa -out server.key 2048
```

#### 3. 生成`CSR`

```bash
$ openssl req -new  -subj "/C=GB/L=Beijing/O=github/CN=liuqh.icu" \
-key server.key -out server.csr -config server.conf
```

#### 4. 基于CA签发证书

```bash
$ openssl x509 -req -sha256 -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 \
-in server.csr -out server.crt -extensions req_ext -extfile server.conf
```

### 3.4 生成客户端证书

#### 1. 生成公私钥

```bash
$ openssl genrsa -out client.key 2048
```

#### 2. 生成`CSR`

```bash
$ openssl req -new -subj "/C=GB/L=Beijing/O=github/CN=liuqh.icu"  \
-key client.key -out client.csr 
```

#### 3.基于CA签发证书

```bash
$ openssl x509 -req -sha256 -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 \
-in client.csr -out client.crt
```

## 4. 目录结构

![](/img/image-20220224121150599.png)



## 5.代码实现

### 5.1 服务端

```go
package main

import (
	"52lu/go-rpc/server/tslservice"
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	"io/ioutil"
	"net"
)

func main() {
	// 公钥中读取和解析公钥/私钥对
	pair, err := tls.LoadX509KeyPair("./pem/server.crt", "./pem/server.key")
	if err != nil {
		fmt.Println("LoadX509KeyPair error", err)
		return
	}
	// 创建一组根证书
	certPool := x509.NewCertPool()
	ca, err := ioutil.ReadFile("./pem/ca.crt")
	if err != nil {
		fmt.Println("read ca pem error ", err)
		return
	}
	// 解析证书
	if ok := certPool.AppendCertsFromPEM(ca); !ok {
		fmt.Println("AppendCertsFromPEM error ")
		return
	}
	cred := credentials.NewTLS(&tls.Config{
		Certificates: []tls.Certificate{pair},
		ClientAuth:   tls.RequireAndVerifyClientCert,
		ClientCAs:    certPool,
	})
	grpcServer := grpc.NewServer(grpc.Creds(cred))
	// 注册服务
	tslservice.RegisterTslServiceServer(grpcServer, new(tslservice.UnimplementedTslServiceServer))
	// 监听端口
	listen, err := net.Listen("tcp", ":1234")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("服务启动成功....")
	// 启动服务
	grpcServer.Serve(listen)
}
```

### 5.2 客户端

```go
package main

import (
	"52lu/go-rpc/server/tslservice"
	"context"
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	wrapperspb "google.golang.org/protobuf/types/known/wrapperspb"
	"io/ioutil"
)

func main() {
	// 公钥中读取和解析公钥/私钥对
	pair, err := tls.LoadX509KeyPair("./pem/client.crt", "./pem/client.key")
	if err != nil {
		fmt.Println("LoadX509KeyPair error ", err)
		return
	}
	// 创建一组根证书
	certPool := x509.NewCertPool()
	ca, err := ioutil.ReadFile("./pem/ca.crt")
	if err != nil {
		fmt.Println("ReadFile ca.crt error ", err)
		return
	}
	// 解析证书
	if ok := certPool.AppendCertsFromPEM(ca); !ok {
		fmt.Println("certPool.AppendCertsFromPEM error ")
		return
	}
	cred := credentials.NewTLS(&tls.Config{
		Certificates: []tls.Certificate{pair},
		//ServerName:   "liuqh.icu",
		ServerName: "test.com",
		RootCAs:    certPool,
	})
	conn, err := grpc.Dial(":1234", grpc.WithTransportCredentials(cred))
	if err != nil {
		fmt.Println("dial error ", err)
		return
	}
	defer conn.Close()
	// 实例化客户端
	client := tslservice.NewTslServiceClient(conn)
	test, err := client.Test(context.TODO(), &wrapperspb.Int32Value{Value: 1})
	fmt.Println("res:", test)
	fmt.Println("err:", err)
}

```

### 5.3 发起请求

```bash
# 证书正常时
$ go run client.go
res: code:200 msg:"success"
err: <nil>

# 故意写错客户端中的 ServerName:test.com
$  go run client.go
res: <nil>
err: rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed: x509: certificate is valid for liuqh.icu, not test.com"
```
















