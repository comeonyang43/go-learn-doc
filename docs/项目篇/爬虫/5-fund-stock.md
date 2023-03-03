## 1. 爬取流程

![爬取流程](https://go.liuqh.icu/img/20210907115003.png)



## 2. 准备工作

### 2.1 分页网页

![页面数据展示](https://go.liuqh.icu/img/20210907115245.png)



![数据接口返回](https://go.liuqh.icu/img/20210907115624.png)

<font color=red>@注意：这次爬取的网页数据是通过ajax加载，所以不能直接使用OnHtml抓取。</font>

### 2.2 编写结构体

```go
// 对应表中的每一tr
type StockPercentageRow struct {
	StockCode  string `selector:"td:nth-of-type(2)"`
	StockName  string `selector:"td:nth-of-type(3)"`
	Percentage string `selector:"td:nth-of-type(7)"`
	Quantity   string `selector:"td:nth-of-type(8)"`
	Amount     string `selector:"td:nth-of-type(9)"`
}
// 对应整个table
type StockPercentageRowsCrawl struct {
	Rows       []StockPercentageRow `selector:"tr"`
	FundCode   string
	CutOffDate string
}
```

## 3. 代码实现

### 3.1 请求流程图

![请求流程图](https://go.liuqh.icu/img/20210907161728.png)



### 3.2 抓取入口函数

**文件位置：`crontab/fund_stock_cron.go`**

```go
type FundStockCron struct {
}

// 声明并发等待组
var wg sync.WaitGroup

// 每次任务抓取总数量
var perTaskTotal = 50

// 记录每次任务对应的基金code
var fundCodeChannel = make(chan []string, perTaskTotal)

// 定时任务启动入口
func (c FundStockCron) Run() {
	btime := time.Now().UnixMilli()
	fmt.Println("基金持仓-股票定时任务准备执行....")
	pageNum := 10
	totalPage := int(math.Ceil(float64(perTaskTotal) / float64(pageNum)))
	// 开启协程分组抓取
	// 创建数据通道
	var dataChan = make(chan [][]entity.FundStock, perTaskTotal/pageNum)
	runWithGoroutine(dataChan, totalPage, pageNum)
	// 读取通道，数据入库
	saveToDb(dataChan)
	fmt.Printf("基金持仓股票-定时任务执行完成，耗时:%vms\n", time.Now().UnixMilli()-btime)
}
```

### 3.3 开启协程分组抓取

**文件位置：`crontab/fund_stock_cron.go`**

```go
// 开启协程分组抓取
func runWithGoroutine(dataChan chan [][]entity.FundStock, totalPage, pageNum int) {
	// 延迟关闭chan
	defer close(dataChan)
	defer close(fundCodeChannel)
	// 开启协程抓取
	wg.Add(totalPage)
	for i := 1; i <= totalPage; i++ {
		page := i
		go func() {
			// 获取对应页数的code
			fundStocks, err := dao.FindNoSyncFundStockByPage(page, pageNum)
			if err == nil {
				var fundStockList [][]entity.FundStock
				var fundCodes []string
				for _, val := range fundStocks {
					rows := &fund.StockPercentageRowsCrawl{}
					rows.CrawlHtml(val.Code)
					fundCodes = append(fundCodes, val.Code)
					if len(rows.Rows) > 0 {
						convertEntity := rows.ConvertEntity()
						fundStockList = append(fundStockList, convertEntity)
					}
				}
				// 数据存入通道
				dataChan <- fundStockList
				fundCodeChannel <- fundCodes
			}
			// 并发等待组减一
			wg.Done()
		}()
	}
	wg.Wait()
}
```

### 3.4 爬取函数(`CrawlHtml`)

**文件位置: `service/crawl/fund/stock_crawl.go`**

<font color=red>@注意：这次爬取的网页数据是通过ajax加载，所以不能直接使用OnHtml抓取。</font>

```go
// 爬取信息
func (c *StockPercentageRowsCrawl) CrawlHtml(fundCode string) {
	collector := colly.NewCollector(colly.UserAgent(crawl.UserAgent), colly.Async(true))
	// 开启限速
	err := collector.Limit(&colly.LimitRule{
		DomainGlob:  "*fundf10.eastmoney.*",
		Delay:       500 * time.Millisecond,
		RandomDelay: 500 * time.Millisecond,
		Parallelism: 20,
	})
	collector.OnRequest(func(request *colly.Request) {
		fmt.Println("url:", request.URL)
	})
	// 处理返回的数据
	collector.OnResponse(func(response *colly.Response) {
		// 替换字符串
		compile := regexp.MustCompile(`var apidata=\{ content:"(.*)",arryear:`)
		matchResult := compile.FindAllStringSubmatch(string(response.Body), -1)
		if len(matchResult) == 0 {
			return
		}
		htmlString := matchResult[0][1]
		htmlString = strings.ReplaceAll(htmlString, "%", "")
		htmlString = strings.ReplaceAll(htmlString, ",", "")
		doc, err := goquery.NewDocumentFromReader(bytes.NewBuffer([]byte(htmlString)))
		if err != nil {
			return
		}
		docSelection := doc.Find("div[class='box']").First()
		e := &colly.HTMLElement{
			DOM: docSelection.Find("table"),
		}
		err = e.Unmarshal(c)
		if err != nil {
			global.GvaLogger.Error("爬虫解析失败", zap.String("error", err.Error()))
			return
		}
		// 过滤header
		if len(c.Rows) > 0 && c.Rows[0].StockCode == "" {
			c.Rows = c.Rows[1:]
		}
		// 获取持仓季度时间信息
		c.CutOffDate = docSelection.Find("h4 label").Eq(1).Find("font").Text()
		// 补充额外信息
		c.FundCode = fundCode
	})
 //topline=30 代表持仓前30的股票
	err = collector.Visit(fmt.Sprintf("https://fundf10.eastmoney.com/FundArchivesDatas.aspx?type=jjcc&code=%s&topline=30", fundCode))
	if err != nil {
		global.GvaLogger.Sugar().Errorf("CrawlHtml error:%s", err)
	}
	collector.Wait()
}
```

### 3.5 数据清洗(`ConvertEntity`)

**文件位置: `service/crawl/fund/stock_crawl.go`**

```go
func (c StockPercentageRowsCrawl) ConvertEntity() []entity.FundStock {
	var fundStocks []entity.FundStock
	if len(c.Rows) < 1 {
		return []entity.FundStock{}
	}
	for _, row := range c.Rows {
		item := entity.FundStock{
			FundCode:   c.FundCode,
			StockCode:  row.StockCode,
			StockName:  row.StockName,
			CutOffDate: c.CutOffDate,
		}
		// 提取交易所信息
		// 提取交易所信息
		compile := regexp.MustCompile(`com\/([a-zA-Z]+)\d+\.html`)
		stringSubMatch := compile.FindAllStringSubmatch(row.StockHref, -1)
		if stringSubMatch != nil {
			 item.StockExchange = strings.ToUpper(stringSubMatch[0][1])
		}
		// 字符串转浮点型
		item.Percentage, _ = strconv.ParseFloat(row.Percentage, 64)
		item.Quantity, _ = strconv.ParseFloat(row.Quantity, 64)
		item.Amount, _ = strconv.ParseFloat(row.Amount, 64)
		fundStocks = append(fundStocks, item)
	}
	return fundStocks
}
```

### 3.6 保存入库(`saveToDb`)

**文件位置：`crontab/fund_stock_cron.go`**

```go
// 保存入库
func saveToDb(dataChan chan [][]entity.FundStock) {
	// 声明基金持仓股票实体列表
	fundStockRows := []entity.FundStock{}
	// 声明股票实体列表
	stockRows := []entity.Stock{}
	// 声明股票实体列表
	checkExistKey := make(map[string]struct{}, perTaskTotal)
	// 遍历
	for fundStockGroup := range dataChan {
		for _, fundStockList := range fundStockGroup {
			for _, fundStock := range fundStockList {
				stockCode := fundStock.StockCode
				fundStockRows = append(fundStockRows, fundStock)
				// 判断是否已经存在
				if _, ok := checkExistKey[stockCode]; !ok {
					stockRows = append(stockRows, entity.Stock{
						Code:         fundStock.StockCode,
						Name:         fundStock.StockName,
						ExchangeCode: fundStock.StockExchange,
					})
					checkExistKey[stockCode] = struct{}{}
				}
			}
		}
	}
	var codeList []string
	for val := range fundCodeChannel {
		for _, c := range val {
			codeList = append(codeList, c)
		}
	}

	// 入库
	if save := global.GvaMysqlClient.Create(fundStockRows); save.Error != nil {
		global.GvaLogger.Sugar().Errorf("基金持仓入库失败:%s", save.Error)
	}
	// 批量更新
	if len(codeList) > 0 {
		if up := global.GvaMysqlClient.Model(&entity.FundBasis{}).Where("`code` in ?", codeList).
			Update("sync_stock", 1); up.Error != nil {
			global.GvaLogger.Sugar().Errorf("信息更新失败:%s", up.Error)
		}
	}
	if save := global.GvaMysqlClient.Create(stockRows); save.Error != nil {
		global.GvaLogger.Sugar().Errorf("股票信息入库失败:%s", save.Error)
	}
}
```

## 4. 添加定时任务

**文件位置：`initialize/cron.go`**

```go
// 添加Job任务
func addJob(c *cron.Cron) {
   ...
  // 爬取基金持仓信息信息(每天 20:30)
 	_, _ = c.AddJob("0 30 20 */1 * *", crontab.FundStockCron{})
}
```

## 5. 运行效果

![](https://go.liuqh.icu/img/20210907163940.png)

![](https://go.liuqh.icu/img/20210907163708.png)



![](https://go.liuqh.icu/img/20210907164350.png)



<font color=red size=4><b>关注公众号【<font color=blue>猿码记</font>】,回复【<font color=blue>基金</font>】获取源码地址。</b></font>


