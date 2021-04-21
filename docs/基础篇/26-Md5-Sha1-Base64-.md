### 1.生成Md5

```go
package main
import (
	"crypto/md5"
	"fmt"
	"io"
)
func main() {
	str := "Hello Word"
	// 方法一
	sum := md5.Sum([]byte(str))
	// 转成16进制
	md51 := fmt.Sprintf("%x", sum)
	fmt.Println("方法一: " + md51)

	// 方法二
	hash := md5.New()
	// 将str写入到hash
	_, err := io.WriteString(hash, str)
	if err != nil {
		fmt.Println(err.Error())
	}
	// 转成16进制
	md52 := fmt.Sprintf("%x",hash.Sum(nil))
	fmt.Println("方法二: " + md52)
}

/** 输出
方法一: ed0a96e83ab7b0910fcbcc131b2e6b82
方法二: ed0a96e83ab7b0910fcbcc131b2e6b82
*/
```

### 2.生成Sha1

```go
package main
import (
	"crypto/sha1"
	"fmt"
	"io"
)
func main() {
	str := "1234"
	// 方法一
	sum := sha1.Sum([]byte(str))
	// 转成16进制
	shaA := fmt.Sprintf("%x", sum)
	fmt.Println("方法一: " + shaA)

	// 方法二
	hash := sha1.New()
	_, err := io.WriteString(hash, str)
	if err != nil {
		fmt.Println(err.Error())
	}
	// 转成16进制
	shaB := fmt.Sprintf("%x",hash.Sum(nil))
	fmt.Println("方法二: " + shaB)
}
/** 输出
方法一: 7110eda4d09e062aa5e4a390b0a572ac0d2c0220
方法二: 7110eda4d09e062aa5e4a390b0a572ac0d2c0220
*/
```

### 3.Base64编码和解码

```go
package main
import (
	"encoding/base64"
	"fmt"
)
func main() {
	str := "1234"
	// 方法一
	// Base64编码
	base64EncodeStr := base64.StdEncoding.EncodeToString([]byte(str))
	fmt.Printf("方法一,Base64编码: %v \n",base64EncodeStr)
	// Base64解码
	decodeString, err := base64.StdEncoding.DecodeString(base64EncodeStr)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Printf("方法一,Base64解码: %s \n",decodeString)

	// 方法二:使用兼容URL的base64编码和解码
	toString := base64.URLEncoding.EncodeToString([]byte(str))
	fmt.Printf("方法二,Base64编码: %v \n",toString)
	bytes, _ := base64.URLEncoding.DecodeString(toString)
	fmt.Printf("方法二,Base64解码: %s \n",bytes)
}

/**输出
方法一,Base64编码: MTIzNA== 
方法一,Base64解码: 1234 
方法二,Base64编码: MTIzNA== 
方法二,Base64解码: 1234 
*/
```

