# How to use SQLite

SQLite는 오픈 소스 임베디드 관계형 데이터베이스 입니다. 설정 값없이 독립적으로 실행되고, 트랜잭션을 지원하는  
SQL 데이터베이스 엔진입니다. 매우 간단하고 쉽게 사용할 수 있고, 작고 효율이 높으며  신뢰성이 있습니다.    
다른 데이터베이스 관리 시스템과는 달리, SQLite는 설치 및 실행이  매우 간단하게 처리 됩니다.   
   
대다수의 경우 그냥 SQLite 바이너리 파일을 준비하는 것만으로 즉시 데이터베이스를 생성, 연결, 사용할 수 있습니다.     
만약 임베디드 데이터베이스 또는 솔루션을 찾고 있다면, SQLite는 절대적으로 고려할  만 합니다.   
SQLite는 오픈 소스 Access와 같은 존재입니다. 

## 드라이버
Go가 지원하는 sqlite 드라이버도 비교적 많지만 대부분은 database/sql 인터페이스를 지원하지 않습니다.

- https://github.com/mattn/go-sqlite3   
  database/sql 인터페이스를 지원하고 있습니다.    
  cgo (cgo에 대한 정보는 공식 문서 또는이 책의 마지막 장을 참고하십시오)에 따라 작성되어 있습니다.  
- https://github.com/feyeleanor/gosqlite3   
  database/sql 인터페이스를 지원하지 않습니다. cgo에 맞게 작성되어 있습니다.
- https://github.com/phf/go-sqlite3   
  database/sql 인터페이스를 지원하지 않습니다. cgo에 맞게 작성되어 있습니다.

현재 database/sql을 지원하는 데이터베이스 드라이버는 첫번째 뿐 입니다.   
필자 또한 이것을 사용해서 프로젝트를  개발하고 있습니다. 표준 인터페이스를 채택하는 것은 앞으로 여러     
데이터베이스에 좀더 유연하게 대처할 수 있음을 의미합니다. 

## 스키마 
예제에서 사용하는 데이터베이스 스키마는 다음과 같습니다. 
```
CREATE TABLE`userinfo` (
    `uid` INTEGER PRIMARY KEY AUTOINCREMENT,
    `username` VARCHAR (64) NULL,
    `departname` VARCHAR (64) NULL,
    `created` DATE NULL
);

CREATE TABLE`userdeatail` (
    `uid` INT (10) NULL,
    `intro` TEXT NULL,
    `profile` TEXT NULL,
    PRIMARY KEY (`uid`)
);
```
아래의 Go 프로그램은 데이터베이스 테이블의 데이터를 추가 · 삭제 · 수정 · 검색하는지를 보여 줍니다. 

``` Go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/mattn/go-sqlite3"
)

func main() {
    db, err := sql.Open("sqlite3", "./foo.db")
    checkErr(err)

    // insert
    stmt, err := db.Prepare("INSERT INTO userinfo(username, departname, created) values(?,?,?)")
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

상기 코드와 MySQL의 예제의 코드는 거의 동일 합니다. 유일한 차이점은 드라이버 가져 오기 부분입니다.   
`sql.Open`에서 SQLite의 방법으로 엽니다.


sqlite 관리 도구 : http://sqliteadmin.orbmu2k.de/  

쉽게 데이터베이스 관리를 새로 만들 수 있습니다.
