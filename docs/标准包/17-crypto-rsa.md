### 1. 维基百科释义

**RSA加密算法**是一种[非对称加密算法](https://zh.wikipedia.org/wiki/非对称加密演算法)，在[公开密钥加密](https://zh.wikipedia.org/wiki/公开密钥加密)和[电子商业](https://zh.wikipedia.org/wiki/电子商业)中被广泛使用。RSA是由[罗纳德·李维斯特](https://zh.wikipedia.org/wiki/罗纳德·李维斯特)（Ron Rivest）、[阿迪·萨莫尔](https://zh.wikipedia.org/wiki/阿迪·萨莫尔)（Adi Shamir）和[伦纳德·阿德曼](https://zh.wikipedia.org/wiki/伦纳德·阿德曼)（Leonard Adleman）在1977年一起提出的。当时他们三人都在[麻省理工学院](https://zh.wikipedia.org/wiki/麻省理工学院)工作。RSA 就是他们三人姓氏开头字母拼在一起组成的。

### 2.生成密钥

`RSA`私钥存在`PKCS1`和`PKCS8`两种格式，通过`openssl`生成的私钥格式为`PKCS1`,公钥格式为`PKCS8`。

#### 2.1 使用`openssl`命令

##### 1.生成私钥(`PKCS1`格式)

```bash
# 生成私钥
➜ openssl genrsa -out private_ssl.pem 1024
Generating RSA private key, 1024 bit long modulus
.......++++++
.............................................++++++
e is 65537 (0x10001)
# 查看私钥
➜ cat private_ssl.pem
-----BEGIN RSA PRIVATE KEY-----
MIICXgIBAAKBgQDSgkiwisLr7yTuKnn6jANvRXRfnA9PINojpPiegMkv/mnScEvM
czP8OZjpFrsjSKFnlc6OX04O+4G3GuBMjc75wQw79auM4WMJtSh3PAWiBGu6Woto
AqZFbWRnIzk2Wjw1xSJxpOUa0ed1plU/jut1dKgDTi4q+BDqGizI2JeJdQIDAQAB
AoGAFij24/bHjDSxi4zXKGPi3KzQElyIVAkeTZBJR85A35eFpkyB/jTGbS/XA/qL
mqxDqXbgtqYbvoIFZrQilox7FCi5AzxhQDB3wD67y5OjuSWhTKypq5UOj/6y9zkC
Gbzi4zIBlOZIskfj81a+WBg7vi6FCaCg105nDJOCDs0IYiECQQDpZNBoolQGgsmR
M/x699WXhG1xYV9bTDhBKpsW/O0mRjiK/6pCbDjRWGxeB+BmFmRofYwWRj2stzkg
We1if8jNAkEA5uYFWBqHpKpFUULM7fqoyixrTXz+3h22vcKc4s/xhSCfM8yCS/ad
7S4tzpEGAenLPWYERhm6Cm/0yVavtYBjSQJBAN0MBG71P0ujVzDU4c29KGiGnfkC
VgPsHmNp7NVK23iijS7okeKzCOUNMCWmpBtMHfDw51q5T/Ri3BIN2cyuZgkCQQCV
Hz2YVxn/mRBHmRLtJ5PXbrSmSPH51crt50CXo6DiT91CAPStxsrcIZTn8fWlSq8+
KiLce0UR9JhtaBT27cIxAkEArozPsVt273/UkAru4LgBURK5ZEaHful/oK0xRDdg
fq+Ij4rs24CVPlNXfycZEDrKYMS9ScZrraf8JsTSEBx++w==
-----END RSA PRIVATE KEY-----
```

##### 2.生成公钥(`PKCS8格式`)

```bash
# 生成公钥
➜ openssl rsa -in private_ssl.pem -pubout -out public_ssl.pem
writing RSA key
# 查看公钥
➜ cat public_ssl.pem
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDSgkiwisLr7yTuKnn6jANvRXRf
nA9PINojpPiegMkv/mnScEvMczP8OZjpFrsjSKFnlc6OX04O+4G3GuBMjc75wQw7
9auM4WMJtSh3PAWiBGu6WotoAqZFbWRnIzk2Wjw1xSJxpOUa0ed1plU/jut1dKgD
Ti4q+BDqGizI2JeJdQIDAQAB
-----END PUBLIC KEY-----
```

##### 3. 私钥PKCS1转PKCS8

```bash
➜ openssl pkcs8 -topk8 -inform pem -in private_ssl.pem -outform PEM -nocrypt
-----BEGIN PRIVATE KEY-----
MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBANKCSLCKwuvvJO4q
efqMA29FdF+cD08g2iOk+J6AyS/+adJwS8xzM/w5mOkWuyNIoWeVzo5fTg77gbca
4EyNzvnBDDv1q4zhYwm1KHc8BaIEa7pai2gCpkVtZGcjOTZaPDXFInGk5RrR53Wm
VT+O63V0qANOLir4EOoaLMjYl4l1AgMBAAECgYAWKPbj9seMNLGLjNcoY+LcrNAS
XIhUCR5NkElHzkDfl4WmTIH+NMZtL9cD+ouarEOpduC2phu+ggVmtCKWjHsUKLkD
PGFAMHfAPrvLk6O5JaFMrKmrlQ6P/rL3OQIZvOLjMgGU5kiyR+PzVr5YGDu+LoUJ
oKDXTmcMk4IOzQhiIQJBAOlk0GiiVAaCyZEz/Hr31ZeEbXFhX1tMOEEqmxb87SZG
OIr/qkJsONFYbF4H4GYWZGh9jBZGPay3OSBZ7WJ/yM0CQQDm5gVYGoekqkVRQszt
+qjKLGtNfP7eHba9wpziz/GFIJ8zzIJL9p3tLi3OkQYB6cs9ZgRGGboKb/TJVq+1
gGNJAkEA3QwEbvU/S6NXMNThzb0oaIad+QJWA+weY2ns1UrbeKKNLuiR4rMI5Q0w
JaakG0wd8PDnWrlP9GLcEg3ZzK5mCQJBAJUfPZhXGf+ZEEeZEu0nk9dutKZI8fnV
yu3nQJejoOJP3UIA9K3GytwhlOfx9aVKrz4qItx7RRH0mG1oFPbtwjECQQCujM+x
W3bvf9SQCu7guAFRErlkRod+6X+grTFEN2B+r4iPiuzbgJU+U1d/JxkQOspgxL1J
xmutp/wmxNIQHH77
-----END PRIVATE KEY-----
```

### 3.读取密钥

#### 3.1 代码

```go
// 读取PKCS1格式私钥
func ReadRSAPKCS1PrivateKey(path string) (*rsa.PrivateKey, error) {
	// 读取文件
	context, err := ioutil.ReadFile(path)
	if err != nil {
		return nil, err
	}
	// pem解码
	pemBlock, _ := pem.Decode(context)
	// x509解码
	privateKey, err := x509.ParsePKCS1PrivateKey(pemBlock.Bytes)
	return privateKey, err
}
// 读取公钥(包含PKCS1和PKCS8)
func ReadRSAPublicKey(path string) (*rsa.PublicKey, error) {
	var  err error
	// 读取文件
	readFile, err := ioutil.ReadFile(path)
	if err != nil {
		return nil, err
	}
	// 使用pem解码
	pemBlock, _ := pem.Decode(readFile)
	var pkixPublicKey interface{}
	if pemBlock.Type == "RSA PUBLIC KEY" {
		// -----BEGIN RSA PUBLIC KEY-----
		pkixPublicKey, err = x509.ParsePKCS1PublicKey(pemBlock.Bytes)
	} else if pemBlock.Type == "PUBLIC KEY" {
		// -----BEGIN PUBLIC KEY-----
		pkixPublicKey, err = x509.ParsePKIXPublicKey(pemBlock.Bytes)
	}
	if err != nil {
		return nil,err
	}
	publicKey := pkixPublicKey.(*rsa.PublicKey)
	return publicKey, nil
}
```

#### 3.2 测试

```go
// 读取密钥
func TestReadKey(t *testing.T) {
	// pkcs1格式-私钥
	privatePKCS1KeyPath := "../../tmp/private_ssl.pem"
	privatePKCS1Key, err := crypto.ReadRSAPKCS1PrivateKey(privatePKCS1KeyPath)
	if err != nil {
		t.Error(err)
	}
	fmt.Printf("PKCS1私钥: %#v\n",privatePKCS1Key)
	// pkcs8格式-公钥
	publicPKCS8KeyPath := "../../tmp/public_ssl.pem"
	publicPKCS8Key, err := crypto.ReadRSAPublicKey(publicPKCS8KeyPath)
	if err != nil {
		t.Error(err)
	}
	fmt.Printf("PKCS8公钥: %#v\n",publicPKCS8Key)
}
```

### 4.加密

#### 4.1 代码

```go
// 加密(使用公钥加密)
func RSAEncrypt(data, publicKeyPath string) (string, error) {
	// 获取公钥
  // ReadRSAPublicKey代码在 【3.读取密钥】
	rsaPublicKey, err := ReadRSAPublicKey(publicKeyPath)
	if err != nil {
		return "", err
	}
	// 加密
	encryptPKCS1v15, err := rsa.EncryptPKCS1v15(rand.Reader, rsaPublicKey, []byte(data))
	if err != nil {
		return "",err
	}
	// 把加密结果转成Base64
	encryptString := base64.StdEncoding.EncodeToString(encryptPKCS1v15)
	return encryptString, err
}
```

#### 4.2 测试

```go
// 加密测试
func TestRsaEncrypt(t *testing.T) {
	publicKeyPath := "../../tmp/public_ssl.pem"
	data := "123456"
	encrypt, err := crypto.RSAEncrypt(data, publicKeyPath)
	if err != nil {
		t.Error(err)
	}
	fmt.Printf("加密结果:%v \n",encrypt)
}
/** 输出
=== RUN   TestRsaEncrypt
加密结果:SRYyBXd4p+wUeTZ5478g+hW2P3OvqhYMyPwW/j91SappgxMWC/O3vCG2aVTcAHknUkK2oEs6e28deKuOvOkjSWl/jnXFDCkXklgbgnXJtfu2FjP9jXhG2b6/Eo3okxLvLXZtkaRAgZKNbbKkeiNASUO4IidkoNrnI4aOuuuVIOY= 
--- PASS: TestRsaEncrypt (0.00s)
PASS
*/
```

### 5.解密

#### 5.1 代码

```go
// 解密(使用私钥解密)
func RSADecrypt(base64data,privateKeyPath string) (string,error) {
	// data反解base64
	decodeString, err := base64.StdEncoding.DecodeString(base64data)
	if err != nil {
		return "", err
	}
	// 读取密钥
	rsaPrivateKey, err := ReadRSAPKCS1PrivateKey(privateKeyPath)
	if err != nil {
		return "", err
	}
	// 解密
	decryptPKCS1v15, err := rsa.DecryptPKCS1v15(rand.Reader, rsaPrivateKey, decodeString)
	return string(decryptPKCS1v15),err
}
```

#### 5.2 测试

```go
// 解密测试
func TestRsaDecrypt(t *testing.T) {
	privateKeyPath := "../../tmp/private_ssl.pem"
	data := "pUYa4set6XkBshfio5g2hzPx1tA67sxEvJBpJiuK3McJ9cPJAXzuRkWIy4s6cDQOhrPUaNXhr3M3WLHH19/eaqcNZz1yOFZwgGKmkWtdmygtLB/wrDant9uRfXrvzlV9iMq+cUlqsrwuCa0wcGEBNHRhIJOQSTs+SxaRTeoRCbU="
	encrypt, err := crypto.RSADecrypt(data, privateKeyPath)
	if err != nil {
		t.Error(err)
	}
	fmt.Printf("解密结果:%v \n",encrypt)
}
/** 输出
=== RUN   TestRsaDecrypt
解密结果:123456 
--- PASS: TestRsaDecrypt (0.00s)
PASS
*/
```

### 6.数据签名

#### 6.1 加签

**a. 代码**

```go
// 对数据进行数字签名
func GetRSASign(data, privateKeyPath string) (string, error) {
	// 读取私钥
	privateKey, err := ReadRSAPKCS1PrivateKey(privateKeyPath)
	if err != nil {
		return "", err
	}
	// 计算Sha1散列值
	hash := sha256.New()
	hash.Write([]byte(data))
	sum := hash.Sum(nil)
	// 从1.5版本规定，使用RSASSA-PKCS1-V1_5-SIGN 方案计算签名
	signPKCS1v15, err := rsa.SignPKCS1v15(rand.Reader, privateKey, crypto.SHA256, sum)
	// 结果转成base64
	toString := base64.StdEncoding.EncodeToString(signPKCS1v15)
	return toString, err
}
```

**b. 测试**

```go
// 数据加签
func TestAddSign(t *testing.T) {
	privateKeyPath := "../../tmp/private_ssl.pem"
	data := "123456"
	sign, err := crypto.GetRSASign(data,privateKeyPath)
	if err != nil {
		t.Error(err)
	}
	fmt.Printf("数据签名: %v \n",sign)
}
/** 输出
=== RUN   TestAddSign
数据签名: QnGqGbIqoHjJG1l+JiaOKWBdX+h00lnKCoO2rTYKIro9hoaDj7nqmu+Mxsuo+2jumicvCNBZNOpMzYryjZf0x7Q4ycLBtqtCWuFRasiInUO7Avy19LRTjdMf2xw9968vilB/xEAQ53JXIDUVvCsMxTfpHI9oRiWEGXWNkhfkjkQ= 
--- PASS: TestAddSign (0.00s)
PASS
*/
```

#### 6.2 验签

**a. 代码**

```go
// 验证签名
func VerifyRsaSign(data, publicKeyPath, base64Sign string) (bool, error) {
	// 反解base64
	sign,err := base64.StdEncoding.DecodeString(base64Sign)
	if err != nil {
		return false, err
	}
	// 获取公钥
	publicKey, err := ReadRSAPublicKey(publicKeyPath)
	if err != nil {
		return false, err
	}
	// 计算Sha1散列值
	hash := sha256.New()
	hash.Write([]byte(data))
	bytes := hash.Sum(nil)
	err = rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, bytes, sign)
	return err == nil, err
}
```

**b. 测试**

```go
// 数据签名验证
func TestVaSign(t *testing.T) {
	publicKeyPath := "../../tmp/public_ssl.pem"
	data := "123456"
	sign := "QnGqGbIqoHjJG1l+JiaOKWBdX+h00lnKCoO2rTYKIro9hoaDj7nqmu+Mxsuo+2jumicvCNBZNOpMzYryjZf0x7Q4ycLBtqtCWuFRasiInUO7Avy19LRTjdMf2xw9968vilB/xEAQ53JXIDUVvCsMxTfpHI9oRiWEGXWNkhfkjkQ="
	verifyRsaSign,err := crypto.VerifyRsaSign(data, publicKeyPath, sign)
	if err != nil {
		fmt.Printf("验签失败: %v \n",err)
	}
	fmt.Printf("验签结果: %v \n",verifyRsaSign)
}
/** 输出
=== RUN   TestVaSign
验签结果: true 
--- PASS: TestVaSign (0.00s)
PASS
*/
```



