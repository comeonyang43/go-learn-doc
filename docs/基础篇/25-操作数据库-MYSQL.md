### 1.介绍

`Go`官方提供了`database`包，`database`包下有`sql/driver`。该包用来定义操作数据库的接口，这保证了无论使用哪种数据库，操作方式都是相同的。但`Go`官方并没有提供连接数据库的`driver`，如果要操作数据库，还需要第三方的`driver`包。

### 2.下载安装驱动

`go-sql-driver`驱动源码地址: https://github.com/go-sql-driver/mysql

#### 2.1 安装驱动

```go
go get -u github.com/go-sql-driver/mysql
```

### 3.匿名导入

<font color=blue>通常来说，导入包后就能调用该包中的数据和方法。但是对于数据库操作来说，不应该直接使用导入驱动包所提供的方法，而应该使用 `sql.DB`对象所提供的统一的方法。因此在导入`MySQL`驱动时，使用了匿名导入包的方式。</font>

> 匿名导入包: 只导入包但是不使用包内的类型和数据，使用匿名的方式: 在包路径前添加下画线`_`

```go
import (
	_ "github.com/go-sql-driver/mysql"
)
```

在导入一个数据库驱动后，该驱动会自行初始化并注册到`Go`的`database/sql上`下文中，这样就可以通过 `database/sql `包所提供的方法来访问数据库了。

### 4.连接数据库

#### 4.1 连接方法
使用`sql`包中的`Open()`函数来连接数据库。

```go
Open(driverName, dataSourceName string) (*DB, error)
```

- `driverName`: 使用的驱动名,如`mysql`。(注册到 `database/sql`时所使用的名字)
- `dataSourceName`:数据库连接信息,格式:[`用户名:密码@tcp(IP:port)/数据库?charset=utf8`],例如:`root:123456@tcp(127.0.0.1:3306)/test?charset=utf8`

#### 4.2 sql.DB作用

- `sql.Open()`返回的`sql.DB`对象是`Goroutine`并发安全的。
- `sql.DB` 通过数据库驱动为开发者提供管理底层数据库连接的打开和关闭操作。
- `sql.DB` 帮助开发者管理数据库连接池。正在使用的连接被标记为繁忙，用完后回到连接池等待下次使用。所以，<font color=red>如果开发者没有把连接释放回连接池，会导致过多连接使系统资源耗尽。</font>

#### 4.3 sql.DB设计目标

`sql.DB`的设计目标就是作为长连接（一次连接多次数据交互）使用，不宜频繁开关。比较好的做法是，为每个不同的`datastore`建一个`DB`对象，保持这些对象打开。如果需要短连接（一次连接一次数据交互），就把`DB`作为参数传入`function`，而不要在`function`中开关。

### 5.写操作(增、删、改)

#### 5.1 执行步骤

1. 先使用`预编译语句（PreparedStatement）`来拼接`sql`。
2. 然后调用`db.Exec()`执行`SQL`，返回执行结果

#### 5.2 代码示例

```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"time"
)
func main() {
	// 连接数据库
	open, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test?charset=utf8")
	checkError(err)
  //插入数据
	//add(open)
	// 更新数据
	//update(open)
	// 删除数据
	del(open)
}
//插入数据
func add(open *sql.DB)  {
	//插入数据
	prepare, err := open.Prepare("insert user set username=?,password=?,mobile=?,createtime=?")
	checkError(err)
	exec, err := prepare.Exec("李四", "123456", "17600000000", time.Now().Unix())
	checkError(err)
	id, err := exec.LastInsertId()
	checkError(err)
	fmt.Printf("插入数据ID: %d \n",id)
}
// 更新
func update(open *sql.DB)  {
	prepare, err := open.Prepare("update user set username=? where id =?")
	checkError(err)
	exec, err := prepare.Exec("王五", "18")
	checkError(err)
	rows, err := exec.RowsAffected()
	checkError(err)
	fmt.Printf("更新数据成功,影响条数 %d \n",rows)
}
// 删除数据
func del(open *sql.DB)  {
	prepare, err := open.Prepare("delete from user  where id =?")
	checkError(err)
	exec, err := prepare.Exec( "8")
	checkError(err)
	rows, err := exec.RowsAffected()
	checkError(err)
	fmt.Printf("删除数据成功,影响条数 %d \n",rows)
}
//检测错误
func checkError(err error)  {
	if err != nil {
		panic("操作失败:"+err.Error())
	}
}
```

### 6. 读操作(查询)

#### 6.1 执行步骤

##### 1. 查询多条步骤

1. 调用`db.Query()`方法执行`SQL`语句，返回一个`Rows`查询结果。
2. 将`rows.Next()`方法的返回值作为`for`循环的条件，迭代查询数据。
3. 在循环中，通过 `rows.Scan()`方法读取每一行数据。
4. 调用`db.Close()`关闭查询。

##### 2.查询单条步骤

1. 调用`db.QueryRow()`方法执行`SQL`语句，返回一个`Row`查询结果。
2. 然后调用`row.Scan()`读取数据。

#### 6.2 代码示例

```go
package main
import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)
func main() {
	// 连接数据库
	db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/nsbd_app?charset=utf8")
	checkError(err)
	//查询多条数据
	rows := queryRows(db)
	fmt.Printf("多条返回: \n%+v\n",rows)
	// 查询单条数据
	row := queryRow(db)
	fmt.Printf("单条返回: \n%+v\n",row)
}

// 创建表的映射对象
type User struct {
	Uid        int
	UserName   string
	CreateTime int
	Birthday   sql.NullString //有的值可能为NULL
}

//查询多条数据
func queryRows(db *sql.DB) []User {
	stmt, err := db.Prepare("select id,username,createtime,birthday from nsbd_user where id < ?")
	checkError(err)
	rows, err := stmt.Query(30)
	// 延迟关闭
	defer rows.Close()
	checkError(err)
	user := new(User)
	//users := make([]User,5)
	var users []User
	for rows.Next() {
		// rows.Scan()方法的参数顺序很重要，必须和查询结果的column相对应（数量和顺序都需要一致）
		err := rows.Scan(&user.Uid, &user.UserName, &user.CreateTime, &user.Birthday)
		checkError(err)
		users = append(users, *user)
	}
	return users
}
// 查询单条数据
func queryRow(db *sql.DB) User {
	stmt, err := db.Prepare("select id,username,createtime,birthday from nsbd_user where id = ?")
	checkError(err)
	user := new(User)
	err = stmt.QueryRow(4).Scan(&user.Uid, &user.UserName, &user.CreateTime, &user.Birthday)
	checkError(err)
	return *user
}
//检测错误
func checkError(err error) {
	if err != nil {
		panic("操作失败:" + err.Error())
	}
}
```

**输出:**

```go
多条返回: 
[{Uid:1 UserName:admin CreateTime:0 Birthday:{String:2017-04-15 Valid:true}} {Uid:2 UserName:u2 CreateTime:1605858072 Birthday:{String:1993-02-14 Valid:true}} {Uid:3 UserName:u3 CreateTime:1606289644 Birthday:{String:1991-05-31 Valid:true}} {Uid:4 UserName:u4 CreateTime:1610521164 Birthday:{String:1989-11-24 Valid:true}} {Uid:5 UserName:u5 CreateTime:1610588359 Birthday:{String: Valid:false}}]
单条返回: 
{Uid:4 UserName:u4 CreateTime:1610521164 Birthday:{String:1989-11-24 Valid:true}}
```
#### 6.3 注意事项

- `rows.Scan()`方法的参数顺序很重要，必须和查询结果的`column`相对应(数量和顺序都需要一致)；
- `Go`是强类型语言，在查询数据时先定义数据类型，针对字段值为`NULL`时，数据类型应定义为:`sql.NullString、sql.NullInt64`等,并可以通过`Valid`值来判断查询到的值是赋值状态还是未赋值状态。
- 每次`db.Query()`操作后，都建议调用`rows.Close()`。`rows.Close()`操作是幂等操作，即便对已关闭的`rows`再执行`close()`也没关系。

#### 6.4 为什么查询后要关闭连接？

因为`db.Query()`会从数据库连接池中获取一个连接，这个底层连接在结果集（`rows`）未关闭前会被标记为处于繁忙状态。当遍历读到最后一条记录时，会发生一个内部`EOF`错误，自动调用`rows.Close()`。但如果出现异常，提前退出循环，`rows`不会关闭，连接不会回到连接池中，连接也不会关闭，则此连接会一直被占用。因此通常使用`defer rows.Close()`来确保数据库连接可以正确放回到连接池中。