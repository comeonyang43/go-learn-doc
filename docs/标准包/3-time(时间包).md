### 1.获取时间

#### 1.1 当前时间

```go
unix := time.Now().Unix()
fmt.Printf("当前时间戳(单位秒): %v \n",unix)
nano := time.Now().UnixNano()
fmt.Printf("当前时间戳(单位纳秒): %v\n",nano)
format := time.Now().Format("2006-01-02 15:04:05")
fmt.Printf("当前时间(Y-m-d H:i:s): %v\n",format)
format2 := time.Now().Format("20060102150405")
fmt.Printf("当前时间(YmdHis): %v\n",format2)
/**
当前时间戳(单位秒): 1616251448 
当前时间戳(单位纳秒): 1616251448041163000
当前时间(Y-m-d H:i:s): 2021-03-20 22:44:08
当前时间(YmdHis): 20210320224408
*/
```



#### 1.2 当前年、月、日、时、分、秒、星期几

```go
format := time.Now().Format("2006-01-02 15:04:05")
fmt.Printf("当前时间(Y-m-d H:i:s): %v\n",format)
fmt.Printf("当前年: %v\n", time.Now().Year())
fmt.Printf("当前月: %v\n", time.Now().Month())
fmt.Printf("当前日: %v\n", time.Now().Day())
fmt.Printf("当前小时: %v\n", time.Now().Hour())
fmt.Printf("当前分钟: %v\n", time.Now().Minute())
fmt.Printf("当前秒: %v\n", time.Now().Second())
fmt.Printf("当前星期几: %v\n", time.Now().Weekday())
/**输出
当前时间(Y-m-d H:i:s): 2021-03-21 17:32:24
当前年: 2021
当前月: March
当前日: 21
当前小时: 17
当前分钟: 32
当前秒: 24
当前星期几: Sunday
*/
```

#### 1.3 获取时分秒

```go
now := time.Now()
hour, min, sec := now.Clock()
fmt.Printf("时间:%v   huor: %v min:%v sec:%v \n",now,hour,min,sec)
//时间:2021-03-22 23:18:43.245006 +0800 CST m=+0.000108086   huor: 23 min:18 sec:43
```

#### 1.4 创建指定时间

```go
date := time.Date(2020,8,23,13,53,32,0,time.Local)
fmt.Printf("类型: %T , %v ",date,date)
// 输出:类型: time.Time , 2020-08-23 13:53:32 +0800 CST 
```

#### 1.5 获取所属第几周

```go
// 获取当前时间是，今年的第几周
week, w := time.Now().ISOWeek()
fmt.Printf("year: %v w:%v \n",week,w)
// year: 2021 w:12 
```

#### 1.6 获取x年的第几天

```go
num := time.Now().YearDay()
fmt.Printf("今天是今年的第 %v 天 \n",num)
// 今天是今年的第 81 天 
```





### 2.字符串转时间

#### 2.1 字符串转时间类型

```go
str := "1616319808"
unix, _ := strconv.ParseInt(str,10,64)
format := time.Unix(unix, 0)
fmt.Printf("字符串时间戳-> 类型:%T 值:%v \n",format,format)
// 字符串时间,
strDate := "2019-04-10 12:54:03"
// 注意layout格式需要和字符串时间格式一致
location, _ := time.Parse("2006-01-02 15:04:05", strDate)
fmt.Printf("字符串时间-> 类型:%T 值:%v \n",location,location)
/**
字符串时间戳-> 类型:time.Time 值:2021-03-21 17:43:28 +0800 CST 
字符串时间-> 类型:time.Time 值:2019-04-10 12:54:03 +0000 UTC 
*/
```

#### 2.2 格式化时间

`Format`根据`layout`指定的格式返回`t`代表的时间点的格式化文本表示；`Format`中内置了几个固定的`layout`常量。

```go
const (
	ANSIC       = "Mon Jan _2 15:04:05 2006"
	UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
	RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
	RFC822      = "02 Jan 06 15:04 MST"
	RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
	RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
	RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
	RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
	RFC3339     = "2006-01-02T15:04:05Z07:00"
	RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
	Kitchen     = "3:04PM"
	// Handy time stamps.
	Stamp      = "Jan _2 15:04:05"
	StampMilli = "Jan _2 15:04:05.000"
	StampMicro = "Jan _2 15:04:05.000000"
	StampNano  = "Jan _2 15:04:05.000000000"
)
```

**使用:**

```go
// 将字符时间: 2020-09-12 14:34:10 转成 20200912143410
strDate := "2020-09-12 14:34:10"
// 1.先转成时间类型
location, _ := time.ParseInLocation("2006-01-02 15:04:05", strDate, time.Local)
// 2.再格式化
format := location.Format("20060102150405")
fmt.Println(format)
// 输出:20200912143410
```

### 3.时间比较

```go
strDate := "2020-09-12 12:00:00"
strDate2 := "2020-09-12 12:00:00"
// 1.先转成时间类型
time1, _ := time.ParseInLocation("2006-01-02 15:04:05", strDate, time.Local)
time2, _ := time.ParseInLocation("2006-01-02 15:04:05", strDate2, time.Local)
// 2.比较 strDate2 > strDate ?
before := time1.Before(time2)
fmt.Printf("strDate2 > strDate ? %t \n",before)
// 3.比较 strDate2 = strDate ?
equal := time1.Equal(time2)
fmt.Printf("strDate2 = strDate ? %t \n",equal)
/**输出
strDate2 > strDate ? false 
strDate2 = strDate ? true 
*/
```

### 4.时间计算

#### 4.1 时间相加

```go
now := time.Now()
fmt.Printf("现在的时间: %v \n",now)
// 十分钟前
duration, _ := time.ParseDuration("-10m")
fmt.Printf("十分钟前: %v \n", now.Add(duration))
// 一小时前
duration2, _ := time.ParseDuration("-1h")
fmt.Printf("一小时前: %v \n", now.Add(duration2))
// 一天后
fmt.Printf("一天后: %v \n", now.AddDate(0,0,1))
fmt.Printf("一月后: %v \n", now.AddDate(0,1,0))
fmt.Printf("一年后: %v \n", now.AddDate(1,0,0))
/**
现在的时间: 2021-03-22 22:54:16.008615 +0800 CST m=+0.000092572 
十分钟前: 2021-03-22 22:44:16.008615 +0800 CST m=-599.999907428 
一小时前: 2021-03-22 21:54:16.008615 +0800 CST m=-3599.999907428 
一天后: 2021-03-23 22:54:16.008615 +0800 CST 
一月后: 2021-04-22 22:54:16.008615 +0800 CST 
一年后: 2022-03-22 22:54:16.008615 +0800 CST 
*/
```

#### 4.2 时间相减

```go
day1, _ := time.ParseInLocation("2006-01-02 15:04:05", "2020-11-02 12:00:00", time.Local)
day2, _ := time.ParseInLocation("2006-01-02 15:04:05", "2020-11-02 15:00:00", time.Local)
fmt.Printf("相差几秒: %v \n",day2.Sub(day1).Seconds())
fmt.Printf("相差多少分钟: %v \n",day2.Sub(day1).Minutes())
fmt.Printf("相差多少小时: %v \n",day2.Sub(day1).Hours())
/**
相差几秒: 10800 
相差多少分钟: 180 
相差多少小时: 3 
*/
```

### 5.定时器

#### 5.1 NewTicker

`NewTicker`返回一个新的`Ticker`，该`Ticker`包含一个通道字段，并会每隔时间段`d`就向该通道发送当时的时间。它会调整时间间隔或者丢弃`tick`信息以适应反应慢的接收者。如果`d<=0会panic`。关闭该`Ticker`可以释放相关资源。

** 使用示例**

```go
func main() {
	// 创建定时器，间隔设置每秒
	ticker := time.NewTicker(time.Second)
	// 启动一个协程，打印定时器里面的时间
	go func(ticker *time.Ticker) {
		for i := 0; i < 3; i++ {
			fmt.Println(<-ticker.C)
		}
		// 关闭定时器
		ticker.Stop()
	}(ticker)
	// 手动阻塞
	time.Sleep(3 * time.Second)
	fmt.Println("end")
}
/**输出
2021-03-30 11:33:23.704677 +0800 CST m=+1.003656575
2021-03-30 11:33:24.704695 +0800 CST m=+2.003675713
2021-03-30 11:33:25.704271 +0800 CST m=+3.003275554
end
*/
```

#### 5.2 Tick

`Tick`是`NewTicker`的封装，只提供对`Ticker`通道的访问。如果不需要关闭`Ticker`，本函数就很方便。

```go
func main() {
	// 创建定时器，间隔设置每秒
	chTime := time.Tick(time.Second)
	// 启动一个协程，打印定时器里面的时间
	go func(ch <- chan time.Time) {
		for i := 0; i < 3; i++ {
			fmt.Println(<-ch)
		}
	}(chTime)
	// 手动阻塞
	time.Sleep(4 * time.Second)
	fmt.Println("end")
}
```

### 6.延迟执行

#### 6.1 NewTimer

`NewTimer`创建一个`Timer`，它会在指定时间后，向其自身的`C`字段发送当时的时间。

```go
func main() {
	fmt.Printf("开始时间: %v \n",time.Now())
	timer := time.NewTimer(time.Second * 3)
	// 此处会阻塞，直到timer.C中有数据写入
	fmt.Printf("timer通道里的时间: %v \n", <- timer.C)
}
/**输出
开始时间: 2021-03-30 14:27:04.966887 +0800 CST m=+0.001502710 
timer通道里的时间: 2021-03-30 14:27:07.970134 +0800 CST m=+3.004701898 
*/
```

#### 6.2 After

`After()`函数相当于`NewTimer(d). C`,如下源码:

```go
func After(d Duration) <-chan Time {
	return NewTimer(d).C
}
```

**使用示例**

```go
func main() {
	// 创建一个计时器,返回的是chan
	ch := time.After(5 * time.Second)
	fmt.Printf("开始时间 %v \n",time.Now())
	// 此处会阻塞5秒
	out := <- ch
	fmt.Printf("变量out->  类型: %T 值:%v  \n",out,out)
	fmt.Printf("开始时间 %v \n",time.Now())
}
/** 输出
  开始时间 2020-12-10 11:01:07.272154 +0800 CST m=+0.000153152 
  变量out->  类型: time.Time 值:2020-12-10 11:01:12.273034 +0800 CST m=+5.000956630  
  开始时间 2020-12-10 11:01:12.273153 +0800 CST m=+5.001076196 
*/
```





