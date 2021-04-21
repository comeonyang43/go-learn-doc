### 1.介绍

`Elasticsearch`是一个分布式、可扩展、近实时的高性能搜索与数据分析引擎。在`Go`中经常使用的包有以下两个:

| 包                                                           | 文档                                        | Star 数量 |
| ------------------------------------------------------------ | ------------------------------------------- | --------- |
| [olivere/elastic](https://github.com/olivere/elastic)        | https://olivere.github.io/elastic/          | 5.7k      |
| [elastic/go-elasticsearch](https://github.com/elastic/go-elasticsearch) | https://github.com/elastic/go-elasticsearch | 3.2k      |

### 2.安装

这里使用`olivere/elastic`

```sh
# 安装v7的版本
go get github.com/olivere/elastic/v7
```

### 3.配置

#### 3.1 编写配置文件

```ini
[elastic]
# 是否开启
enabled=true
#es服务地址
url=http://127.0.0.1:9200
# 是否转换请求地址，默认为true,当等于true时 请求http://ip:port/_nodes/http，将其返回的url作为请求路径
sniffer_enabled=false
# 心跳检查,间隔时间
health_check_interval=5s
```

#### 3.2 编写配置结构体

```go
/**
 * @description: ES配置
 * @user: Mr.LiuQH
 */
type ElasticConfig struct {
	Enabled             bool   `ini:"enabled"`
	Url                 string `ini:"url"`
	SnifferEnabled      bool   `ini:"sniffer_enabled"`
	HealthCheckInterval string `ini:"health_check_interval"`
}
```

#### 3.3 加载配置

```go
/**
 * @description: 加载配置文件
 * @user: Mr.LiuQH
 * @receiver app
 * @date 2021-02-04 15:29:17
 */
func (app *App) loadConfig() {
	iniPath := ConfigPath + app.Env + ".ini"
	CliInfoInstance.ConfigFile = iniPath
	cfg, err := ini.Load(iniPath)
	BusErrorInstance.ThrowError(err)
	err = cfg.Section("app").MapTo(app)
	// 加载mysql配置
	err = cfg.Section("mysql").MapTo(MysqlConfigInstance)
	// 加载redis配置
	err = cfg.Section("redis").MapTo(RedisConfigInstance)
	// 加载日志配置
	err = cfg.Section("log").MapTo(LogrusConfigInstance)
	// 加载ES配置
	err = cfg.Section("elastic").MapTo(ElasticConfigInstance)
	BusErrorInstance.ThrowError(err)
}
```

### 4.初始化客户端

```go

/**
 * @description:
 * @user: Mr.LiuQH
 * @date 2021-03-11 15:56:48
 */
func connectElastic() {
	duration, err := time.ParseDuration(ElasticConfigInstance.HealthCheckInterval)
	BusErrorInstance.ThrowError(err)
  // ElasticClient是全局变量
	ElasticClient, err = elastic.NewClient(
		elastic.SetURL(ElasticConfigInstance.Url),
		elastic.SetSniff(ElasticConfigInstance.SnifferEnabled),
		elastic.SetHealthcheckInterval(duration),
		elastic.SetErrorLog(log.New(os.Stderr, "ES ", log.LstdFlags)),
		elastic.SetInfoLog(log.New(os.Stdout, " ", log.LstdFlags)),
	)
	BusErrorInstance.ThrowError(err)
	res, err := ElasticClient.ClusterHealth().Index("test").Level("shards").Pretty(true).Do(context.TODO())
	LoggerClient.Info("es:%v", res)
}
```

### 5.使用

[示例源码地址(https://github.com/52lu/goe/blob/master/app/controllers/v1/elastic.go)](https://github.com/52lu/goe/blob/master/app/controllers/v1/elastic.go)

#### 5.1 创建索引

```go

// 定义用户mapping
const userMapping = `
{
    "mappings":{
        "properties":{
            "name":{
                "type":"text"
            },
            "age":{
                "type":"byte"
            },
            "phone":{
                "type":"text"
            },
            "birth":{
                "type":"date"
            },
            "height":{
                "type":"float"
            },
            "smoke":{
                "type":"boolean"
            },
            "home":{
                "type":"geo_point"
            }
        }
    }
}`

// 定义索引名
const userIndex = "go-user"
var esCtx = context.Background()
// 创建索引
func (e ElasticController) CreateIndex() error {
	exist, err := ElasticClient.IndexExists(userIndex).Do(esCtx)
	if err != nil {
		return e.Error(err.Error())
	}
	if exist {
		return e.Error("索引已经存在，无需重复创建!")
	}
	// 创建索引
	res, err := ElasticClient.CreateIndex(userIndex).BodyString(userMapping).Do(esCtx)
	LoggerClient.WithFields(logrus.Fields{
		"res": res,
	}).Info("es返回")
	if err != nil {
		return e.Error(err.Error())
	}
	return e.Success(res)
}

```

#### 5.2 添加

```go
type User struct {
	Name   string    `json:"name"`
	Age    int       `json:"age"`
	Phone  string    `json:"phone"`
	Birth  time.Time `json:"birth"`
	Height float32   `json:"height"`
	Smoke  bool      `json:"smoke"`
	Home   string    `json:"home"`
}

// 添加文档
func (e ElasticController) AddOneDoc() error {
	// 定义上下文
	loc, err := time.LoadLocation("Local")
	birth, err := time.ParseInLocation("2006-01-02", "1991-04-25", loc)
	userOne := User{"张三", 23, "17600000000", birth, 170.5, false, "41.40338,2.17403"}
	do, err := ElasticClient.Index().Index(userIndex).Id(strconv.FormatInt(time.Now().UnixNano(), 10)).BodyJson(userOne).Do(esCtx)
	if err != nil {
		return e.Error(err.Error())
	}
	return e.Success(do)
}
```

#### 5.3 批量添加

```go
// 批量添加
func (e ElasticController) BatchAddDoc() error {
	userBulk := ElasticClient.Bulk().Index(userIndex)
	loc, _ := time.LoadLocation("Local")
	// 生日
	birthSlice := []string{"1991-04-25", "1990-01-15", "1989-11-05", "1988-01-25", "1994-10-12"}
	// 姓名
	nameSlice := []string{"李四", "张飞", "赵云", "关羽", "刘备"}
	rand.Seed(time.Now().Unix())
	for i := 1; i < 20; i++ {
		birth, _ := time.ParseInLocation("2006-01-02", birthSlice[rand.Intn(len(birthSlice))], loc)
		height, _ := strconv.ParseFloat(fmt.Sprintf("%.2f", rand.Float32()+175.0), 32)
		user := User{
			nameSlice[rand.Intn(len(nameSlice))],
			rand.Intn(10) + 18,
			"1760000000" + strconv.Itoa(i),
			birth,
			float32(height),
			false,
			"41.40338,2.17403",
		}
		fmt.Println(user, userBulk)
		doc := elastic.NewBulkIndexRequest().Id(strconv.FormatInt(time.Now().UnixNano(), 10)).Doc(user)
		userBulk.Add(doc)
	}
	if userBulk.NumberOfActions() < 0 {
		return e.Error("没有要保存的数据")
	}
	if _, err := userBulk.Do(esCtx); err != nil {
		return e.Error(err.Error())
	}
	return e.Success(nil)
}
```



#### 5.4 查询

```go
// 根据名字查询
func (e ElasticController) Get() error {
	name := e.GetParam("name")
	list, err := ElasticClient.Search().Index(userIndex).
		Query(elastic.NewMatchQuery("name", name)).
		Sort("age", true). //根据age字段，正序
		From(0). // 从第几条开始取
		Size(20). // 每页数量
		Pretty(true). //返回json格式
		Do(context.Background())
	if err != nil {
		return e.Error(err.Error())
	}
	var userDto User
	var userList []User
	for _, v := range list.Each(reflect.TypeOf(userDto)) {
		tmp := v.(User)
		userList = append(userList, tmp)
	}
	// 遍历获取结果
	return e.Success(userList)
}
```

#### 5.5 删除

```go
// 删除
func (e ElasticController) Del() error {
	id := e.GetParam("id")
	name := e.GetParam("name")
	age := e.GetParam("age")
	var do interface{}
	var err error
	if id != "" {
		// 根据主键删除
		do, err = ElasticClient.Delete().Index(userIndex).Id(id).Do(context.Background())
	} else if name != "" && age != "" {
		// 根据名称和年龄删除
		do, err = ElasticClient.DeleteByQuery().Index(userIndex).
			Query(elastic.NewMatchQuery("name", name)).
			Query(elastic.NewTermQuery("age", age)).
			Do(context.Background())
	} else {
		return e.Error("缺少参数信息")
	}
	if err != nil {
		return e.Error(err.Error())
	}
	return e.Success(do)
}
```

#### 5.6 更新

```go
// 更新
func (e ElasticController) Update() error {
	id := e.GetParam("id")
	phone := e.GetParam("phone")
	var do interface{}
	var err error
	if id != "" {
		// 根据主键更新
		do, err = ElasticClient.Update().Index(userIndex).Id(id).
			Script(elastic.NewScript("ctx._source.age=73")).
			Script(elastic.NewScript("ctx._source.phone='110110110110'")).
			Do(context.Background())
	} else if phone != "" {
		// 根据非ID条件更新
		do, err = ElasticClient.UpdateByQuery(userIndex).Query(elastic.NewTermQuery("phone", phone)).
			// 通过脚本更新字段name
			Script(elastic.NewScript("ctx._source.name='龙少爷'")).
			// 如果文档版本冲突继续执行
			ProceedOnVersionConflict().
			Do(context.Background())
	} else {
		return e.Error("缺少参数信息")
	}
	if err != nil {
		return e.Error(err.Error())
	}
	return e.Success(do)
}
```

