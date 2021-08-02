## 1. 介绍

`Go`标准包`math`,提供了基本的数学常数和数学函数。

## 2. 常量

### 2.1 整数取值

```go
const (
    MaxInt8   = 1<<7 - 1 // int8最大值
    MinInt8   = -1 << 7 // int8最小值
    MaxInt16  = 1<<15 - 1
    MinInt16  = -1 << 15
    MaxInt32  = 1<<31 - 1
    MinInt32  = -1 << 31
    MaxInt64  = 1<<63 - 1
    MinInt64  = -1 << 63
    MaxUint8  = 1<<8 - 1
    MaxUint16 = 1<<16 - 1
    MaxUint32 = 1<<32 - 1
    MaxUint64 = 1<<64 - 1
)
```

### 2.2 浮点数取值

```go
const (
    MaxFloat32             = 3.40282346638528859811704183484516925440e+38
    SmallestNonzeroFloat32 = 1.401298464324817070923729583289916131280e-45
    MaxFloat64             = 1.797693134862315708145274237317043567981e+308
    SmallestNonzeroFloat64 = 4.940656458412465441765687928682213723651e-324
)
```

## 3. 函数

### 3.1 取整

```go
// 取整
func TestRounding(t *testing.T) {
	// 向上取整
	fmt.Println("50.345,向上取整 -> ", math.Ceil(50.345))
	fmt.Println("50.745,向上取整 -> ", math.Ceil(50.745))
	// 向下取整
	fmt.Println("50.345,向上取整 -> ", math.Floor(50.345))
	fmt.Println("50.745,向上取整 -> ", math.Floor(50.745))
	// 四舍五入
	fmt.Println("50.345,四舍五入 -> ", math.Floor(50.345+0.5))
	fmt.Println("50.745,四舍五入 -> ", math.Floor(50.745+0.5))
}
/** 输出
=== RUN   TestRounding
50.345,向上取整 ->  51
50.745,向上取整 ->  51
50.345,向上取整 ->  50
50.745,向上取整 ->  50
50.345,四舍五入 ->  50
50.745,四舍五入 ->  51
--- PASS: TestRounding (0.00s)
PASS
*/
```

<font color=red>@@注意: golang没有四舍五入函数，常见处理先+0.5，然后向下取整！</font>

### 3.2 最大和最小

```go
// 最大值和最小值
func TestCompare(t *testing.T) {
	a := 12.4555
	b := 12.8234
	fmt.Printf("%.4f和%.4f,最大值是:%.4f \n", a,b,math.Max(a,b))
	fmt.Printf("%f和%f,最小值是:%f \n", a,b,math.Min(a,b))
}
/** 输出
=== RUN   TestCompare
12.4555和12.8234,最大值是:12.8234
12.455500和12.823400,最小值是:12.455500
--- PASS: TestCompare (0.00s)
PASS
*/
```

### 3.3 取模和取余

```go
// 取模和取余
func TestR(t *testing.T) {
	a := 20.0
	b := -3.0
	fmt.Printf("%.2f对%.2f 取模:%.2f \n", a,b,math.Mod(a,b))
	fmt.Printf("%.2f对%f 取余:%.2f \n", a,b,math.Remainder(a,b))
}
/** 输出
=== RUN   TestR
20.00对-3.00 取模:2.00
20.00对-3.000000 取余:-1.00
--- PASS: TestR (0.00s)
PASS
*/
```

### 3.4 随机数

`rand`属于`math`子包

```go
// 随机数
func TestRand(t *testing.T) {
	// 设置随机因子(需要设置成非固定值)
	rand.Seed(time.Now().Unix())
	for i := 0; i < 5; i++ {
		fmt.Println("随机整数[0,100): ",rand.Intn(100))
		fmt.Println("随机浮点数[0.0,1.0):",rand.Float64())
	}
}
```

<font color=red>@@注意: `golang`使用一个`seed`作为`source`来生成伪随机数字，默认`seed`为1，只要`seed`固定了，那么所有随机数就固定了。**如果seed固定，那么每次程序重启后重新生成随机数会重复上一次的随机数。**</font>