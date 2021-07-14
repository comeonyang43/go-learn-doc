### 1. 介绍

`gorm`是一个使用`Go`语言编写的`ORM`框架。 它文档齐全，对开发者友好，支持主流数据库。[官方中文文档 https://gorm.io/zh_CN/docs/index.html](https://gorm.io/zh_CN/docs/index.html)

### 2. 安装

```go
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```

### 3. 连接MYSQL

`GORM` 官方支持的数据库类型有： `MySQL, PostgreSQL, SQlite, SQL Server`,这里只讲常见的`MYSQL`。其他类型数据库参见[官方文档](https://gorm.io/zh_CN/docs/connecting_to_the_database.html)。

#### 3.1 安装`mysql`依赖 

```bash
go get -u gorm.io/driver/mysql
```

#### 3.2 建立连接

##### a. 默认连接

```go
// 默认连接
func ConnectMysqlByDefault(host, port, user, pass, dbname string) (*gorm.DB, error) {
	// user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local
	dns := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		user, pass, host, port, dbname)
	return gorm.Open(mysql.Open(dns), &gorm.Config{})
}
```

##### b. 自定义配置连接

```go
// 自定义配置连接
func ConnectMysqlByCustom(host, port, user, pass, dbname string) (*gorm.DB, error) {
	// user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local
	dns := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		user, pass, host, port, dbname)
	return gorm.Open(mysql.New(mysql.Config{
		DSN:                       dns,
		DefaultStringSize:         256,   // string 类型字段的默认长度
		DisableDatetimePrecision:  true,  //禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
		DontSupportRenameIndex:    true,  //重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
		DontSupportRenameColumn:   true,  //用change重命名列，MySQL8之前的数据库和 MariaDB 不支持重命名列
		SkipInitializeWithVersion: false, // 根据当前 MySQL 版本自动配置
	}))
}
```

##### c. 基于`database/sql`连接

```go
// 基于database/sql连接，进行二次封装
func ConnectMysqlByDatabaseSql(host, port, user, pass, dbname string) (*gorm.DB, error) {
	dns := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		user, pass, host, port, dbname)
	db, err := sql.Open("mysql", dns)
	if err != nil {
		return nil, err
	}
	return gorm.Open(mysql.New(mysql.Config{Conn: db}))
}
```

[`gorm.Config`配置使用,参见官方文档: https://gorm.io/zh_CN/docs/gorm_config.html](https://gorm.io/zh_CN/docs/gorm_config.html)

### 4. 声明模型

#### 4.1 什么是模型

模型其实就是标准的结构体(`struct`),其属性和表结构的字段按照 **<font color=red>gorm约定</font>** 一一对应。约定如下:

- `gorm`使用`ID`作为主键,对应表中的`id`。
- 使用结构体名的 `蛇形复数` 作为表名。对于结构体 `User`，根据约定，其表名为 `users`
- 使用 `CreatedAt`、`UpdatedAt` 字段追踪创建、更新时间。

`gorm` 定义一个 `gorm.Model` 结构体，其包括字段 `ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt`,其他结构体直接匿名嵌套即可。

#### 4.2 字段标签

| 标签名          | 说明                                                         |
| :-------------- | :----------------------------------------------------------- |
| `column`        | 指定 db 列名                                                 |
| `type`          | 列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：`not null`、`size`, `autoIncrement`… 像 `varbinary(8)` 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：`MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT` |
| `size`          | 指定列大小，例如：`size:256`                                 |
| `primaryKey`    | 指定列为主键                                                 |
| `unique`        | 指定列为唯一                                                 |
| `default`       | 指定列的默认值                                               |
| `not null`      | 指定列为 NOT NULL                                            |
| `autoIncrement` | 指定列为自动增长                                             |
| `comment`       | 迁移时为字段添加注释                                         |
| `index`         | 根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 [索引](https://gorm.io/zh_CN/docs/indexes.html) 获取详情 |

[更多约定，请查看官方文档](https://gorm.io/zh_CN/docs/models.html)

#### 4.3 创建用户模型

按照`gorm`约定创建用户模型,便于后面使用。用户模型如下

```go
// 用户模型
type User struct {
	gorm.Model
	NickName     string         `gorm:"type:varchar(20);not null;default:'';comment:昵称"`
	Age          uint8          `gorm:"size:4;comment:年龄"`
	Phone        string         `gorm:"type:char(11);index:un_phone;comment:手机号"`
	MemberNumber string         `gorm:"type:varchar(20);index:un_phone;comment:会员编号"`
	Birthday     sql.NullString `gorm:"type:varchar(10);comment:生日"`
	ActivatedAt  sql.NullTime   `gorm:"comment:激活时间"`
}
```

### 5. 迁移

#### 5.1 根据模型生成表结构

根据上面创建的用户模型`User`，生成对应的表结构

**a. 代码**

```go
// 自动迁移schema,(根据结构体创建或者更新schema)
func GormAutoMigrate(host, port, use, pass, database string) error {
  // ConnectMysqlByDefault 代码参见上面的: 默认连接
	mysqlByDefault, err := ConnectMysqlByDefault(host, port, use, pass, database)
	if err != nil {
		return err
	}
	// 指定引擎和表备注
	err = mysqlByDefault.Set("gorm:table_options", "ENGINE=InnoDB COMMENT='用户表'").AutoMigrate(&User{})
	if err != nil {
		return err
	}
	return nil
}

// 测试迁移
func TestAutoMigrate(t *testing.T) {
	host := "127.0.0.1"
	use,pass,port,database := "root","root","3306","test"
	err := gorme.GormAutoMigrate(host, port, use, pass, database)
	if err != nil {
		t.Error(err)
	}
	fmt.Println("创建表结构完成!")
}
```

**b. 生成的表结构如下:**

```sql
CREATE TABLE `users` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT,
  `created_at` datetime(3) DEFAULT NULL,
  `updated_at` datetime(3) DEFAULT NULL,
  `deleted_at` datetime(3) DEFAULT NULL,
  `nick_name` varchar(20) COLLATE utf8mb4_bin NOT NULL DEFAULT '' COMMENT '昵称',
  `age` tinyint unsigned DEFAULT NULL COMMENT '年龄',
  `phone` char(11) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '手机号',
  `member_number` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '会员编号',
  `birthday` varchar(10) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '生日',
  `activated_at` datetime(3) DEFAULT NULL COMMENT '激活时间',
  PRIMARY KEY (`id`),
  KEY `idx_users_deleted_at` (`deleted_at`),
  KEY `un_phone` (`phone`,`member_number`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='用户表';
```

### 6.CRUD

#### 6.1 初始化客户端

```go
// 定义包内全局变量
var mysqlClient *gorm.DB
// 初始化mysql客户端
func init()  {
	// 连接客户端
	host := "127.0.0.1"
	user, pass, port, database := "root", "root", "3306", "test"
	databaseSql, _ := gorme.ConnectMysqlByDatabaseSql(host, port, user, pass, database)
	mysqlClient = databaseSql
}
```

#### 6.2 单条插入

```go
// 保存一条记录
func TestAddOne(t *testing.T) {
	// 初始化结构体
	userRow := gorme.User{
		NickName:     "李四",
		Age:          18,
		Phone:        "12340000",
		MemberNumber: "A0001",
		Birthday:     sql.NullString{String: "1991-03-04",Valid: true},
		ActivatedAt:  sql.NullTime{Time: time.Now(),Valid: true},
	}
	// 传入指针
	result := mysqlClient.Create(&userRow)
	fmt.Println(result)
	fmt.Println("id: ",userRow.ID)
	fmt.Println("插入记录错误: ",result.Error)
	fmt.Println("插入记录的条数: ",result.RowsAffected)
}
/*** 输出
=== RUN   TestAddOne
&{0xc000184240 <nil> 1 0xc0001a21c0 0}
id:  1
插入记录错误:  <nil>
插入记录的条数:  1
--- PASS: TestAddOne (0.04s)
PASS
*/
```

#### 6.3 批量插入

```go
// 批量插入
func TestBatchInsert(t *testing.T) {
	// 定义user 切片
	 userRows := []gorme.User{
		{NickName: "路人甲", Age: 20, Phone: "20000000", MemberNumber: "A0002"},
		{NickName: "路人乙", Age: 22, Phone: "30000000", MemberNumber: "A0003"},
		{NickName: "路人丙", Age: 24, Phone: "40000000", MemberNumber: "A0004"},
	}
	// 保存
	result := mysqlClient.Create(&userRows)
	fmt.Println("插入记录错误: ", result.Error)
	fmt.Println("插入记录的条数: ", result.RowsAffected)
	// 打印ID
	for _, row := range userRows {
		fmt.Println("插入ID: ", row.ID)
	}
}
/** 输出
=== RUN   TestBatchInsert
插入记录错误:  <nil>
插入记录的条数:  3
插入ID:  4
插入ID:  5
插入ID:  6
--- PASS: TestBatchInsert (0.02s)
PASS
*/
```

[更新插入方式，参见文档: https://gorm.io/zh_CN/docs/create.html](https://gorm.io/zh_CN/docs/create.html)

#### 6.4 查询

##### a. 单条记录: `First`、`Take`、`Last`

- `First`:  获取第一条记录（主键升序）
- `Take`: 获取一条记录，没有指定排序字段
- `Last`: 获取最后一条记录（主键降序）

```go
// 使用提供的First、Take、Last，查询单条记录
func TestGetOne(t *testing.T) {
	// 定义对应的结构体变量存储结果
	var firstUser gorme.User
	var taskUser gorme.User
	var lastUser gorme.User
	var result *gorm.DB
	// 获取第一条记录（主键升序） SELECT * FROM users ORDER BY id LIMIT 1;
	result = mysqlClient.First(&firstUser)
	fmt.Printf("First Result: %+v\n", result.RowsAffected)
	// 获取一条记录，没有指定排序字段 SELECT * FROM users LIMIT 1;
	result = mysqlClient.Take(&taskUser)
	fmt.Printf("Take Result: %+v\n", result.RowsAffected)
	// 获取最后一条记录（主键降序）SELECT * FROM users ORDER BY id DESC LIMIT 1;
	result = mysqlClient.Last(&lastUser)
	fmt.Printf("lastUser Result: %+v\n", result.RowsAffected)
}
```

<font color=red>@注意: 如果表里没有数据，会报错`result.Error返回:ErrRecordNotFound`</font>,可以使用下面语句检查错误。如果想避免`ErrRecordNotFound`错误，你可以使用`Find`,如:`db.Limit(1).Find(&user)`

```go
// 检查 ErrRecordNotFound 错误
errors.Is(result.Error, gorm.ErrRecordNotFound)
```

##### b. 多条记录: Find

```go
// 使用Find(默认查询的是检索全部)
func TestGetByFind(t *testing.T) {
	var userList []gorme.User
	// 指针查询字段
	result := mysqlClient.Select("id", "nick_name").Find(&userList)
	for _, user := range userList {
		fmt.Printf("id: %d nick_name: %s \n", user.ID,user.NickName)
	}
	fmt.Println("查询记录数: ", result.RowsAffected)
	fmt.Println("查询错误: ", result.Error)
}
/** 输出
=== RUN   TestGetByFind
id: 2 nick_name: 李四 
id: 3 nick_name: 张三 
id: 4 nick_name: 路人甲 
id: 5 nick_name: 路人乙 
id: 6 nick_name: 路人丙 
查询记录数:  5
查询错误:  <nil>
--- PASS: TestGetByFind (0.00s)
PASS
*/
```

##### c. 条件查询

**1. String条件**

```go
// 根据String条件查询
func TestGetByStringWhere(t *testing.T) {
	// 定义对应的结构体变量
	var user gorme.User
	var userList []gorme.User
	var result *gorm.DB
	// 字符串条件查询一条
	result = mysqlClient.Where("nick_name = ?", "张三").First(&user)
	fmt.Printf("Res1: %v err:%v \n", result.RowsAffected,result.Error)
	// 字符串条件查询多条
	result = mysqlClient.Where("nick_name <> ?", "张三").Find(&userList)
	fmt.Printf("Res2: %v err:%v \n", result.RowsAffected,result.Error)
	// 多个条件
	result = mysqlClient.Where("nick_name = ? and age >= ?", "张三", 18).First(&user)
	fmt.Printf("Res3: %v err:%v \n", result.RowsAffected,result.Error)
}
/** 输出
=== RUN   TestGetByStringWhere
Res1: 1 err:<nil> 
Res2: 4 err:<nil> 
Res3: 1 err:<nil> 
--- PASS: TestGetByStringWhere (0.02s)
PASS
*/
```
**2. Struct和Map条件**

```go
// 根据struct和map 条件查询结果
func TestGetByStructAndMapWhere(t *testing.T) {
	// 定义对应的结构体变量
	var user gorme.User
	var userList []gorme.User
	var result *gorm.DB
	// 结构体条件
	result = mysqlClient.Where(&gorme.User{NickName: "张三",Age: 18}).First(&user)
	fmt.Printf("结构体条件: %+v err:%v \n", result.RowsAffected,result.Error)
	// map条件
	result = mysqlClient.Where(map[string]interface{}{"age":18}).Find(&userList)
	fmt.Printf("map条件: %+v err:%v \n", result.RowsAffected,result.Error)
	// 主键切片
	result = mysqlClient.Where([]int64{2,3,4,5}).Find(&userList)
	fmt.Printf("主键切片: %+v err:%v \n", result.RowsAffected,result.Error)
}
/** 输出
=== RUN   TestGetByStructAndMapWhere
结构体条件: 1 err:<nil> 
map条件: 3 err:<nil> 
主键切片: 4 err:<nil> 
--- PASS: TestGetByStructAndMapWhere (0.01s)
PASS
*/
```

> <font color=red>@注意</font> 当使用结构作为条件查询时，`GORM` 只会查询非零值字段。这意味着如果您的字段值为 `0`、`''`、`false` 或其他 [零值](https://tour.golang.org/basics/12)，该字段不会被用于构建查询条件，例如：
>
> ```go
> db.Where(&User{NickName: "张三", Age: 0}).Find(&users)
> // SELECT * FROM users WHERE name = "张三";
> ```

[更多查询规则和方法，见: https://gorm.io/zh_CN/docs/query.html](https://gorm.io/zh_CN/docs/query.html)

#### 6.5 更新

##### a. 更新单个字段

```go
// 更新单个字段
func TestUpdateColumn(t *testing.T) {
	var result *gorm.DB
	// 字符串条件更新
	// UPDATE users SET nick_name='张三A', updated_at=当前时间 WHERE nick_name='张三;
	result = mysqlClient.Model(&gorme.User{}).Where("nick_name = ?", "张三").
		Update("nick_name", "张三A")
	fmt.Printf("条件更新: %+v err:%v \n", result.RowsAffected, result.Error)
	// 结构体条件更新
	// UPDATE users SET age=28, updated_at=当前时间 WHERE member_number='A0001;
	result = mysqlClient.Model(&gorme.User{}).Where(&gorme.User{MemberNumber: "A0001"}).Update("age",28)
	fmt.Printf("结构体条件更新: %+v err:%v \n", result.RowsAffected, result.Error)
}
/** 输出
=== RUN   TestUpdateColumn
条件更新: 1 err:<nil> 
结构体条件更新: 1 err:<nil> 
--- PASS: TestUpdateColumn (0.06s)
PASS
*/
```

##### b. 更新多个字段

```go
// 更新多个字段
func TestUpdateMultipleColumn(t *testing.T) {
	var result *gorm.DB
	// 使用map
	updateMap := map[string]interface{}{
		"age":      32,
		"birthday": "1991-01-05",
	}
	// UPDATE users SET age=32,birthday='1991-01-05',updated_at=当前时间 WHERE id=6;
	result = mysqlClient.Model(&gorme.User{}).Where("id = ?", 6).Updates(updateMap)
	fmt.Printf("使用map结构更新: %+v err:%v \n", result.RowsAffected, result.Error)
	// 使用结构体(不使用Select)
	updateUser := gorme.User{
		Birthday: sql.NullString{String: "1993-10-10", Valid: true},
		Age:      0,
	}
	// @注意这里的age=0不会更新到MySQL
	// UPDATE users SET birthday='1993-09-09',updated_at=当前时间 WHERE id=5;
	result = mysqlClient.Model(&gorme.User{}).Where("id = ?", 5).Updates(updateUser)
	fmt.Printf("使用struct结构更新: %+v err:%v \n", result.RowsAffected, result.Error)
	// 使用结构体(使用Select)
	updateUser2 := gorme.User{
		Birthday: sql.NullString{String: "1993-09-09", Valid: true},
		Age:      0,
	}
	// UPDATE users SET birthday='1993-09-09',age=0,updated_at=当前时间 WHERE id=4;
	result = mysqlClient.Model(&gorme.User{}).
		Select("birthday", "age"). //指定要更新的字段
		Where("id = ?", 4).Updates(updateUser2)
	fmt.Printf("使用struct结构更新2: %+v err:%v \n", result.RowsAffected, result.Error)
}
```

#### 6.6 删除

##### a. 软删除

```go
// 删除数据
func TestSoftDel(t *testing.T) {
	var result *gorm.DB
	// 根据主键，删除一条记录
	result = mysqlClient.Delete(&gorme.User{},1)
	fmt.Printf("根据主键删除一条: %+v err:%v \n", result.RowsAffected, result.Error)
	// 根据主键切片，删除多条记录
	result = mysqlClient.Delete(&gorme.User{},[]int64{2,3})
	fmt.Printf("根据主键切片删除多条: %+v err:%v \n", result.RowsAffected, result.Error)
	// 根据条件删除
	result = mysqlClient.Where("age = ?",0).Delete(&gorme.User{})
	fmt.Printf("根据条件删除: %+v err:%v \n", result.RowsAffected, result.Error)
}
```

> <font color=red>执行上面代码后，发现表里面的记录依然存在，而`DeletedAt` 置为当前时间。原因是`user`模型中包含了一个 `gorm.DeletedAt` 字段，它将自动获得软删除的能力！</font>

##### b. 永久删除

可以使用 `Unscoped` 永久删除匹配的记录。

```go
// 删除数据(硬删除)
func TestStrongDel(t *testing.T) {
	var result *gorm.DB
	result = mysqlClient.Unscoped().Delete(&gorme.User{}, 1)
	fmt.Printf("硬删除: %+v err:%v \n", result.RowsAffected, result.Error)
}
```

#### 6.7 事务

##### a. 使用`Transaction`

```go
// 事务使用(Transaction)
func TestTransaction(t *testing.T) {
	err := mysqlClient.Transaction(func(tx *gorm.DB) error {
		//在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
		// 创建用户记录
		user := gorme.User{NickName: "老王", Age: 48}
		if err := tx.Create(&user).Error; err != nil {
			// 回滚事务
			return err
		}
		// 创建用户地址
		userAddress := gorme.UserAddress{Uid: user.ID, Province: "北京", City: "北京", Area: "海淀区"}
		if err := tx.Create(&userAddress).Error; err != nil {
			// 回滚事务
			return err
		}
		return nil
	})
	if err != nil {
		t.Error(err)
	}
	fmt.Println("执行完成")
}
```

##### b. 手动事务

```go
// 手动事务
func TestUseManualTx(t *testing.T) {
    // 用户表
	user := gorme.User{NickName: "小丽", Age: 19}
	// 开启事务
	tx := mysqlClient.Begin()
	// 添加用户
	if err := tx.Create(&user).Error; err != nil {
		// 遇到错误时回滚事务
		fmt.Println("添加用户失败: ",err)
		tx.Rollback()
	}
	// 用户地址表
	userAddress := gorme.UserAddress{Uid: user.ID, Province: "北京", City: "北京", Area: "昌平区"}
	// 添加用户地址
	if err := tx.Create(&userAddress).Error; err != nil {
		// 遇到错误时回滚事务
		fmt.Println("添加用户地址失败: ",err)
		tx.Rollback()
	}
	// 提交事务
	tx.Commit()
	fmt.Println("执行完成")
}
```

<font color=red>@注意: 在实践中发现，执行时虽然事务失败，但是主键依然递增。具体看下图效果:</font>

![](https://gitee.com/QingHui/picGo-img-bed/raw/master/img/20210714184151.png)

