## 1 介绍
sql.DB不是一个连接，它是数据库的抽象接口。它可以根据driver打开关闭数据库连接，管理连接池。正在使用的连接被标记为繁忙，用完后回到连接池等待下次使用。所以，如果你没有把连接释放回连接池，会导致过多连接使系统资源耗尽。
## 2 导入driver

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)
```
## 3 连接DB
### 3.1 sql.Open()
```go
package main

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "log"
)

func main() {
    db, err := sql.Open("mysql", "root:Password01!@tcp(127.0.1:3306)/dbaas_check")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    log.Println(db)  //打印连接没啥用
}
[root@localhost sql]# go run sql1.go
2020/04/06 17:16:45 &{0 0xc000092018 0 {0 0} [] map[] 0 0 0xc0000680c0 0xc000094120 false map[] map[] 0 0 0 <nil> 0 0 0 0x47f530}

```

 1. sql.Open的第一个参数是驱动名称。这是驱动用来注册`database/sql`的字符串，并且通常与包名相同以避免混淆。例如，它是`github.com/go-sql-driver/mysql`的MySql驱动(作者:jmhodges)。某些驱动不遵守公约的名称，例如`github.com/mattn/go-sqlite3`的sqlite3(作者:matte)和`github.com/lib/pq`的`postgres`(作者:mjibson)。
 2. 第二个参数是一个驱动特定的语法，它告诉驱动如何访问底层数据存储。在本例中，我们将连接本地的MySql服务器实例中的“hello”数据库。
 3. 你应该(几乎)总是检查并处理从所有database/sql操作返回的错误。有一些特殊情况，我们稍后将讨论这样做事没有意义的。
 4. 如果sql.DB不应该超出该函数的作用范围，则延迟函数defer db.Close()来关闭数据库连接。

### sql.Open()与NewConfig()（待详解）

```go
func newPool() *sql.DB {
	cfg := mysql.NewConfig()
	cfg.User = "root"
	cfg.Passwd = "xxxxxx"
	cfg.Net = "tcp"
	cfg.Addr = "127.0.0.1:3306"
	cfg.DBName = "mydb"
	dsn := cfg.FormatDSN()

	db, err := sql.Open("mysql", dsn)
	if err != nil {
		log.Fatal(err)
	}
	if err := db.Ping(); err != nil {
		log.Fatal(err)
	}
	return db
}

var pool = newPool()
```

 1. sql.DB 表示一个连接池
 2. sql.Open 的第一个参数是驱动名称，这里是 "mysql"这个名称是在 mysql 包初始化时注册的，代码`github.com/go-sql-driver/mysql/driver.go`
 3. sql.Open 的第二个参数是数据源名称，这里通过 mysql.Config 结构来配置，然后调用 FormatDSN
    方法得出数据源名称为："root:xxxxxx@tcp(127.0.0.1:3306)/mydb"
 4. db.Ping() 用来检查网络连通性以及用户密码是否正确
 5. pool 是一个包级变量，生命周期持续整个应用，所以不需要关闭
 6. sql.DB 设计上就是作为长期存活的对象来使用的。


## 4 db.Ping()检查是否可以建立网络连接并登陆

```go
package main

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "log"
)

func main() {
    db, err := sql.Open("mysql", "root:Password01!@tcp(127.0.0.1:3306)/api_server")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    if err := db.Ping(); err != nil {
	log.Fatal(err)
    }
    log.Println("ping successfully")
}
```

```go
[root@localhost sql]# go run sql2.go
2020/04/06 17:49:22 ping successfully
```


sql.DB的设计就是用来作为长连接使用的。不要频繁Open, Close。比较好的做法是，为每个不同的datastore建一个DB对象，保持这些对象Open。如果需要短连接，那么把DB作为参数传入function，而不要在function中Open, Close。
## 5 查询数据
### 5.1 db.Query()多行查询，
如果方法包含Query，那么这个方法是用于查询并返回rows的。其他情况应该用Exec()。

```go

package main

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "log"
)


func dbcheck() error  {

    db, err := sql.Open("mysql", "root:Password01!@tcp(192.168.1.190:3306)/api_server")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    var (
       id string
       name string
    )

    rows, err := db.Query("select id, serv_name  from tbl_serv limit 1;")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    for rows.Next() {
        err := rows.Scan(&id, &name)
        if err != nil {
           log.Fatal(err)
           return err
        }
        log.Println(id, name)
    }
    err = rows.Err()
    if err != nil {
        log.Fatal(err)
    }
   return err 
}

func main() {
    err := dbcheck()
    if err != nil {
       log.Println(err)
	}

}
 
[root@localhost sql]# go run sql3.go
2020/04/06 19:03:00 3ebd186a830c4737abe57e1c33aaf091 zxtest001

```
db.Query()表示向数据库发送一个query
rows.Next()遍历
rows.Scan()赋值

defer db.Close()来关闭数据库连接。
遍历rows使用rows.Next()， 把遍历到的数据存入变量使用rows.Scan(), 遍历完成后检查error。有几点需要注意：

检查遍历是否有error
结果集(rows)未关闭前，底层的连接处于繁忙状态。当遍历读到最后一条记录时，会发生一个内部EOF错误，自动调用rows.Close()，但是如果提前退出循环，rows不会关闭，连接不会回到连接池中，连接也不会关闭。所以手动关闭非常重要。rows.Close()可以多次调用，是无害操作。

### 5.2 db.QueryRow()单行查询
err在Scan后才产生，所以可以如下写：
id 为3ebd186a830c4737abe57e1c33aaf091的值

```bash
package main

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "log"
)

func main() {
    db, err := sql.Open("mysql", "root:Password01!@tcp(127.0.0.1:3306)/api_server")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    var name string
    err = db.QueryRow("select serv_name from tbl_serv where id = ?", "3ebd186a830c4737abe57e1c33aaf091").Scan(&name)
    if err != nil {
       log.Fatal(err)
    }
    log.Println("ping successfully" + name)
}
```

```go
[root@localhost sql]# go run sql4.go
2020/04/06 19:11:53 ping successfullyzxtest001
```
## 6. 执行数据
### 6.1 Exec()插入

```go
package main

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "log"
)


func dbexec() error  {

    db, err := sql.Open("mysql", "user1:ZongXun123.@tcp(192.168.1.94:31008)/data")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    ret, err := db.Exec(`INSERT INTO runoob_tbl  (runoob_title, runoob_author, submission_date) VALUES("learn golang", "test", NOW());`)
    if err != nil {
	    log.Fatal("insert data error: %v\n", err)
	    return err
    }
    if LastInsertId, err := ret.LastInsertId(); nil == err {
	    log.Println("LastInsertId:", LastInsertId)
    }
    if RowsAffected, err := ret.RowsAffected(); nil == err {
	    log.Println("RowsAffected:", RowsAffected)
    }
    return err 
}

func main() {
    err := dbexec()
    if err != nil {
       log.Println(err)
	}

}
```

```go
[root@localhost sql]# go run sql5.go
2020/04/07 00:27:40 LastInsertId: 11
2020/04/07 00:27:40 RowsAffected: 1

mysql> select * from runoob_tbl;
+-----------+--------------+---------------+-----------------+
| runoob_id | runoob_title | runoob_author | submission_date |
+-----------+--------------+---------------+-----------------+
|         1 | learn PHP    | test          | 2020-04-07      |
|        11 | learn golang | test          | 2020-04-07      |
+-----------+--------------+---------------+-----------------+

```
### 6.1 Exec()与db.Prepare()插入

```go
package main

import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
    "log"
)

func dbexec() error  {

    db, err := sql.Open("mysql", "user1:ZongXun123.@tcp(192.168.1.94:31008)/data")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    ret, err := db.Prepare("INSERT INTO runoob_tbl  (runoob_title, runoob_author, submission_date) VALUES(?, ?, NOW());")
    if err != nil {
	    log.Fatal("insert data error: %v\n", err)
	    return err
    }
    res, err := ret.Exec("learn python", "test3")
    if err != nil {
       log.Fatal(err)
    }
    lastId, err := res.LastInsertId()
    if err != nil {
	log.Fatal(err)
    }
    rowCnt, err := res.RowsAffected()
    if err != nil {
	log.Fatal(err)
    }
    log.Printf("ID = %d, affected = %d\n", lastId, rowCnt)
    return err 
}

func main() {
    err := dbexec()
    if err != nil {
       log.Println(err)
	}

} 
```

```go
[root@localhost sql]# go run sql6.go
2020/04/07 00:35:54 ID = 31, affected = 1
mysql> select * from runoob_tbl;
+-----------+--------------+---------------+-----------------+
| runoob_id | runoob_title | runoob_author | submission_date |
+-----------+--------------+---------------+-----------------+
|         1 | learn PHP    | test          | 2020-04-07      |
|        11 | learn golang | test          | 2020-04-07      |
|        21 | learn python | test3         | 2020-04-07      |
|        31 | learn python | test3         | 2020-04-07      |
```

