### 1.介绍

`bufio`包实现了有缓冲的`I/O`。它包装一个`io.Reader`或`io.Writer`接口对象，使用这个包可以大幅提高文件读写的效率。

#### 1.如何提高效率?

![image-20210408135753993](https://go.liuqh.icu/img/image-20210408135753993.png)

**1.写入流程梳理:**

- 当写入内容小于缓冲区(`buf`)的可用大小时,内容写入缓存区(`buf`)；
- 当缓冲区(`buf`)空间不够时，一次性将缓冲区(`buf`)内容写入文件,并清空缓存区(`buf`)；
- 当写入内容大于缓冲区(`buf`)空间时，将内容直接写入文件；

**2.读取流程梳理:**

- 当读取内容小于缓冲区(`buf`)空间时,从缓存区(`buf`)读取；
- 当缓冲区(`buf`)内容为空时，一次性从文件中读取大小等于缓冲区(`buf`)的内容；
- 当写入内容大于缓冲区(`buf`)空间时，将内容直接写入文件；

<font color=red>总结:   避免频繁操作文件、减少访问本地磁盘次数，从而提高效率。</font>

### 2.读取(`Read*`)

#### 2.1 创建读取器

##### 1. 创建默认缓冲大小 (`NewReader`)

###### a.签名

```go
func NewReader(rd io.Reader) *Reader
```

根据`rd`创建一个具有默认大小缓冲的`*Reader`。默认大小:`4096字节`

###### b.使用

```go
reader := bufio.NewReader(strings.NewReader("hello Word"))
fmt.Printf("默认大小:%v",reader.Size())
// 输出: 默认大小:4096
```

##### 2. 创建指定缓冲大小( `NewReaderSize`)
###### 1.签名

```go
func NewReaderSize(rd io.Reader, size int) *Reader
```

根据`rd`创建指定`size`大小缓冲的`*Reader`。最小大小为`16字节`

###### 2.使用

```go
reader := bufio.NewReaderSize(strings.NewReader("hello Word"),40)
fmt.Printf("大于16字节:%v\n",reader.Size())
reader2 := bufio.NewReaderSize(strings.NewReader("hello Word"),4)
fmt.Printf("小于16字节:%v\n",reader2.Size())
/**输出
大于16字节:40
小于16字节:16
*/
```

#### 2.2 读取指定字节(`Read`)

##### 1.签名

```go
func (b *Reader) Read(p []byte) (n int, err error)
```

将读取数据写入`p`。返回写入`p`的字节数和错误信息。当读取到达结尾时，返回值n将为0而err将为`io.EOF`。

##### 2.使用

现有文件`a.txt`,内容如下:

```properties
张三 23 北京 男
李四 22 南京 男
郭明 32 上海 男
王英 18 武汉 女
```

**读取程序如下：**

```go
func main() {
	//打开一个文件
	file, err := os.Open("./a.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	// 关闭文件
	defer file.Close()
	// 创建读取器
	reader := bufio.NewReader(file)
	for  {
		// 一次读取21个字节
		context := make([]byte, 21)
		_, err = reader.Read(context)
		// 读取完毕，则跳出
		if err == io.EOF {
			break
		}
		fmt.Printf("读取内容:%s", context)
	}
}
/*输出
读取内容:张三 23 北京 男
读取内容:李四 22 南京 男
读取内容:郭明 32 上海 男
读取内容:王英 18 武汉 女
*/
```

#### 2.3 读取一个字节(`ReadByte`)

##### 1.签名

```go
func (b *Reader) ReadByte() (byte, error)
```

每次读取一个字节返回。当读取到达结尾时，返回值n将为0而err将为`io.EOF`。

##### 2.使用

```go
func main() {
	// 基于字符串创建读取器
	reader := bufio.NewReader(strings.NewReader("Go,Word!"))
	for  {
		// 一次读取1个字节
		readByte, err := reader.ReadByte()
		// 读取完毕，则跳出
		if err == io.EOF {
			break
		}
		fmt.Printf("读取内容:%s\n", string(readByte))
	}
}
/**输出
读取内容:G
读取内容:o
读取内容:,
读取内容:W
读取内容:o
读取内容:r
读取内容:d
读取内容:!
*/
```

#### 2.4 读取一行(`ReadLine`)

##### 1.签名

```go
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```

`ReadLine`尝试读取返回一行数据，不包括行尾标志的字节,但是有以下几个场景需要注意:

- 如果行太长超过了缓冲，返回值`isPrefix=true`,`Line`为行的前一部分。该行剩下部分将在下一次读取中返回并且此时的`isPrefix=false`,<font color=red>**即`isPrefix`会在行的最后一个片段时才设为`false`**。</font>
- 如果行内小于等于缓冲,`isPrefix=false`,`Line`为一整行的内容。

> <font color=red>官方文档中推荐使用 `ReadBytes('\n')`或`ReadString('\n')`来代替`ReadLine`。</font>

##### 2.当行超过了缓冲缓存时

```go
func main() {
	// 创建字符串，每行超过16个字节
	str := strings.Repeat("Hello,Hello,Hello,Hello!\n",3)
	// 基于字符串创建一个缓冲区=16字节的读取器
	reader := bufio.NewReaderSize(strings.NewReader(str),16)
	for  {
		// 每次读取一行
		line, prefix, err := reader.ReadLine()
		// 读取完毕，则跳出
		if err == io.EOF {
			break
		}
		fmt.Printf("line:%s isPrefix:%t \n", line,prefix)
	}
}
```

**输出:**

![image-20210410172327103](https://go.liuqh.icu/img/image-20210410172327103.png)

##### 3.当行小于缓冲缓存时

```go
func main() {
	// 每行不超过16个字节
	str := strings.Repeat("Hello Word!\n",3)
	// 基于字符串创建读取器
	reader := bufio.NewReaderSize(strings.NewReader(str),16)
	for  {
		// 每次读取一行
		line, prefix, err := reader.ReadLine()
		// 读取完毕，则跳出
		if err == io.EOF {
			break
		}
		fmt.Printf("line:%s isPrefix:%t \n", line,prefix)
	}
}
/**输出
line:Hello Word! isPrefix:false 
line:Hello Word! isPrefix:false 
line:Hello Word! isPrefix:false 
*/
```

#### 2.5 读取到指定的字符

##### 1.同类方法列表

| 方法                                             | 无`delim`是否报错          |
| ------------------------------------------------ | -------------------------- |
| `ReadSlice(delim byte) (line []byte, err error)` | 报错(`bufio: buffer full`) |
| `ReadBytes(delim byte) ([]byte, error)`          | 不会                       |
| `ReadString(delim byte) (string, error)`         | 不会                       |

每次读取到字符`delim`(包含)

##### 2.常规使用

```go
func main() {
	// 基于字符串创建读取器
	reader := bufio.NewReaderSize(strings.NewReader("Go,PHP,Java,Python,C"),16)
	for  {
		// 读取
		s, err := reader.ReadString(',')
		// 读取完毕，则跳出
		if err == io.EOF {
			break
		} else if err != nil{
			fmt.Printf("err:%s",err.Error())
			break
		}
		fmt.Printf("%s\n", s)
	}
}
/**输出
Go,
PHP,
Java,
Python,
*/
```

> 上述例子中`C`后面没有`,`,所以不会读出

##### 3.模拟每次读取一行

```go
	// 设置每行超过缓冲区大小
	repeat := strings.Repeat("Hello,Hello,Hello,Hello\n",3)
	// 基于字符串创建读取器
	reader := bufio.NewReaderSize(strings.NewReader(repeat),16)
	for  {
		// 读取
		s, err := reader.ReadString('\n')
		// 读取完毕，则跳出
		if err == io.EOF {
			break
		} else if err != nil{
			fmt.Printf("err:%s",err.Error())
			break
		}
		fmt.Printf("%s", s)
	}
}
/** 输出
Hello,Hello,Hello,Hello
Hello,Hello,Hello,Hello
Hello,Hello,Hello,Hello
*/
```

> 虽然每行超过缓冲区大小，但是依然可以整行读出

#### 2.6 每次读取前几个字节(Peek)

##### 1.签名

```go
func (b *Reader) Peek(n int) ([]byte, error)
```

每次读取输入流的前`n`个字节，且不会移动读取位置,所以当输入流不变时，每次读取都是一样的。

- 当内容小于`n`时，`error`会返回`io.EOF`
- 当`n`大于缓冲区大小时，`error`会返回`ErrBufferFull`

##### 2.内容小于`n`时

```go
func main() {
	// 基于字符串创建读取器
	reader := bufio.NewReaderSize(strings.NewReader("hello"),16)
	// 读取
	for i := 0; i < 2; i++ {
		s, err := reader.Peek(16)
    fmt.Printf("结果: %s ", s)
		if err == io.EOF {
			fmt.Printf("%s","已读完")
			break
		} else if err != nil{
			fmt.Printf("err:%s",err.Error())
			break
		}
	}
}
// 输出-> 结果: hello 已读完
```

##### 3.当`n`大于缓冲区大小时

```go
func main() {
	// 基于字符串创建读取器
	reader := bufio.NewReaderSize(strings.NewReader("hello"),16)
	// 读取
	for i := 0; i < 2; i++ {
		// 大于缓冲区大小
		s, err := reader.Peek(17)
		fmt.Printf("结果: %s ", s)
		if err == io.EOF {
			fmt.Printf("%s","已读完")
			break
		} else if err != nil{
			fmt.Printf("err:%s",err.Error())
			break
		}
	}
}
// 输出-> 结果: hello err:bufio: buffer full
```

### 3.写入(`Write*`)

#### 3.1 创建写入器

##### 1.创建默认缓冲大小(`NewWriter`)

```go
func NewWriter(w io.Writer) *Writer
```

根据`w`创建一个具有默认大小缓冲的`*Writer`。默认大小:`4096字节`

###### 2.创建默认缓冲大小(`NewWriter`)

```go
func NewWriterSize(w io.Writer, size int) *Writer 
```

#### 3.2 `Write*`

##### 1.方法列表

| 方法名                                | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `Write(p []byte) (nn int, err error)` | `Write`将`p`的内容写入缓冲。返回写入的字节数。<br>如果返回值`nn < len(p)`，还会返回一个错误说明原因。 |
| `WriteString(s string) (int, error)`  | 同上,只是入参变成字符串。                                    |



##### 2.当写入内容大于缓冲区时

```go
func main() {
	file, _ := os.OpenFile("./test.txt",os.O_APPEND|os.O_WRONLY,os.ModePerm)
	defer file.Close()
	reader := bufio.NewWriterSize(file,20)
	// 当写入的内容字节大于缓冲区大小时,会直接写入文件
	write, err := reader.Write([]byte("hello,hello,hello 你好！"))
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("直接写入文件: %d",write)
}
// 输出——> 直接写入文件: 27
```

##### 3.当写入内容小于缓冲区时

```go
func main() {
	file, _ := os.OpenFile("./test.txt",os.O_APPEND|os.O_WRONLY,os.ModePerm)
	defer file.Close()
	reader := bufio.NewWriterSize(file,20)
	// 当写入的内容字节小于缓冲区大小时,不会直接写入文件
	write, err := reader.Write([]byte("\nhello,Go!"))
	if err != nil {
		fmt.Println(err)
		return
	}
	// 该函数会使缓冲区的内容，直接写入文件
	_ = reader.Flush()
	fmt.Printf("Flush写入文件: %d",write)
}
// 输出-> Flush写入文件: 10
```

![](https://go.liuqh.icu/img/image-20210412184116487.png)

