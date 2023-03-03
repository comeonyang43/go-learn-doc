## 1.流程介绍

![](./img/20210901190333.png)

## 2.页面分析

![](./img/20210901183654.png)

## 3. 爬虫脚本

###  3.1 编写结构体

**代码文件位置：`service/crawl/fund/top_crawl.go`**

```go
// 对应每一行的基金信息
type fundItem struct {
	FundCode         string `selector:"td:nth-of-type(1)"`
	FundName         string `selector:"td:nth-of-type(2)"`
	NetWorth         string `selector:"td:nth-of-type(3) > span.fb"`
	TopDate          string `selector:"td:nth-of-type(3) > span.date"`
	DayChange        string `selector:"td:nth-of-type(4)"`
	WeekChange       string `selector:"td:nth-of-type(5)"`
	MouthChange      string `selector:"td:nth-of-type(6)"`
	ThreeMouthChange string `selector:"td:nth-of-type(7)"`
	SixMouthChange   string `selector:"td:nth-of-type(8)"`
	YearChange       string `selector:"td:nth-of-type(9)"`
	TwoYearChange    string `selector:"td:nth-of-type(10)"`
	ThreeYearChange  string `selector:"td:nth-of-type(11)"`
	CurrentChange    string `selector:"td:nth-of-type(12)"`
	CreateChange     string `selector:"td:nth-of-type(13)"`
}
type TopCrawlService struct {
	Item []*fundItem `selector:"tr"`
}
```

`td:nth-of-type(n)`代表每行的第`n`列，<font color=red>@注意：这里的NetWorth和TopDate,取的都是第3列的数据</font>

![](./img/20210901185821.png)

### 3.2 爬取网页

**代码文件位置：`service/crawl/fund/top_crawl.go`**

```go
// CrawlHtml 抓取网页信息
func (f *TopCrawlService) CrawlHtml() {
	collector := colly.NewCollector(
		colly.UserAgent(crawl.UserAgent),
	)
	// 设置Header
	collector.OnRequest(func(request *colly.Request) {
		request.Headers.Set("Accept-Language", "zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7")
	})
	collector.OnError(func(response *colly.Response, err error) {
		global.GvaLogger.Sugar().Errorf("基金排行榜,信息获取失败: %s", err)
		return
	})
  // 选中id=tblite_hh的table,使用Unmarshal方法会把每行数据，自动映射结构体
	collector.OnHTML("#tblite_hh", func(element *colly.HTMLElement) {
		err := element.Unmarshal(f)
		if err != nil {
			fmt.Println("element.Unmarshal error: ", err)
		}
	})
	// 获取响应
	collector.OnResponse(func(response *colly.Response) {
		// 将返回中的html中所有的%去掉
		newBody := strings.ReplaceAll(string(response.Body), "%", "")
		response.Body = []byte(newBody)
	})
	// 爬取收益排行榜,(默认是按照近一年的排行)
	err := collector.Visit("https://fundact.eastmoney.com/banner/hh.html")
	if err != nil {
		global.GvaLogger.Sugar().Errorf("基金排行榜爬取失败: %s", err)
	}
}
```

### 3.3 数据清洗

**代码文件位置：`service/crawl/fund/top_crawl.go`**

```go
// ConvertEntity 格式化类型
func (f *TopCrawlService) ConvertEntity() []entity.FundDayTop {
	var topList []entity.FundDayTop
	for _, item := range f.Item {
		if item.FundCode == "" {
			continue
		}
		fundTmp := entity.FundDayTop{}
		fundTmp.FundCode = item.FundCode
		// 格式化日期
		format := time.Now().Format("2006")
		fundTmp.TopDate = fmt.Sprintf("%s-%s", format, item.TopDate)
		// 转换编码
		fundTmp.FundName, _ = utils.GbkToUtf8(item.FundName)
		// 字符串转浮点型
		fundTmp.NetWorth, _ = strconv.ParseFloat(item.NetWorth, 64)
		fundTmp.DayChange, _ = strconv.ParseFloat(item.DayChange, 64)
		fundTmp.WeekChange, _ = strconv.ParseFloat(item.WeekChange, 64)
		fundTmp.MouthChange, _ = strconv.ParseFloat(item.MouthChange, 64)
		fundTmp.ThreeMouthChange, _ = strconv.ParseFloat(item.ThreeMouthChange, 64)
		fundTmp.SixMouthChange, _ = strconv.ParseFloat(item.SixMouthChange, 64)
		fundTmp.YearChange, _ = strconv.ParseFloat(item.YearChange, 64)
		fundTmp.TwoYearChange, _ = strconv.ParseFloat(item.TwoYearChange, 64)
		fundTmp.ThreeYearChange, _ = strconv.ParseFloat(item.ThreeYearChange, 64)
		fundTmp.CurrentChange, _ = strconv.ParseFloat(item.CurrentChange, 64)
		fundTmp.CreateChange, _ = strconv.ParseFloat(item.CreateChange, 64)
		topList = append(topList, fundTmp)
	}
	return topList
}
```

## 4. 定时任务

### 4.1 实现`cron.Job`接口

**代码文件位置：`crontab/fund_top_cron.go`**

```go
type FundTopCron struct {}
func (c FundTopCron) Run()  {
	fmt.Println("基金排行榜-定时任务准备运行....")
	f := &fund.TopCrawlService{}
	// 爬取网页
	f.CrawlHtml()
	// 转换数据
	fundDayTopList := f.ConvertEntity()
	// 入库
	if !f.ExistTopDate() {
		result := global.GvaMysqlClient.Create(fundDayTopList)
		if result.Error != nil {
			global.GvaLogger.Sugar().Errorf("本次任务保存数据失败：%条",result.Error)
			return
		}
		global.GvaLogger.Sugar().Infof("本次任务保存数据：%条",result.RowsAffected)
		return
	}
	global.GvaLogger.Sugar().Info("任务运行成功，无数据要保存！")
	fmt.Println("基金排行榜-定时任务运行结束！")
}
```

### 4.2 注册任务

**代码文件位置：`initialize/cron.go`**

```go
// 添加Job任务
func addJob(c *cron.Cron) {
	...
  // 爬取每日基金排行榜单:每天16:30执行
	// _, _ = c.AddJob("0 30 16 */1 * *", crontab.FundTopCron{})
  // 为了测试，先写成 每10s一次
  _, _ = c.AddJob("@every 10s", crontab.FundTopCron{})
}
```

## 5. 效果展示

### 5.1 运行项目

```bash
...
[GIN-debug] GET    /demo/es/create           --> 52lu/fund-analye-system/api/demo.CreateIndex (3 handlers)
[GIN-debug] GET    /demo/es/searchById       --> 52lu/fund-analye-system/api/demo.SearchById (3 handlers)

【 当前环境: dev 当前版本: v1.0.0 接口地址: http://0.0.0.0:8088 】
基金排行榜-定时任务准备运行....
基金排行榜-定时任务运行结束！
```

### 5.2 数据表

![](./img/20210901192350.png)




