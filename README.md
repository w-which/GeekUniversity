# GeekUniversity
极客大学作业
1. 我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

dao层不应该把异常处理掉，因为事务的边界不在dao层上，而是在sevice层上，假如一个业务逻辑调用了两个dao层的方法（A、B）组成一个事务体，若A正常执行了，而B执行时却发生了异常，这时A的执行就需要回滚，若将B中的异常捕获了，那根本就不知道B是正常执行而是异常执行了，也无法处理事务了。所以需要向上抛。

代码：

package main

import (
	"fmt"

	"github.com/jmoiron/sqlx"
	"github.com/pkg/errors"
)

var DB *sqlx.DB

// UserInfo 用户信息
type UserInfo struct {
	name string
	age  int8
	sex  int8
}

// DBConnect 初始化数据库
func DBConnect() error {
	dsName := "root:root@tcp(127.0.0.1:3306)/test"
	database, err := sqlx.Open("mysql", dsName)
	if err != nil {
		return errors.Wrapf(err, "db connect failed, dataSource:%s", dsName)
	}

	DB = database
	return nil
}

// QueryDemo 查询demo
func QueryDemo() error {
	// 初始化数据库
	err := DBConnect()
	if err != nil {
		return err
	}

	defer DB.Close()

	sql := "select * from user"

	rows, err := DB.Query(sql)
	if err != nil {
		return errors.Wrapf(err, "Query failed, sql:%s", sql)
	}

	for rows.Next() {
		var userInfo UserInfo
		err := rows.Scan(&userInfo.name, &userInfo.age, &userInfo.sex)
		if err != nil {
			return errors.Wrap(err, "Scan failed")
		}
	}

	return nil
}

func main() {
	err := QueryDemo()
	if err != nil {
		fmt.Printf("QueryDemo error, err:%+v", err)
	}
}
