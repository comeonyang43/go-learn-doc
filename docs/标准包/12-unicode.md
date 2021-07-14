### 1.介绍

`unicode`包提供了诸多处理`rune`字符相关功能的函数（比如区分字母和数字，或者是字母的大写和小写转换等)

### 2.判断函数

#### 2.1 是否为空格(`IsSpace`)

```go
func TestJudge(t *testing.T) {
	rune1 := ' '
	rune2 := 'h'
	fmt.Printf("[%c] 是空格? %t\n",rune1,unicode.IsSpace(rune1))
  // 输出 [ ] 是空格? true
	fmt.Printf("[%c] 是空格? %t\n",rune2,unicode.IsSpace(rune2))
  // 输出 [h] 是空格? false
}
```

#### 2.2 是否为十进制数(`IsDigit`)

```go
d1 := '1'
d2 := 'w'
fmt.Printf("[%c] 是十进制数? %t\n",d1,unicode.IsDigit(d1))
// 输出: [1] 是十进制数? true
fmt.Printf("[%c] 是十进制数? %t\n",d2,unicode.IsDigit(d2))
// 输出: [w] 是十进制数? false
```

#### 2.3 是否为数字(`IsNumber`)

```go
d1 := '1'
d2 := 'w'
fmt.Printf("[%c] 是数字? %t\n",d1,unicode.IsNumber(d1))
// 输出: [1] 是数字? true
fmt.Printf("[%c] 是数字? %t\n",d2,unicode.IsNumber(d2))
// 输出: [w] 是数字? false
```

#### 2.4 是否为字母字符(`IsLetter`)

```go
// 是否为字母字符
str1 := '刘'
str2 := 'l'
str3 := 'W'
str4 := '!'
fmt.Printf("[%c] 是字母? %t\n",str1,unicode.IsLetter(str1))
// [刘] 是字母? true
fmt.Printf("[%c] 是字母? %t\n",str2,unicode.IsLetter(str2))
// [l] 是字母? true
fmt.Printf("[%c] 是字母? %t\n",str3,unicode.IsLetter(str3))
// [W] 是字母? true
fmt.Printf("[%c] 是字母? %t\n",str4,unicode.IsLetter(str4))
// [!] 是字母? false
```

<font color=red>@注意: 汉字也是一个字母字符</font>

#### 2.5 是否为标点符号(`IsPunct`)

```go
str1 := '刘'
str2 := 'l'
str3 := '!'
fmt.Printf("[%c] 是标点符号? %t\n",str1,unicode.IsPunct(str1))
// [刘] 是标点符号? false
fmt.Printf("[%c] 是标点符号? %t\n",str2,unicode.IsPunct(str2))
// [l] 是标点符号? false
fmt.Printf("[%c] 是标点符号? %t\n",str3,unicode.IsPunct(str3))
// [!] 是标点符号? true
```

#### 2.6 是否为小写字母(`IsLower`)

```go
str1 := '刘'
str2 := 'l'
str3 := 'W'
str4 := '!'
fmt.Printf("[%c] 是小写字母? %t\n", str1, unicode.IsLower(str1))
// [刘] 是小写字母? false
fmt.Printf("[%c] 是小写字母? %t\n", str2, unicode.IsLower(str2))
// [l] 是小写字母? true
fmt.Printf("[%c] 是小写字母? %t\n", str3, unicode.IsLower(str3))
// [W] 是小写字母? false
fmt.Printf("[%c] 是小写字母? %t\n", str4, unicode.IsLower(str4))
// [!] 是小写字母? false
```

#### 2.7  是否为大写字母(`IsUpper`)

```go
str1 := '刘'
str2 := 'l'
str3 := 'W'
str4 := '!'
fmt.Printf("[%c] 是大写字母? %t\n", str1, unicode.IsUpper(str1))
// [刘] 是大写字母? false
fmt.Printf("[%c] 是大写字母? %t\n", str2, unicode.IsUpper(str2))
// [l] 是大写字母? false
fmt.Printf("[%c] 是大写字母? %t\n", str3, unicode.IsUpper(str3))
// [W] 是大写字母? true
fmt.Printf("[%c] 是大写字母? %t\n", str4, unicode.IsUpper(str4))
// [!] 是大写字母? false
```

#### 2.8 是否为汉字

```go
str1 := '刘'
str2 := 'l'
str3 := '!'
fmt.Printf("[%c] 是汉字? %t\n", str1, unicode.Is(unicode.Scripts["Han"], str1))
// [刘] 是汉字? true
fmt.Printf("[%c] 是汉字? %t\n", str2, unicode.Is(unicode.Scripts["Han"], str2))
// [l] 是汉字? false
fmt.Printf("[%c] 是汉字? %t\n", str3, unicode.Is(unicode.Scripts["Han"], str3))
// [!] 是汉字? false
```

### 3. 转换函数

#### 3.1 转成小写(`ToLower`)

```go
str1 := 'W'
fmt.Printf("[%c] 转成小写: %c \n", str1, unicode.ToLower(str1))
// [W] 转成小写: w 
```

#### 3.2 转成大写(`ToUpper`)

```go
str2 := 'a'
fmt.Printf("[%c] 转成大写: %c \n", str2, unicode.ToUpper(str2))
// [a] 转成大写: A
```





