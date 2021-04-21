### 1.字符串查找

`strings`是Go内置的标准包,主要用于字符串查找、替换、比较等。常用方法如下:

#### 1.1 方法列表

| 方法名                                        | 描述                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| `Count(s, substr string)int`                 | 返回字符串s包含字符串substr的个数                            |
| `Contains(s, substr string)bool`             | 判断字符串s是否包含substr字符串                              |
| `ContainsAny(s, chars string)bool`           | 判断字符串s是否包含chars字符串中的任意一个字符               |
| `ContainsRune(s string, r rune)bool`         | 判断字符串s是否包含unicode的码值r                            |
| `LastIndex(s, substr string)int`             | 返回字符串s中字符串substr最后一次出现的位置                  |
| `Index(s, substr string)int`                 | 返回字符串s中字符串substr首次出现的位置                      |
| `IndexByte(s string, c byte)int`             | 返回字符串s中字符c首次出现的位置                             |
| `IndexRune(s string, r rune)int`             | 返回unicode的码值r在字符串s中首次出现的位置                  |
| `IndexAny(s, chars string)int`               | 返回字符串chars中的任意一个字符unicode码值,<br>在s中首次出现的位置 |
| `LastIndexAny(s, chars string)int`           | 返回字符串chars中的任意一个字符unicode码值,<br/>在s中最后一次出现的位置 |
| `LastIndexByte(s string, c byte)int`         | 返回字符串s中字符c最后一次出现的位置                         |
| `HasPrefix(s, prefix string)bool`            | 判断字符串s是否有前缀prefix                                  |
| `HasSuffix(s, suffix string)bool`            | 判断字符串s是否有后缀suffix                                  |
| `IndexFunc(s string, f func(r)bool)int`     | 返回字符串s中满足函数f(r)==true,<br/>字符首次出现的位置      |
| `LastIndexFunc(s string, f func(r)bool)int` | 返回字符串s中满足函数f(r)==true ,<br>字符最后一次出现的位置  |

#### 1.2 使用示例

```go
package main

import (
	"fmt"
	"strings"
	"unicode"
)

func main() {
	// 返回字符串s包含字符串substr的个数
	s := "hello word"
	fmt.Printf("字符串:%s,o出现数量: %d\n",s,strings.Count(s,"o"))
	// 判断字符串s是否包含substr字符串
	fmt.Printf("字符串:%s 是否包含%s ? %t \n",s,"word",strings.Contains(s,"word"))
	fmt.Printf("字符串:%s 是否包含%s ? %t \n",s,"go",strings.Contains(s,"go"))
	// 判断字符串s是否包含chars字符串中的任意一个字符
	fmt.Printf("字符串:%s 是否包含%s中的任意一个字符 ? %t \n",s,"go",strings.ContainsAny(s,"go"))
	fmt.Printf("字符串:%s 是否包含%s中的任意一个字符 ? %t \n",s,"gg",strings.ContainsAny(s,"gg"))
	// 判断字符串s是否包含unicode的码值r
	r := 'w'
	fmt.Printf("字符串:%s 是否包含unicode的码值%c? %t \n",s,r,strings.ContainsRune(s,r))
	fmt.Printf("字符串:%s 是否包含unicode的码值%d? %t \n",s,119,strings.ContainsRune(s,119))
	// 返回字符串s中字符串substr最后一次出现的位置
	fmt.Printf("在字符串%s中,字符串%s最后一次出现的位置? %d \n",s,"o",strings.LastIndex(s,"o"))
	// 返回字符串s中字符串substr首次出现的位置
	fmt.Printf("在字符串%s中,字符串%s首次出现的位置? %d \n",s,"o",strings.Index(s,"o"))
	// 返回字符串s中字符c首次出现的位置
	var b byte = 'l'
	fmt.Printf("在字符串%s中,字符%c首次出现的位置? %d \n",s,b,strings.IndexByte(s,b))
	// 返回字符串s中字符c最后一次出现的位置
	fmt.Printf("在字符串%s中,字符%c最后一次出现的位置? %d \n",s,b,strings.LastIndexByte(s,b))

	// 返回unicode的码值r在字符串s中首次出现的位置
	fmt.Printf("在字符串%s中,unicode的码值%d(%c)首次出现的位置? %d \n",s,104,104,strings.IndexRune(s,104))
	// 返回字符串chars中的任意一个字符unicode码值,在s中首次出现的位置
	s3 := "rd"
	fmt.Printf("返回字符串%s中的任意一个字符unicode码值(%s)首次出现的位置? %d \n",s,s3,strings.LastIndexAny(s,s3))

	// 判断字符串s是否有前缀prefix
	a := "VIP001"
	fmt.Printf("字符串:%s 是否有前缀%s ? %t \n",a,"vip",strings.HasPrefix(a,"vip"))
	fmt.Printf("字符串:%s 是否有前缀%s ? %t \n",a,"VIP",strings.HasPrefix(a,"VIP"))

	// 判断字符串s是否有后缀suffix
	sn := "K011_Mn"
	fmt.Printf("字符串:%s 是否有后缀%s ? %t \n",sn,"MN",strings.HasSuffix(sn,"MN"))
	fmt.Printf("字符串:%s 是否有后缀%s ? %t \n",sn,"Mn",strings.HasSuffix(sn,"Mn"))
	// 返回字符串s中满足函数f(r)==true,字符首次出现的位置 (判断第一个汉字的位置)
	f := func(c rune) bool {
		return unicode.Is(unicode.Han,c)
	}
	s4 := "go!中国人"
	fmt.Printf("字符串:%s 首次出现汉字的位置%d \n",s4,strings.IndexFunc(s4,f))
	fmt.Printf("字符串:%s 最后一次出现汉字的位置%d \n",s4,strings.LastIndexFunc(s4,f))
}
/** 输出
  字符串:hello word,o出现数量: 2
  字符串:hello word 是否包含word ? true 
  字符串:hello word 是否包含go ? false 
  字符串:hello word 是否包含go中的任意一个字符 ? true 
  字符串:hello word 是否包含gg中的任意一个字符 ? false 
  字符串:hello word 是否包含unicode的码值w? true 
  字符串:hello word 是否包含unicode的码值119? true 
  在字符串hello word中,字符串o最后一次出现的位置? 7 
  在字符串hello word中,字符串o首次出现的位置? 4 
  在字符串hello word中,字符l首次出现的位置? 2 
  在字符串hello word中,字符l最后一次出现的位置? 3 
  在字符串hello word中,unicode的码值104(h)首次出现的位置? 0 
  返回字符串hello word中的任意一个字符unicode码值(rd)首次出现的位置? 9 
  字符串:VIP001 是否有前缀vip ? false 
  字符串:VIP001 是否有前缀VIP ? true 
  字符串:K011_Mn 是否有后缀MN ? false 
  字符串:K011_Mn 是否有后缀Mn ? true 
  字符串:go!中国人 首次出现汉字的位置3 
  字符串:go!中国人 最后一次出现汉字的位置9 
*/
```

### 2. 字符串分割

#### 2.1 方法列表

| 方法名                                          | 描述                                                         |
| :---------------------------------------------- | ------------------------------------------------------------ |
| `Fields(s string)[]string`                     | 将字符串s以空白字符分割，返回切片                            |
| `FieldsFunc(s string, f func(r) bool)[]string` | 将字符串s以满足`f(r)==true`的字符分割，<br/>分割后返回切片   |
| `Split(s,sep string)[]string`                 | 将字符串s以sep作为分割符进行分割，<br/><font color=red>分割后字符最后去掉sep</font>,返回切片 |
| `SplitAfter(s,sep string)[]string`            | 将字符串s以sep作为分割符进行分割，<br/><font color=green>分割后字符最后加上sep</font>,返回切片 |
| `SplitAfterN(s,sep string, n int)[]string`    | 将字符串s以sep作为分割符进行分割，<br/><font color=green>分割后字符最后加上sep</font>,n决定分割成切片长度 |
| `SplitN(s,sep string, n int)[]string`         | 将字符串s以sep作为分割符进行分割，<br/><font color=red>分割后字符最后去掉sep</font>,n决定分割成切片长度 |

#### 2.2 使用示例

```go
package main
import (
	"fmt"
	"strings"
	"unicode"
)
func main() {
	s := "Go! Go! 中国人!"
	// 将字符串s以空白字符分割，返回切片
	slice := strings.Fields(s)
	fmt.Printf("将字符串:【%s】以空白字符分割，返回切片:%v \n",s,slice)
	// 将字符串s以满足f(r)==true的字符分割，分割后返回切片。
	// 以特殊符号分割
    f := func(r rune) bool{
    	// 不是字母，也不是数字
    	return !unicode.IsLetter(r) && !unicode.IsNumber(r)
	}
	ss := "张三@19*BeiJing&高中生|男(打球"
	slice2 := strings.FieldsFunc(ss,f)
	fmt.Printf("将字符串:【%s】 以满足f(r)==true【不是数字和字母都是分隔符】的字符分割，返回切片:%v \n",ss,slice2)

	// 将字符串s以sep作为分割符进行分割，分割后字符最后去掉sep
	s2 := "@123@张@AB@001"
	sep1 := "@"
	slic1 := strings.Split(s2,sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，分割后最后去掉:%s 返回切片: %v 切片长度: %d \n",s2,sep1,sep1,slic1,len(slic1))

	// 将字符串s以sep作为分割符进行分割，分割后字符最后加上sep,返回切片
	slic2 := strings.SplitAfter(s2,sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，分割后最后加上:%s 返回切片: %v 切片长度: %d \n",s2,sep1,sep1,slic2,len(slic2))
	// 将字符串s以sep作为分割符进行分割，分割后字符最后加上sep,n决定分割成切片长度
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后加上%s \n",s2,sep1,0,strings.SplitAfterN(s2,sep1,0),sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后加上%s \n",s2,sep1,1,strings.SplitAfterN(s2,sep1,1),sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后加上%s \n",s2,sep1,5,strings.SplitAfterN(s2,sep1,5),sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后加上%s \n",s2,sep1,6,strings.SplitAfterN(s2,sep1,6),sep1)
	// 将字符串s以sep作为分割符进行分割，分割后字符最后去掉sep,n决定分割成切片长度
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后去掉%s \n",s2,sep1,1,strings.SplitN(s2,sep1,1),sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后去掉%s \n",s2,sep1,3,strings.SplitN(s2,sep1,3),sep1)
	fmt.Printf("将字符串:【%s】以%s进行分割，指定分割切片长度%d: %v 分割后去掉%s \n",s2,sep1,5,strings.SplitN(s2,sep1,5),sep1)
}
/** 输出
将字符串:【Go! Go! 中国人!】以空白字符分割，返回切片:[Go! Go! 中国人!] 
将字符串:【张三@19*BeiJing&高中生|男(打球】 以满足f(r)==true【不是数字和字母都是分隔符】的字符分割，返回切片:[张三 19 BeiJing 高中生 男 打球] 
将字符串:【@123@张@AB@001】以@进行分割，分割后最后去掉:@ 返回切片: [ 123 张 AB 001] 切片长度: 5 
将字符串:【@123@张@AB@001】以@进行分割，分割后最后加上:@ 返回切片: [@ 123@ 张@ AB@ 001] 切片长度: 5 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度0: [] 分割后加上@ 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度1: [@123@张@AB@001] 分割后加上@ 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度5: [@ 123@ 张@ AB@ 001] 分割后加上@ 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度6: [@ 123@ 张@ AB@ 001] 分割后加上@ 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度1: [@123@张@AB@001] 分割后去掉@ 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度3: [ 123 张@AB@001] 分割后去掉@ 
将字符串:【@123@张@AB@001】以@进行分割，指定分割切片长度5: [ 123 张 AB 001] 分割后去掉@ 
*/
```

### 3.大小写切换

#### 3.1 方法列表

| 方法名                                                  | 描述                                                  |
| ------------------------------------------------------- | ----------------------------------------------------- |
| `Title(s string)string`                                 | 将字符串s每个单词首字母大写                           |
| `ToLower(s string)string`                               | 将字符串s转换成小写返回                               |
| `ToLowerSpecial(c unicode.SpecialCase,s string)string`  | 将字符串s中所有字符串按c指定的<br/>映射转换成小写返回 |
| `ToTitle(s string)string`                               | 将字符串s转换成大写返回                               |
| `ToTitleSpecial(c unicode.SpecialCase,s string) string` | 将字符串s中所有的字符按c指定的<br/>映射转换成大写返回 |
| `ToUpper(s string)string`                               | 将字符串s转换成大写返回                               |
| `ToUpperSpecial(c unicode.SpecialCase,s string) string` | 将字符串s中所有的字符按c指定的<br/>映射转换成大写返回 |

#### 3.2 使用示例

```go
package main
import (
	"fmt"
	"strings"
)
func main() {

	str := "hello word"
	str1 := "HELLO WORD"
	// Title(s string) string: 每个单词首字母大写
	fmt.Printf("Title->将字符串%s 每个单词首字母大写: %s\n",str,strings.Title(str))
	// ToLower(s string) string : 将字符串s转换成小写返回
	fmt.Printf("ToLower->将字符串%s 转换成小写返回: %s\n",str1,strings.ToLower(str1))
	// ToTitle(s string)string: 将字符串s转换成大写返回
	fmt.Printf("ToTitle->将字符串%s 转换成大写返回: %s\n",str,strings.ToTitle(str))
	// ToUpper(s string)string: 将字符串s转换成大写返回
	fmt.Printf("ToUpper->将字符串%s 转换成大写返回: %s\n",str,strings.ToUpper(str))
}
/**输出
Title->将字符串hello word 每个单词首字母大写: Hello Word
ToLower->将字符串HELLO WORD 转换成小写返回: hello word
ToTitle->将字符串hello word 转换成大写返回: HELLO WORD
ToUpper->将字符串hello word 转换成大写返回: HELLO WORD
*/
```

#### 3.3 ToTitle和ToUpper的区别

大部分情况下， `ToUpper` 与 `ToTitle` 返回值相同，但在处理某些`unicode`编码字符则不同,示例如下:

```go
package main
import (
	"fmt"
	"strings"
)
func main() {
	strr := "ǳ ǵǵǳǳǳ hello world！"
	fmt.Printf("ToTitle->将字符串%s 转换成大写返回: %s\n",strr,strings.ToTitle(strr))
	// ToUpper(s string)string: 将字符串s转换成大写返回
	fmt.Printf("ToUpper->将字符串%s 转换成大写返回: %s\n",strr,strings.ToUpper(strr))
}
/** 输出
ToTitle->将字符串ǳ ǵǵǳǳǳ hello world！ 转换成大写返回: ǲ ǴǴǲǲǲ HELLO WORLD！
ToUpper->将字符串ǳ ǵǵǳǳǳ hello world！ 转换成大写返回: Ǳ ǴǴǱǱǱ HELLO WORLD！
*/
```

### 4.字符串删除

#### 4.1 方法列表

| 方法名                                           | 描述                                            |
| ------------------------------------------------ | ----------------------------------------------- |
| `Trim(s,cutset string)string`                    | 将字符串`s`首尾包含在`cutset`中的任一字符去掉   |
| `TrimFunc(s string,f func(r)bool)string`         | 将字符串`s`首尾满足函数`f(r)==true`的字符串去掉 |
| `TrimLeft(s,cutset string)string`                | 将字符串`s`左边包含在`cutset`中的任一字符去掉   |
| `TrimLeftFunc(s string,f func(r)bool) string`    | 将字符串`s`左边满足函数`f(r)==true`的字符串去掉 |
| `TrimPrefix(s,prefix string)string`              | 将字符串`s`中前缀字符串`prefix`去掉             |
| `TrimRight(s,cutset string) string`              | 将字符串`s`右边包含在`cutset`中的任一字符去掉   |
| `TrimRightFunc(s string, f func(r) bool) string` | 将字符串`s`右边满足函数`f(r)==true`的字符串去掉 |
| `TrimSpace(s string) string`                     | 将字符串首尾空白去掉                            |
| `TrimSuffix(s, suffix string) string`            | 将字符串`s`中后缀字符串`suffix`去掉             |

#### 4.2 使用示例

```go
package main
import (
	"fmt"
	"strings"
)
func main() {
	// 将字符串首尾包含在cutset中的任一字符去掉
	str := "@*test@-@124@!*"
	cutset := "*#@!"
	fmt.Printf("将字符串【%s】首尾包含在【%s】中的任一字符去掉,返回:【%s】\n",str,cutset,strings.Trim(str,cutset))
	// 将字符串首尾满足函数`f(r)==true`的字符串去掉
	f := func(r rune) bool {
		if strings.Contains("*#@!",string(r)) {
			return true
		}
		return false
	}
	fmt.Printf("将字符串【%s】首尾满足函数f的字符去掉,返回:【%s】\n",str,strings.TrimFunc(str,f))
	// 将字符串左边包含在cutset中的任一字符去掉
	fmt.Printf("将字符串【%s】左边包含在【%s】中的任一字符去掉,返回:【%s】\n",str,cutset,strings.TrimLeft(str,cutset))

	// 将字符串左边满足函数`f(r)==true`的字符串去掉
	fmt.Printf("将字符串【%s】左边满足函数f的字符去掉,返回:【%s】\n",str,strings.TrimLeftFunc(str,f))
	
	// 将字符串右边包含在cutset中的任一字符去掉
	fmt.Printf("将字符串【%s】右边包含在【%s】中的任一字符去掉,返回:【%s】\n",str,cutset,strings.TrimRight(str,cutset))
	fmt.Printf("将字符串【%s】右边满足函数f的字符去掉,返回:【%s】\n",str,strings.TrimRightFunc(str,f))

	// 将字符串中前缀字符串prefix去掉
	str1 := "VIP00001_U"
	fmt.Printf("将字符串【%s】前缀【%s】去掉,返回:【%s】\n",str1,"VIP",strings.TrimPrefix(str1,"VIP"))
	fmt.Printf("将字符串【%s】前缀【%s】去掉,返回:【%s】\n",str1,"vip",strings.TrimPrefix(str1,"vip"))

  // 将字符串中后缀字符串suffix去掉
	fmt.Printf("将字符串【%s】后缀【%s】去掉,返回:【%s】\n",str1,"U",strings.TrimSuffix(str1,"U"))
	fmt.Printf("将字符串【%s】后缀【%s】去掉,返回:【%s】\n",str1,"u",strings.TrimSuffix(str1,"u"))

	// 将字符串首尾空白去掉
	str2 := "  hello  word !  "
	fmt.Printf("将字符串【%s】首尾空白去掉,返回:【%s】\n",str2,strings.TrimSpace(str2))
}
/**输出
将字符串【@*test@-@124@!*】首尾包含在【*#@!】中的任一字符去掉,返回:【test@-@124】
将字符串【@*test@-@124@!*】首尾满足函数f的字符去掉,返回:【test@-@124】
将字符串【@*test@-@124@!*】左边包含在【*#@!】中的任一字符去掉,返回:【test@-@124@!*】
将字符串【@*test@-@124@!*】左边满足函数f的字符去掉,返回:【test@-@124@!*】
将字符串【@*test@-@124@!*】右边包含在【*#@!】中的任一字符去掉,返回:【@*test@-@124】
将字符串【@*test@-@124@!*】右边满足函数f的字符去掉,返回:【@*test@-@124】
将字符串【VIP00001_U】前缀【VIP】去掉,返回:【00001_U】
将字符串【VIP00001_U】前缀【vip】去掉,返回:【VIP00001_U】
将字符串【VIP00001_U】后缀【U】去掉,返回:【VIP00001_】
将字符串【VIP00001_U】后缀【u】去掉,返回:【VIP00001_U】
将字符串【  hello  word !  】首尾空白去掉,返回:【hello  word !】
*/
```

### 5.拼接和重复

#### 5.1 方法列表

| 方法名                                     | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| `Join(elems []string, sep string) string`  | 将字符串切片elems，使用sep进行拼接                           |
| `Repeat(s string, count int) string`       | 将字符串s,重复count次                                        |

#### 5.3 使用示例

```go
package main
import (
	"fmt"
	"strings"
)
func main() {
	// 字符串拼接
	fmt.Printf("字符串拼接:Join-> %s\n",strings.Join([]string{"a","b","c"},"|"))
	// 字符串重复
	fmt.Printf("字符串重复:Repeat-> %s\n",strings.Repeat("Go!",10))
}
/**输出
字符串拼接:Join-> a|b|c
字符串重复:Repeat-> Go!Go!Go!Go!Go!Go!Go!Go!Go!Go!
*/
```

### 6.字符串替换

#### 6.1 方法列表

| 方法名                                     | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| `Replace(s, old, new string, n int)string` | 将字符串s前n个不重叠old子串都替换为new的新字符串<br/>如果n<0会替换所有old子串。 |
| `ReplaceAll(s, old, new string) string`    | 将字符串s中的old子串全部替换为new的新字符串                  |

#### 6.2 使用示例

```go
package main
import (
	"fmt"
	"strings"
)
func main() {
	// 字符串替换,如果n<0会替换所有old子串。
	s := "a,b,c,d,e,f"
	old := ","
	newStr := "."
    fmt.Printf("将字符串【%s】中的前%d个【%s】替换为【%s】结果是【%s】\n",s,2,old,newStr,strings.Replace(s,old,newStr,2))
    fmt.Printf("将字符串【%s】中的前%d个【%s】替换为【%s】结果是【%s】\n",s,7,old,newStr,strings.Replace(s,old,newStr,7))
    fmt.Printf("将字符串【%s】中的前%d个【%s】替换为【%s】结果是【%s】\n",s,-1,old,newStr,strings.Replace(s,old,newStr,-1))
	// 字符串全部替换
	fmt.Printf("将字符串【%s】中的【%s】全部替换为【%s】结果是【%s】\n",s,old,newStr,strings.ReplaceAll(s,old,newStr))
}
/**输出
将字符串【a,b,c,d,e,f】中的前2个【,】替换为【.】结果是【a.b.c,d,e,f】
将字符串【a,b,c,d,e,f】中的前7个【,】替换为【.】结果是【a.b.c.d.e.f】
将字符串【a,b,c,d,e,f】中的前-1个【,】替换为【.】结果是【a.b.c.d.e.f】
将字符串【a,b,c,d,e,f】中的【,】全部替换为【.】结果是【a.b.c.d.e.f】
*/
```

### 7.字符串比较

#### 7.1 方法列表

| 方法名                        | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `Compare(a, b string) int`    | 按字典顺序比较a和b字符串的大小<br>如果 a > b，返回一个大于 0 的数<br/>如果 a == b，返回 0<br>果 a > b，返回一个小于 0 的数 |
| `EqualFold(s, t string) bool` | 判断s和t两个UTF-8字符串是否相等，<font color=red>忽略大小写</font> |

#### 7.1 使用示例

```go
package main
import (
	"fmt"
	"strings"
)
func main() {
	// 字符串比较大小
	s := "a"
	s1 := "c"
	s2 := "c"
	fmt.Printf("%s > %s 返回 : %d \n",s,s1,strings.Compare(s,s1))
	fmt.Printf("%s > %s 返回 : %d \n",s1,s,strings.Compare(s1,s))
	fmt.Printf("%s > %s 返回 : %d \n",s1,s2,strings.Compare(s1,s2))
	// 字符串比较一致性
	a := "go"
	b := "Go"
	c := "go"
	fmt.Printf("%s和%s是否相等(忽略大小写)？%t \n",a,b,strings.EqualFold(a,b))
	fmt.Printf("%s和%s是否相等(忽略大小写)？%t \n",a,c,strings.EqualFold(a,c))
	fmt.Printf("%s和%s是否相等(不忽略大小写)？%t \n",a,b,a == b)
	fmt.Printf("%s和%s是否相等(不忽略大小写)？%t \n",a,c,a == c)
}
/**输出
a > c 返回 : -1 
c > a 返回 : 1 
c > c 返回 : 0 
go和Go是否相等(忽略大小写)？true 
go和go是否相等(忽略大小写)？true 
go和Go是否相等(不忽略大小写)？false 
go和go是否相等(不忽略大小写)？true 
*/
```

