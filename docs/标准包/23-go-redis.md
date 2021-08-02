## 1. 介绍

`redis`官网推荐使用`redigo`(`https://github.com/gomodule/redigo`)，截止到今天`Github Start是8.2k` 但`go-redis(https://github.com/go-redis/redis)`使用的人更多， 并且`go-redis`封装得更好。截止到今天`Github Start 是12.1k`。

## 2. 安装

### 2.1 历史版本

```go
go get -u github.com/go-redis/redis
```

### 2.2 最新版本

```go
go get github.com/go-redis/redis/v8
```

<font color=red>@注意: 最新版本的客户端在操作redis时，相关函数需要传递上下文(`context.Context`),以下内容都是基于最新版本。</font>

## 3. 连接`Redis`

### 3.1 单机连接(`NewClient`)

```go
// 单机连接redis
func TestConnect(t *testing.T) {
	client := redis.NewClient(&redis.Options{
		Addr: "127.0.0.1:6379",
	})
	// 检测是否建立连接(需要传递上下文)
	timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), time.Second*5)
	defer cancelFunc()
	// 检测
	_, err := client.Ping(timeoutCtx).Result()
	if err != nil {
		t.Error(err)
    return
	}
}
```

### 3.2 哨兵模式连接(`NewFailoverClient`)

```go
// 哨兵模式连接
func TestConnectSentinel(t *testing.T) {
	client := redis.NewFailoverClient(&redis.FailoverOptions{
		MasterName:    "master-name",
		SentinelAddrs: []string{":9126", ":9127", ":9128"},
	})
	// 检测是否建立连接(需要传递上下文)
	timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), time.Second*5)
	defer cancelFunc()
	// 检测
	_, err := client.Ping(timeoutCtx).Result()
	if err != nil {
		t.Error(err)
		return
	}
}
```

### 3.3 集群模式连接(`NewClusterClient`)

```go
// 集群模式连接
func TestConnectCluster(t *testing.T) {
	client := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{":7000", ":7001", ":7002", ":7003"},
	})
	// 检测是否建立连接(需要传递上下文)
	timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), time.Second*5)
	defer cancelFunc()
	// 检测
	_, err := client.Ping(timeoutCtx).Result()
	if err != nil {
		t.Error(err)
		return
	}
}
```

## 4. redis.Options参数详解

```go
type Options struct {
  // 网络类型:tcp|unix 默认:tcp
	Network string
  
	// redis地址，格式 host:port
	Addr string
  
  // 当新建一个redis连接的时候，会回调这个函数
	OnConnect func(ctx context.Context, cn *Conn) error
  
  // 指定用户名连接(从redis6.0版本或更高以上支持)
	Username string
  
	// redis密码，可以为空。
	Password string
  
  // redis数据库，范围:0~15，默认是0，可以为空
	DB int
  
  // 最大重试次数，默认是3次，-1:表示关闭
	MaxRetries int
  
	// 最小重试时间间隔，默认是 8ms ; -1 表示关闭.
	MinRetryBackoff time.Duration
  
	// 最大重试时间间隔.默认是 512ms; -1 表示关闭.
	MaxRetryBackoff time.Duration

	// redis连接超时时间.默认是 5 秒.
	DialTimeout time.Duration
  
	// 读取超时时间,默认3秒.
	ReadTimeout time.Duration
  
	// 写入超时时间,默认和ReadTimeout一致
	WriteTimeout time.Duration

  // redis最大连接数,默认连接池大小等于 runtime.GOMAXPROCS(cpu个数) * 10
	PoolSize int
  
	// 启动时，创建最小空闲连接数
	MinIdleConns int
  
	// redis连接最大的存活时间，默认不会关闭过时的连接.
	MaxConnAge time.Duration
  
  // 当从redis连接池获取一个连接之后，连接池最多等待这个拿出去的连接多长时间。
	// 默认是等待 ReadTimeout + 1 秒.
	PoolTimeout time.Duration
  
	// redis连接池多久会关闭一个空闲连接.
	// 默认是 5 分钟. -1 则表示关闭这个配置项
	IdleTimeout time.Duration
  
  // 多长时间检测一下，空闲连接
	// 默认是 1 分钟. -1 表示关闭空闲连接检测
	IdleCheckFrequency time.Duration

	// 设置只读模式，如果设置为true redis只能查不能更新
	readOnly bool

  // 限流接口
	Limiter Limiter
}
```

## 5. 基本键值操作

### 5.1 设置和获取

#### 1.单值操作: `Set和Get`

```go
// Get&Set
func TestGetAndSet(t *testing.T) {
	// 参考上面的单机模式连接redis
	client, err := goredis.ConnectSingle()
	if err != nil {
		t.Error(err)
		return
	}
	ctx := context.Background()
	// 设置缓存
	key := "test:abc"
	err = client.Set(ctx, key, "hello word!", time.Minute*10).Err()
	if err != nil {
		t.Error("设置缓存失败"+err.Error())
		return
	}
	// 获取缓存
	result, err := client.Get(ctx, key).Result()
	if err != nil {
		t.Error("获取缓存失败"+err.Error())
		return
	}
	fmt.Println("获取结果: ",result)
}
/*** 输出
=== RUN   TestGetAndSet
获取结果:  hello word!
--- PASS: TestGetAndSet (0.01s)
PASS
*/
```
#### 2.批量操作: `MSet和MGet`
```go
// 批量设置和获取
func TestMGetSet(t *testing.T) {
	// 连接redis
	client, _ := goredis.ConnectSingle()
	ctx := context.Background()
	// MSet: 同时设置一个或多个 key-value 对。
	err := client.MSet(ctx, "key1", "val1", "key2", "val2", "key3", "val3").Err()
	if err != nil {
		t.Error(err)
		return
	}
	// 使用Get获取缓存
	for _, key := range []string{"key1","key2","key3"} {
		result, _ := client.Get(ctx, key).Result()
		fmt.Printf("Get获取,键: %s 值: %v \n",key,result)
	}
	// 使用MGet获取
	result, _ := client.MGet(ctx, "key1", "key2","key3").Result()
	fmt.Println("MGet批量获取:",result)
}

/**输出
=== RUN   TestMGetSet
Get获取,键: key1 值: val1 
Get获取,键: key2 值: val2 
Get获取,键: key3 值: val3 
MGet批量获取: [val1 val2 val3]
--- PASS: TestMGetSet (0.01s)
PASS
*/
```


### 5.2 锁( `SetNX`)

```go
// SetNX: 指定的 key 不存在时，为 key 设置指定的值
func TestSetNx(t *testing.T) {
	// 连接redis
	client, err := goredis.ConnectSingle()
	if err != nil {
		t.Error(err)
		return
	}
	ctx := context.Background()
	for i := 0; i < 3; i++ {
		res, err := client.SetNX(ctx, "abc", time.Now().Unix(), time.Hour).Result()
		if err != nil {
			t.Error(err)
			break
		}
		fmt.Println("SetNX abc success :", res)
	}
}
/** 输出
=== RUN   TestSetNx
SetNX abc success : true
SetNX abc success : false
SetNX abc success : false
--- PASS: TestSetNx (0.01s)
PASS
*/
```

### 5.3 自增自减

```go
// 自增和自减
func TestIncrAndDecr(t *testing.T) {
	// 连接redis
	client, _ := goredis.ConnectSingle()
	ctx := context.Background()
	// 自增
	for i := 1; i <= 5; i++ {
		// Incr: 每次调用+1
		client.Incr(ctx,"incr1")
		// IncrBy: 每次调用+5
		client.IncrBy(ctx,"incr2",5)
		// IncrByFloat: 每次调用 +0.5
		client.IncrByFloat(ctx,"incr3",0.5)
		// 查询缓存
		result, _ := client.MGet(ctx, "incr1", "incr2", "incr3").Result()
		fmt.Printf("第%d次自增后查询: %v \n",i,result)
	}
	// 自减
	for i := 1; i <= 5; i++ {
		// Decr: 每次调用-1
		client.Decr(ctx,"decr1")
		// DecrBy: 每次调用-5
		client.DecrBy(ctx,"decr2",5)
		// 查询缓存
		result, _ := client.MGet(ctx, "decr1", "decr2").Result()
		fmt.Printf("第%d次自减后查询: %v \n",i,result)
	}
}

/**输出
=== RUN   TestIncrAndDecr
第1次自增后查询: [10 50 5] 
第2次自增后查询: [11 55 5.5] 
第3次自增后查询: [12 60 6] 
第4次自增后查询: [13 65 6.5] 
第5次自增后查询: [14 70 7] 
第1次自减后查询: [-5 -25] 
第2次自减后查询: [-6 -30] 
第3次自减后查询: [-7 -35] 
第4次自减后查询: [-8 -40] 
第5次自减后查询: [-9 -45] 
--- PASS: TestIncrAndDecr (0.08s)
PASS
*/
```

### 5.4 删除和追加

```go
// 删除和追加
func TestDelAndAppend(t *testing.T) {
	client, _ := goredis.ConnectSingle()
	ctx := context.Background()
  // ========== 删除测试 ==========
	// 删除单个
	client.Del(ctx,"key1")
	// 删除多个
	client.Del(ctx,"incr1","incr2","incr3")

	// ======== 追加value 测试使用 ========
	client.Set(ctx,"key","hello",time.Hour)
	// 获取值
	res1, _ := client.Get(ctx,"key").Result()
	fmt.Println("追加前的值:",res1)
	// 追加
	client.Append(ctx,"key"," word")
	res2, _ := client.Get(ctx,"key").Result()
	fmt.Println("追加后的值:",res2)
}
/**输出
=== RUN   TestDelAndAppend
追加前的值: hello
追加后的值: hello word
--- PASS: TestDelAndAppend (0.02s)
PASS
*/
```

## 6. 列表(`List`)操作

### 6.1 插入

```go
func TestInsertList(t *testing.T) {
	// 连接redis
	client, _ := ConnectSingle()
	ctx := context.Background()
	key := "insertList"
	//从列表头部插入,不存在则新建并插入数据
	for _, val := range []string{"php", "go"} {
		// 插入头部(左边)
		client.LPush(ctx, key, val)
		// 获取
		result, _ := client.LRange(ctx, key, 0, -1).Result()
		fmt.Printf("LPush 从头部插入【%v】: %v\n", val,result)
	}
	// 从列表尾部插入
	for _,val := range []string{"张三","李四"} {
		// 插入尾部(右边)
		client.RPush(ctx, key, val)
		// 获取
		result, _ := client.LRange(ctx, key, 0, -1).Result()
		fmt.Printf("RPush 从尾部插入【%v】: %v\n", val,result)
	}
	result, _ := client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("当前列表所有值: %+v\n", result)

	// 在指定的值前插入
	client.LInsertBefore(ctx,key,"php","php5.6")
	result, _ = client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("在php前插入%v,当前列表所有值: %v\n", "php5.6",result)
	// 在指定的值后插入
	client.LInsertAfter(ctx,key,"go","go1.0")
	result, _ = client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("在go后插入%v,当前列表所有值: %v\n", "go1.0",result)
}
/**输出
=== RUN   TestInsertList
LPush 从头部插入【php】: [php]
LPush 从头部插入【go】: [go php]
RPush 从尾部插入【张三】: [go php 张三]
RPush 从尾部插入【李四】: [go php 张三 李四]
当前列表所有值: [go php 张三 李四]
在php前插入php5.6,当前列表所有值: [go php5.6 php 张三 李四]
在go后插入go1.0,当前列表所有值: [go go1.0 php5.6 php 张三 李四]
--- PASS: TestInsertList (0.04s)
PASS
*/
```

### 6.2 读取

```go
func TestReadList(t *testing.T) {
	// 连接redis
	client, _ := ConnectSingle()
	ctx := context.Background()
	key := "language-list"
	// 插入元素
	client.LPush(ctx, key, "php", "go", "java", "c", "c++", "python")
	fmt.Println("插入列表: ", "php", "go", "java", "c", "c++", "python")
	// 获取长度
	result, _ := client.LLen(ctx, key).Result()
	fmt.Println("列表长度: ", result)
	// =========获取指定key的值=======
	val, _ := client.LIndex(ctx, key, 0).Result()
	fmt.Println("获取索引为0的值: ", val)

	// =========获取指定范围的值=======
	// 获取[0,2]的元素
	strings, _ := client.LRange(ctx, key, 0, 2).Result()
	fmt.Printf("获取列表位置为[0,2]的元素：%v\n", strings)
	// 获取全部元素[0,-1]
	all, _ := client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("获取列表所有[0,-1]元素：%v\n", all)
}
/**输出
=== RUN   TestReadList
插入列表:  php go java c c++ python
列表长度:  6
获取索引为0的值:  python
获取列表位置为[0,2]的元素：[python c++ c]
获取列表所有[0,-1]元素：[python c++ c java go php]
--- PASS: TestReadList (0.02s)
PASS
*/
```

### 6.3 删除

```go
func TestDelList(t *testing.T)  {
	// 连接redis
	client, _ := ConnectSingle()
	ctx := context.Background()
	key := "language-list"
	// 插入元素
	client.LPush(ctx, key, "php", "go", "java", "c", "c++", "python")
	all, _ := client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("当前列表所有值：%v\n", all)

	// 移出并获取列表的第一个元素
	first, _ := client.LPop(ctx, key).Result()
	fmt.Printf("LPop,移出并获取列表的第一个元素：%v\n", first)
	// 当前列表所有值
	all, _ = client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("当前列表所有值：%v\n", all)
	// 移出并获取列表的最后一个元素
	last, _ := client.RPop(ctx, key).Result()
	fmt.Printf("RPop,移出并获取列表的最后一个元素：%v\n", last)
	// 当前列表所有值
	all, _ = client.LRange(ctx, key, 0, -1).Result()
	fmt.Printf("当前列表所有值：%v\n", all)
}
/** 输出
=== RUN   TestDelList
当前列表所有值：[python c++ c java go php]
LPop,移出并获取列表的第一个元素：python
当前列表所有值：[c++ c java go php]
RPop,移出并获取列表的最后一个元素：php
当前列表所有值：[c++ c java go]
--- PASS: TestDelList (0.02s)
PASS
*/
```





