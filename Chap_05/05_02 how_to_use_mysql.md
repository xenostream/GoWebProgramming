# How to use MySQL

현재 Internet에서 유행하고 있는 홈페이지 프레임워크의 방법 중 하나는 `LAMP` 입니다.   
이 중 M은 MySQL을 의미합니다. MySQL은 무료이며, 오픈 소스에 기반한  데이터베이스 이며, 현재 수 많은   
Web 개발시에 데이터를 저장하는 백엔드 데이터베이스 스토리지 엔진으로 사용되고 있습니다. 

## MySQL 드라이버
Go는 MySQL을 지원하는 드라이버가 현재까지 다음과 같이 몇 가지가 존재 합니다. 
이 중에는 database/sql 표준을 지원하고, 또 어떤 것은 그 자체로 인터페이스의 구현을 사용하는 것도 있습니다.   
자주 사용되는 것은 다음의 몇 가지 있습니다 :

- https://github.com/go-sql-driver/mysql   
  database/sql을 지원하고 있으며, 모두  go로 작성되어 있습니다.
- https://github.com/ziutek/mymysql   
  database/sql을 지원하고 있으며, 독자적으로 정의 된 인터페이스도 지원하고 있습니다. go로 작성되어 있습니다.  
- https://github.com/Philio/GoMySQL  
  database/sql을 지원하지 않습니다. 고유의 인터페이스를 가지며, go로 작성되어 있습니다.

예제에서는 처음 드라이버를 사용 하겠습니다. 또한 이 드라이버를 여러분에게 추천하는 이유는 다음과 같습니다.   

- 이 드라이버는 비교적 최근에 만들어 졌으며,  유지 보수도 좋은 편 입니다.
- 완전한  database/sql 인터페이스를 지원 합니다.
- keepalive를 지원하고 있습니다. 즉, 연결을 유지할 수 있습니다.  
  mymysql도 keepalive를 지원하지만 스레드로부터 안전하지 않습니다. 이것은 저 수준에서 keepalive를 지원하고    
  있으며, 스레드로 부터 안전하게 작동합니다. 

## 코드 예
이후 몇 개의 절에서 사용하는 코드에서는 동일한 데이터베이스 스키마를 사용하고 있습니다.  
데이터베이스 test,  사용자 이름 test, 관련 사용자 정보 테이블, userinfo, 사용자의 자세한 정보는 userdetail.  
```
CREATE TABLE `userinfo` (
    `uid` INT(10) NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(64) NULL DEFAULT NULL,
    `departname` VARCHAR(64) NULL DEFAULT NULL,
    `created` DATE NULL DEFAULT NULL,
    PRIMARY KEY (`uid`)
);


CREATE TABLE`userdetail` (
    `uid` INT (10) NOT NULL DEFAULT '0'
    `intro` TEXT NULL,
    `profile` TEXT NULL,
    PRIMARY KEY (`uid`)
);
```
다음 예제에서는 database/sql 인터페이스를 사용하여 데이터베이스의 테이블에 추가 · 삭제 · 수정 · 검색 작업을  
어떻게 수행하는지 보여 줍니다.
``` Go
package main

import (
    _ "github.com/go-sql-driver/mysql"
    "database/sql"
    "fmt"
)

func main() {
    db, err := sql.Open("mysql", "astaxie:astaxie@/test?charset=utf8")
    checkErr(err)

    // insert
    stmt, err := db.Prepare("INSERT userinfo SET username=?,departname=?,created=?")
    checkErr(err)

    res, err := stmt.Exec("astaxie", "研发部门", "2012-12-09")
    checkErr(err)

    id, err := res.LastInsertId()
    checkErr(err)

    fmt.Println(id)
    // update
    stmt, err = db.Prepare("update userinfo set username=? where uid=?")
    checkErr(err)

    res, err = stmt.Exec("astaxieupdate", id)
    checkErr(err)

    affect, err := res.RowsAffected()
    checkErr(err)

    fmt.Println(affect)

    // query
    rows, err := db.Query("SELECT * FROM userinfo")
    checkErr(err)

    for rows.Next() {
        var uid int
        var username string
        var department string
        var created string
        err = rows.Scan(&uid, &username, &department, &created)
        checkErr(err)
        fmt.Println(uid)
        fmt.Println(username)
        fmt.Println(department)
        fmt.Println(created)
    }

    // delete
    stmt, err = db.Prepare("delete from userinfo where uid=?")
    checkErr(err)

    res, err = stmt.Exec(id)
    checkErr(err)

    affect, err = res.RowsAffected()
    checkErr(err)

    fmt.Println(affect)

    db.Close()

}

func checkErr(err error) {
    if err != nil {
        panic(err)
    }
}

```

위의 코드에서 알 수 있듯이, Mysql 데이터베이스를 조작하는 것은  매우 간단하게 처리할 수 있습니다.  

몇 가지 핵심 기능에 대해 설명합니다.

sql.Open () 함수는 이미 등록 된 데이터베이스 드라이버를 여는 데 사용 됩니다.   
go-sql-driver에서 mysql 데이터베이스 드라이버를 등록하고 두 번째 인수는 DSN(Data Source Name)입니다.   
이것은 go-sql-driver가 정의 데이터베이스 연결 및 설정 정보이며, 다음 구문을 지원 합니다 :
```
  user@unix(/path/to/socket)/dbname?charset=utf8
  user:password@tcp(localhost:5555)/dbname?charset=utf8
  user:password@/dbname
  user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname
```
db.Prepare() 함수는 sql 작업을 수행 할때 인자를 동적으로 설정해서 데이터를 반환하는 데 사용됩니다. 
그 후, 준비의 실행 상태를 반환 합니다.

db.Query() 함수는 직접 Sql을 실행 Rows 결과를 반환하는 데 사용됩니다.

stmt.Exec() 함수는 stmt가 준비된 SQL 문을 실행하는 데 사용 됩니다.

`=?` 구문을 사용해서 인수를 전달해서 처리한다는 것을 알 수 있습니다.  
이런 방식으로 어느 정도 SQL 인젝션을 방지 할 수 있습니다.




