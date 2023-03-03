### 1.介绍

在`Go`语言中，`io`包主要声明的是对`I/O`操作的基本接口，而`io/ioutil`包则实现了对应的方法，方便我们直接使用。

### 2.方法列表

| 方法名      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| `NopCloser` | 对`Reader`进行封装，返回一个`ReadCloser`                     |
| `ReadAll`   | 对 `Reader` 进行读取， 直到发生错误或者遇到` EOF` 为止， 然后返回被读取的数据。 |
| `ReadFile`  | 读取指定文件的内容，并返回。                                 |
| `ReadDir`   | 读取 指定的目录， 并返回排序后的目录列表(根据名称排序)       |
| `TempDir`   | 在指定目录新创建一个临时目录， 并返回该目录的路径            |
| `TempFile`  | 在指定目录新创建一个临时文件， 以可读写的方式打开它， 并返回 `*os.File` 指针 |
| `WriteFile` | 将给定的数据 写入到指定文件里面。 <br/>如果文件不存在， 则去创建它； <br/>如果文件已经存在， 则再写入之前会先清空文件中已有的内容。 |

### 3.`ReadAll`

#### 3.1 方法签名

```go
func ReadAll(r io.Reader) ([]byte, error) 
```

`ReadAll`从`r`读取数据直到`EOF`或`error`，返回读取内容(`[]byte`)和错误(`error`)。读取成功时`err返回nil`。<font color=purple>因为本函数定义为读取`r`直到`EOF`，它不会将读取返回的`EOF`视为应报告的错误。</font>

#### 3.2 使用

```go
func main() {
	// NewReader创建一个从s读取数据的Reader。本函数类似bytes.NewBufferString
	reader := strings.NewReader("Hello word !")
	all, err := ioutil.ReadAll(reader)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("%s",all)
}
// 输出: Hello word !
```

### 4.`ReadFile`

#### 4.1 方法签名

```go
func ReadFile(filename string) ([]byte, error) 
```

`ReadFile` 从`filename`指定的文件中读取内容并返回。读取成功时`err返回nil`。<font color=purple>因为本函数定义为读取`r`直到`EOF`，它不会将读取返回的`EOF`视为应报告的错误。</font>

#### 4.2 使用

```go
func main() {
   // 文件路径
   fileName := "./test.log"
   data, err := ioutil.ReadFile(fileName)
   if err != nil {
      fmt.Println(err)
      return
   }
   fmt.Printf("%s",data)
}
```

### 5.`ReadDir`

#### 5.1 方法签名

```go
func ReadDir(dirname string) ([]os.FileInfo, error) 
```

返回`dirname`指定的目录的目录信息的有序列表。

#### 5.2 使用

```go
func main() {
	dirName := "./log"
	dirList, err := ioutil.ReadDir(dirName)
	if err != nil {
		fmt.Println(err)
		return
	}
	for _, info := range dirList {
		fmt.Println(info.Name())
	}
}
/**输出
d1
test.log
*/
```

![image-20210402180333879](https://go.liuqh.icu/img/image-20210402180333879.png)

### 6.`TempDir`

#### 6.1 方法签名

```go
func TempDir(dir, pattern string) (name string, err error)
```

在`dir`目录里创建一个新的、使用`pattern`作为前缀的临时文件夹，并返回文件夹的路径。

- 如果`dir`是空字符串，`TempDir`使用默认用于临时文件的目录。 
- 不同程序同时调用该函数会创建不同的临时目录，
- 创建的临时文件夹时在不使用时，需要删除。

#### 6.2 使用

```go
func main() {
	var pathList []string
	for i := 0; i < 4; i++ {
		var path string
		var err error
		switch i {
		case 0:
			// 指定目录和前缀
			path, err = ioutil.TempDir("./log","tmp_")
		case 1:
			// 只指定目录
			path, err = ioutil.TempDir("./log","")
		case 2:
			// 目录为空、前缀不为空
			path, err = ioutil.TempDir("","tmp_")
		case 3:
			// 目录和前缀都是空
			path, err = ioutil.TempDir("","")
		}
		if err != nil {
			fmt.Println(err)
			continue
		}
		pathList = append(pathList, path)
    // 删除临时目录,测试时注释
		// _ = os.RemoveAll(path)
	}
	fmt.Println(pathList)
}
/**输出
[log/tmp_984088491 log/346734350 /var/folders/8f/vnx2y1h57nzbd2p4vtcl6hv80000gn/T/tmp_561537301 /var/folders/8f/vnx2y1h57nzbd2p4vtcl6hv80000gn/T/840593776]
*/
```

![image-20210402190041423](https://go.liuqh.icu/img/image-20210402190041423.png)

### 7.`TempFile`

#### 7.1 方法签名

```go
func TempFile(dir, pattern string) (f *os.File, err error)
```
在`dir`目录下创建一个新的、使用`pattern`为前缀的临时文件，以读写模式打开该文件并返回`os.File`指针。

- 如果`dir`是空字符串，`TempFile`使用默认用于临时文件的目录。
- 不同程序同时调用该函数会创建不同的临时文件。
- 创建的临时文件夹时在不使用时，需要删除。


#### 7.2 使用

```go
func main() {
	var fileList []string
	for i := 0; i < 4; i++ {
		var file *os.File
		var err error
		switch i {
		case 0:
			// 指定目录和前缀
			file, err = ioutil.TempFile("./log", "tmp_")
		case 1:
			// 只指定目录
			file, err = ioutil.TempFile("./log", "")
		case 2:
			// 目录为空、前缀不为空
			file, err = ioutil.TempFile("", "tmp_")
		case 3:
			// 目录和前缀都是空
			file, err = ioutil.TempFile("", "")
		}
		if err != nil {
			fmt.Println(err)
			continue
		}
		fileList = append(fileList, file.Name())
		//删除
		//_ = os.Remove(file.Name())
	}
	fmt.Println(fileList)
}
/**输出
[log/tmp_598309912 log/983448727 /var/folders/8f/vnx2y1h57nzbd2p4vtcl6hv80000gn/T/tmp_134984970 /var/folders/8f/vnx2y1h57nzbd2p4vtcl6hv80000gn/T/246937825]
*/
```

![image-20210402192208342](https://go.liuqh.icu/img/image-20210402192208342.png)

### 8.`WriteFile`

#### 8.1 方法签名

```go
func WriteFile(filename string, data []byte, perm os.FileMode) error
```

函数向`filename`指定的文件中写入数据。

- 如果文件不存在,将按给出的权限创建文件，
- 如果文件存在,在写入数据之前清空文件。

#### 8.2 使用

```go
func main() {
	fileName := "./test.txt"
	for i := 0; i < 3; i++ {
		_ = ioutil.WriteFile(fileName, []byte("Hello Word!"), os.ModePerm)
	}
	fmt.Println("end")
}
```

