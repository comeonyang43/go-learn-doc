## 1. 介绍

数据模型指的是和表结构一一对应的结构体，通过编写结构体可通过迁移直接创建表结构,后续作为对表的增删改查的参数。

## 2. 用户模型

### 2.1 用户表

**文件：`model/entity/user.go`**

```go
// 用户表
type User struct {
	global.BaseModel
	NickName string   `json:"nickName" gorm:"type:varchar(20);not null;default:'';comment:昵称"`
	Phone    string   `json:"phone" gorm:"type:char(11);unique:un_phone;comment:手机号"`
	Password string   `json:"-" gorm:"type:varchar(40);comment:密码"`
	Status   int      `json:"status" gorm:"size:4;default:1;comment:状态 1:正常 2:白名单 3:黑名单"`
	UserInfo UserInfo `json:"userInfo" gorm:"-"`
	Token    string   `json:"token" gorm:"-"`
}
```

### 2.2 用户信息表

**文件：`model/entity/user.go`**

```go
// 用户信息表
type UserInfo struct {
	global.BaseModel
	Uid      uint   `json:"uid" gorm:"comment:用户id"`
	Birthday string `json:"birthday" gorm:"type:varchar(10);comment:生日"`
	Address  string `json:"address" gorm:"type:text;comment:地址"`
}
```

## 3. 基金模型

### 3.1 基金基础表

**文件：`model/entity/fund.go`**

```go
// 基金基础信息表
type FundBasis struct {
	global.BaseModel
	Code            string  `json:"code" gorm:"type:char(6);unique:un_code;comment:基金代码"`
	FullName        string  `json:"fullName" gorm:"type:varchar(50);comment:基金全称"`
	ShortName       string  `json:"shortName" gorm:"type:varchar(50);comment:基金简称"`
	Type            string  `json:"type" gorm:"type:varchar(50);comment:基金类型"`
	Company         string  `json:"company" gorm:"type:varchar(50);comment:基金公司"`
	ReleaseDate     string  `json:"releaseDate" gorm:"type:varchar(12);comment:发布时间"`
	EstablishDate   string  `json:"establishDate" gorm:"type:varchar(12);comment:成立时间"`
	EstablishShares float64 `json:"establishShares" gorm:"type:decimal(12,4);comment:成立时规模(单位:亿份)"`
	ManageFeeRate   float64 `json:"manageFeeRate" gorm:"type:decimal(4,2);comment:管理费率(百分比)"`
	CustodyFeeRate  float64 `json:"custodyFeeRate" gorm:"type:decimal(4,2);comment:托管费率(百分比)"`
	SaleFeeRate     float64 `json:"saleFeeRate" gorm:"type:decimal(4,2);comment:销售服务费率(百分比)"`
	Benchmark       string `json:"benchmark" gorm:"type:varchar(255);comment:业绩比较基准"`
}
```

### 3.2 基金持仓表

**文件：`model/entity/fund.go`**

```go
// 基金股票持仓
type FundStock struct {
	global.BaseModel
	FundCode   string  `json:"fundCode" gorm:"type:varchar(10);index;comment:基金code"`
	StockCode  string  `json:"stockCode" gorm:"type:varchar(10);index;comment:股票code"`
	Percentage float64 `json:"percentage" gorm:"type:decimal(4,2);comment:持仓占比(百分比)"`
	Quantity   float64 `json:"quantity" gorm:"type:decimal(5,2);comment:持股数(万股)"`
	Amount     float64 `json:"amount" gorm:"type:decimal(5,2);comment:持股市值(万元)"`
	CutOffDate string  `json:"cutOffDate" gorm:"type:char(10);comment:截止时间"`
}
```

### 3.3 基金每日排行表

**文件：`model/entity/fund.go`**

```go
// FundDayTop 基金每日排行
type FundDayTop struct {
	global.BaseModel
	FundCode         string  `json:"fundCode" gorm:"type:varchar(10);index;comment:基金code"`
	FundName         string  `json:"fundName" gorm:"type:varchar(10);index;comment:基金名称"`
	TopDate          string  `json:"topDate" gorm:"type:varchar(12);index;comment:排名日期"`
	NetWorth         float64 `json:"netWorth" gorm:"type:decimal(10,4);comment:单位净值"`
	TotalWorth       float64 `json:"totalWorth" gorm:"type:decimal(10,4);comment:累计净值"`
	DayChange        float64 `json:"dayChange" gorm:"type:decimal(10,4);comment:日增长率"`
	WeekChange       float64 `json:"weekChange" gorm:"type:decimal(10,4);comment:近一周"`
	MouthChange      float64 `json:"mouthChange" gorm:"type:decimal(10,4);comment:近一月"`
	ThreeMouthChange float64 `json:"threeMouthChange" gorm:"type:decimal(10,4);comment:近3个月"`
	SixMouthChange   float64 `json:"sixMouthChange" gorm:"type:decimal(10,4);comment:近6个月"`
	YearChange       float64 `json:"yearChange" gorm:"type:decimal(10,4);comment:近1年"`
	TwoYearChange    float64 `json:"twoYearChange" gorm:"type:decimal(10,4);comment:近2年"`
	ThreeYearChange  float64 `json:"threeYearChange" gorm:"type:decimal(10,4);comment:近3年"`
	CurrentChange    float64 `json:"CurrentChange" gorm:"type:decimal(10,4);comment:今年来"`
	CreateChange     float64 `json:"createChange" gorm:"type:decimal(10,4);comment:成立以来"`
}
```

## 4. 股票模型

### 4.1 股票基础表

**文件：`model/entity/stock.go`**

```go
// 股票基础信息
type Stock struct {
	global.BaseModel
	Code           string  `json:"code" gorm:"type:varchar(10);unique:un_code;comment:股票代码"`
	Name           string  `json:"name" gorm:"type:varchar(50);comment:股票名称"`
	Industry       string  `json:"industry" gorm:"type:varchar(20);comment:所属行业"`
	ExchangeCode   string  `json:"exchangeCode" gorm:"type:varchar(5);comment:所属交易所，SZ:深圳 SH:上海 HK:港股"`
	Tag            string  `json:"tag" gorm:"type:text;comment:所属概念"`
	SetUpDate      string  `json:"setUpDate" gorm:"type:char(10);comment:公司成立时间"`
	MarketDate     string  `json:"marketDate" gorm:"type:char(10);comment:上市时间"`
	MarketPE       float64 `json:"marketPE" gorm:"type:decimal(10,2);comment:发行市盈率"`
	MarketQuantity float64 `json:"marketQuantity" gorm:"type:decimal(10,2);comment:发行量(万股)"`
	MarketPrice    float64 `json:"marketPrice" gorm:"type:decimal(10,2);comment:发行价格"`
	MarketAmount   float64 `json:"marketAmount" gorm:"type:decimal(10,2);comment:实际募资(亿)"`
	Company        string  `json:"company" gorm:"type:varchar(255);comment:公司名称"`
	Employees      uint    `json:"employees" gorm:"comment:公司员工"`
	MoneyUnit      string  `json:"moneyUnit" gorm:"type:varchar(10);comment:货币单位"`
}
```

### 4.2 股票行情表

**文件：`model/entity/stock.go`**

```go
// 股票行情
type StockQuotes struct {
	global.BaseModel
	StockCode         uint    `json:"stockCode" gorm:"index;comment:股票ID"`
	RecordDate        string  `json:"recordDate" gorm:"type:char(10);comment:记录时间"`
	TotalQuantity     float64 `json:"totalQuantity" gorm:"type:decimal(10,2);comment:总股本(万股)"`
	TotalAmount       float64 `json:"totalAmount" gorm:"type:decimal(10,2);comment:总市值(亿元)"`
	CirculateQuantity float64 `json:"circulateQuantity" gorm:"type:decimal(10,2);comment:流通股本(万股)"`
	CirculateAmount   float64 `json:"circulateAmount" gorm:"type:decimal(10,2);comment:流通市值(亿)"`
	TodayOpenPrice    float64 `json:"todayOpenPrice" gorm:"type:decimal(10,2);comment:今日开盘价格"`
	TodayMaxPrice     float64 `json:"todayMaxPrice" gorm:"type:decimal(10,2);comment:今日最高价格"`
	TodayPrice        float64 `json:"todayPrice" gorm:"type:decimal(10,2);comment:今日收盘价格"`
	LastPrice         float64 `json:"lastPrice" gorm:"type:decimal(10,2);comment:昨日收盘价格"`
	MoneyUnit         string  `json:"moneyUnit" gorm:"type:varchar(10);comment:货币单位"`
	ChangeRatio       float64 `json:"changeRatio" gorm:"type:decimal(5,2);comment:换手率(百分比)"`
	DynamicPE         float64 `json:"dynamicPE" gorm:"type:decimal(10,2);comment:市盈率(动)"`
	StaticPE          float64 `json:"staticPE" gorm:"type:decimal(10,2);comment:市盈率(静)"`
	PeTtm             float64 `json:"peTTM" gorm:"type:decimal(10,2);comment:市盈率(TTM)"`
	PB                float64 `json:"PB" gorm:"type:decimal(10,2);comment:市净率"`
	EarningsPerShare  float64 `json:"earningsPerShare" gorm:"type:decimal(10,2);comment:每股收益"`
	NetAssetsPerShare float64 `json:"netAssetsPerShare" gorm:"type:decimal(10,2);comment:每股净资产"`
	Dividends         float64 `json:"dividends" gorm:"type:decimal(10,2);comment:股息(TTM)"`
	DividendsRatio    float64 `json:"dividendsRatio" gorm:"type:decimal(5,2);comment:股息率(TTM)"`
}
```

## 5. 编写迁移代码

### 5.1 添加对应的方法

**文件：`model/migrate/migrate.go`**

```go
// 设置表信息
func setTableOption(tableComment string) *gorm.DB {
	value := fmt.Sprintf("ENGINE=InnoDB COMMENT='%s'", tableComment)
	return global.GvaMysqlClient.Set("gorm:table_options", value)
}

// 用户相关表
func userMigrate() {
	// 用户账号表
	_ = setTableOption("用户表").AutoMigrate(&entity.User{})
	// 用户信息表
	_ = setTableOption("用户信息表").AutoMigrate(&entity.UserInfo{})
}
// 基金表
func fundMigrate()  {
	// 基金基础表
	_ = setTableOption("基金表").AutoMigrate(&entity.FundBasis{})
	// 基金持仓表
	_ = setTableOption("基金持仓表").AutoMigrate(&entity.FundStock{})
  // 基金每日排行  
  _ = setTableOption("基金每日排行").AutoMigrate(&entity.FundDayTop{})  
}
// 股票相关表
func stockMigrate()  {
	// 股票基础表
	_ = setTableOption("股票表").AutoMigrate(&entity.Stock{})
	// 股票行情表
	_ = setTableOption("股票行情表").AutoMigrate(&entity.StockQuotes{})
}
```

### 5.2 修改迁移入口函数

**文件：`model/migrate/migrate.go`**

```go
// 数据表迁移
func AutoMigrate() {
	// 用户相关表
	userMigrate()
	// 基金相关表
	fundMigrate()
	// 股票相关表
	stockMigrate()
}
```

## 6. 启动项目

### 6.1 修改配置

**文件：`./config.yaml`**

```yaml
...
mysql:
  enable: true
  ....
  database: fund_analye_system # 库需要手动创建
  autoMigrate: true # 开启时，每次服务启动都会根据实体创建/更新表结构
  ...
  gorm: # 
    tablePrefix: "fas_" #设置表前缀
    ...
```

### 6.2 自动创建表

当在配置中,打开`autoMigrate=true`时，成功启动项目后，会创建以下表信息。

![i](/img/20210831154957.png)

## 7. 表关系图

![](/img/20210831155140.png)


