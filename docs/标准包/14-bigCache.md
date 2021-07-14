### 1. 介绍

`BigCache` 是一个快速，支持并发访问，自淘汰的内存型缓存，可以在存储大量元素的同时依然保持高性能。`BigCache`将元素保存在堆上却避免了`GC`的开销。[源码地址:https://github.com/allegro/bigcache](https://github.com/allegro/bigcache)

#### 1.1 为什么开发`bigcache`?

`bigcache`团队接到一个任务，需要开发一个非常快速的缓存服务，并满足以下几点需求:

- 使用 HTTP 协议处理请求。
- 处理`10k rps` (写5000，读5000)。
-  `cache`对象至少存活10分钟。
- 更快的响应时间。
- `POST`请求的每条 `JSON` 消息,一有含有ID，二不大于500字节.
- `POST`请求添加缓存后，`GET`能获取到最新结果。

> <font color=red>简单地说，我们的任务是编写一个带有过期和 REST 接口的快速字典。</font>



#### 1.2 为什么不用第三方服务?

为了满足上述任务需求,要求开发的`cache`库要保证：

- 即使有百万的缓存对象也要非常快
- 支持大并发访问
- 一定时间后支持剔除

**官方原文:**

> Considering the first point we decided to give up external caches like [Redis](http://redis.io/), [Memcached](http://memcached.org/) or [Couchbase](http://www.couchbase.com/) mainly because of additional time needed on the network. Therefore we focused on in-memory caches. In Go there are already caches of this type, i.e. [LRU groups cache](https://github.com/golang/groupcache/tree/master/lru), [go-cache](https://github.com/patrickmn/go-cache), [ttlcache](https://github.com/diegobernardes/ttlcache), [freecache](https://github.com/coocood/freecache). Only freecache fulfilled our needs. Next subchapters reveal why we decided to roll our own anyway and describe how the characteristics mentioned above were achieved.

**翻译后:**

> 考虑到第一点，我们决定放弃外部缓存，如 Redis，Memcached 或 Couchbase 主要是因为额外的时间需要在网络上。因此，我们主要关注内存缓存。在 Go 中已经有这种类型的缓存，如 `LRU groups cache`、` Go-cache`、 `ttlcache` 和` freecache`。只有 `freecache `满足了我们的需要。接下来的分章揭示了为什么我们决定滚动我们自己的无论如何，并描述了如何实现上面提到的特点。



### 2. 安装

```go
go get -u github.com/allegro/bigcache 
```

### 3. 初始化

#### 3.1 默认初始化

**a. 代码示例**

```go
// 默认初始化
func TestInitDefaultCache(t *testing.T) {
	// 创建一个LifeWindow为5秒的cache实例
	cache, _ := bigcache.NewBigCache(bigcache.DefaultConfig(time.Second * 5))
  defer cache.Close()
	// 设置缓存
	err := cache.Set("key1", []byte("hello word"))
	if err != nil {
		t.Errorf("设置缓存失败:%v",err)
	}
	// 获取缓存
	data, err := cache.Get("key1")
	if err != nil {
		t.Errorf("获取缓存失败:%v",err)
	}
	fmt.Printf("获取结果:%s\n",data)
}
```

**b. 输出**

```bash
=== RUN   TestInitDefaultCache
获取结果:hello word
--- PASS: TestInitDefaultCache (0.03s)
PASS
```

#### 3.2 自定义初始化

**a. 代码示例**

```go
// 创建自定义缓存
func TestInitCustom(t *testing.T) {
	// 指定创建属性
	config := bigcache.Config{
		// 设置分区的数量，必须是2的整倍数
		Shards: 1024,
		// LifeWindow后,缓存对象被认为不活跃,但并不会删除对象
		LifeWindow: 5 * time.Second,
		// CleanWindow后，会删除被认为不活跃的对象，0代表不操作；
		CleanWindow: 3 * time.Second,
		// 设置最大存储对象数量，仅在初始化时可以设置
		//MaxEntriesInWindow: 1000 * 10 * 60,
		MaxEntriesInWindow: 1,
		// 缓存对象的最大字节数，仅在初始化时可以设置
		MaxEntrySize: 500,
		// 是否打印内存分配信息
		Verbose: true,
		// 设置缓存最大值(单位为MB),0表示无限制
		HardMaxCacheSize: 8192,
		// 在缓存过期或者被删除时,可设置回调函数，参数是(key、val)，默认是nil不设置
		OnRemove: callBack,
		// 在缓存过期或者被删除时,可设置回调函数，参数是(key、val,reason)默认是nil不设置
		OnRemoveWithReason: nil,
	}
	cache,err := bigcache.NewBigCache(config)
	if err != nil {
		t.Error(err)
	}
	defer cache.Close()
	// 设置缓存
	_ = cache.Set("key1", []byte("hello word"))
	// 验证CleanWindow是否生效
	time.Sleep(10 * time.Second)
	// 获取缓存
	data, err := cache.Get("key1")
	if err != nil {
		t.Errorf("获取缓存失败:%v",err)
	}
	fmt.Printf("获取结果:%s\n",data)
	fmt.Println("运行结束！")
}
```

**b.输出**

```bash
=== RUN   TestInitCustom
过期回调: key=key1 val=hello word 
    bigcache_test.go:74: 获取缓存失败:Entry not found
获取结果:
运行结束！
--- FAIL: TestInitCustom (10.00s)
```

> <font color=red>在实际使用中发现,只设置`CleanWindow = n`,缓存并不一定会在`n`秒后自动删除，需要结合</font>`LifeWindow`。如`CleanWindow = 4s LifeWindow=3s 代表 4s后会删除LifeWindow中已经被标记为不活跃的缓存(有效期为3s)`

### 4.使用

#### 4.1 添加和获取( `Set|Get`)

```go
func TestSetAndGet(t *testing.T) {
	cache, _ := bigcache.NewBigCache(bigcache.DefaultConfig(time.Minute))
	// 设置缓存
	err := cache.Set("key1", []byte("php"))
	if err != nil {
		t.Errorf("设置缓存失败:%v",err)
	}
	_ = cache.Set("key2",[]byte("go"))
	// 获取缓存
	for _, key := range []string{"key1","key2"} {
		if data, err := cache.Get(key);err == nil {
			fmt.Printf("key: %s 结果:%s\n",key,data)
		}
	}
}
/** 输出
=== RUN   TestSetAndGet
key: key1 结果:php
key: key2 结果:go
--- PASS: TestSetAndGet (0.02s)
PASS
*/
```

#### 4.2 删除缓存(`Delete`)

```go
func TestDelCache(t *testing.T) {
	cache, _ := bigcache.NewBigCache(bigcache.DefaultConfig(time.Minute))
	key := "key"
	// 设置
	_ = cache.Set(key,[]byte("111"))
	// 删除
	_ = cache.Delete(key)
	// 获取
	if _, err := cache.Get(key);err != nil {
		fmt.Println(err)
	}
}
/** 输出
=== RUN   TestUseCache
Entry not found
--- PASS: TestUseCache (0.02s)
PASS
*/
```

#### 4.3 长度和容量(`Len|Capacity`)

```go
// 统计缓存数量和容量
func TestLenAndCap(t *testing.T) {
	cache, _ := bigcache.NewBigCache(bigcache.DefaultConfig(time.Minute))
	_ = cache.Set("key", []byte("1"))
	_ = cache.Set("key1", []byte("1"))
	_ = cache.Set("key2", []byte("1"))
	_ = cache.Set("key3", []byte("1"))
	fmt.Printf("缓存数量: %d \n", cache.Len())
	fmt.Printf("缓存容量: %d \n", cache.Capacity())
}

/** 输出
=== RUN   TestLen
缓存数量: 4 
缓存容量: 299520000 
--- PASS: TestLen (0.02s)
PASS
*/
```

#### 4.4 重置(`Reset`)

```go
// 重置所有分区的缓存
func TestReset(t *testing.T) {
	cache, _ := bigcache.NewBigCache(bigcache.DefaultConfig(time.Minute))
	for i := 0; i < 10; i++ {
		k := fmt.Sprintf("key%d",i)
		_ = cache.Set(k,[]byte(strconv.Itoa(i)))
	}
	fmt.Printf("重置前缓存数量: %d \n", cache.Len())
	// 重置所有分区的缓存
	_ = cache.Reset()
	fmt.Printf("重置后缓存数量: %d \n", cache.Len())
}
/** 输出
=== RUN   TestReset
重置前缓存数量: 10 
重置后缓存数量: 0 
--- PASS: TestReset (0.02s)
PASS
*/
```
