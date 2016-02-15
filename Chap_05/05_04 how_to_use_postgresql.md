# How to use PostgreSQL

PostgreSQL은 오픈소스 관계형 데이터베이스 서버 (데이터베이스 관리 시스템)입니다. 
이것은 BSD 계열 라이센스로 공개되어 있습니다. 다른 오픈 소스 데이터베이스 시스템 (MySQL과 Firebird)와
Oracle, Sybase, IBM의 DB2 및 Microsoft SQL Server와 같은 독점 시스템에 대한 대안 중 하나입니다.

PostgreSQL과 MySQL을 비교하면 약간 크고 무겁습니다. 이유는 바로 Oracle의 대안으로 설계되어 있기 때문 입니다.   
따라서 기업의 응용 프로그램에서 PostgreSQL를  선택하는 것은 현명한 선택 중 하나일 것입니다. 

MySQL은 Oracle에 인수된 후 현재 점차 폐쇄되고 있습니다.(MySQL 5.5.31 이후의 모든 버전이 GPL 라이센스를    
준수하지 않음). 따라서, 대부분의 경우 프로젝트의 백엔드 데이터베이스로 MySQL 대신 PostgreSQL를 선택하게 
될지도 모릅니다.

## 드라이버
Go언어는 PostgreSQL을 지원하는 드라이버도 꽤 많이 구현되어 있습니다. 
국외에서는 많은 사람들이 개발에 PostgreSQL 데이터베이스를 사용하고 있기 때문입니다.

- https://github.com/lib/pq   
  database/sql 드라이버를 지원 합니다. 순수 Go로 작성되어 있습니다.
- https://github.com/jbarham/gopgsqldriver  
  database/sql 드라이버를 지원합니다. 순수 Go로 작성되어 있습니다.
- https://github.com/lxn/go-pgsql   
  database/sql 드라이버를 지원합니다. 순수 Go로 작성되어 있습니다.

예제에서는 첫 번째 드라이버를 사용해서 설명 합니다. 
이것은 가장 사용하는 사람이 많고, github에서도 비교적 활발하게 업데이트 되기 때문입니다.

## 스키마 
데이터베이스 테이블 작성:
``` SQL
CREATE TABLE userinfo
(
    uid serial NOT NULL,
    username character varying (100) NOT NULL,
    departname character varying (500) NOT NULL,
    Created date,
    CONSTRAINT userinfo_pkey PRIMARY KEY (uid)
)
WITH (OIDS = FALSE);

CREATE TABLE userdeatail
(
    uid integer,
    intro character varying (100)
    profile character varying (100)
)
WITH (OIDS = FALSE);
```
아래 예제에서는 데이터베이스 테이블의 데이터를 조작하거나 검색합니다. (추가 · 삭제 · 수정 · 검색)  
``` Go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
    "time"
)

const (
    DB_USER     = "postgres"
    DB_PASSWORD = "postgres"
    DB_NAME     = "test"
)

func main() {
    dbinfo := fmt.Sprintf("user=%s password=%s dbname=%s sslmode=disable",
        DB_USER, DB_PASSWORD, DB_NAME)
    db, err := sql.Open("postgres", dbinfo)
    checkErr(err)
    defer db.Close()

    fmt.Println("# Inserting values")

    var lastInsertId int
    err = db.QueryRow("INSERT INTO userinfo(username,departname,created) VALUES($1,$2,$3) returning uid;", "astaxie", "研发部门", "2012-12-09").Scan(&lastInsertId)
    checkErr(err)
    fmt.Println("last inserted id =", lastInsertId)

    fmt.Println("# Updating")
    stmt, err := db.Prepare("update userinfo set username=$1 where uid=$2")
    checkErr(err)

    res, err := stmt.Exec("astaxieupdate", lastInsertId)
    checkErr(err)

    affect, err := res.RowsAffected()
    checkErr(err)

    fmt.Println(affect, "rows changed")

    fmt.Println("# Querying")
    rows, err := db.Query("SELECT * FROM userinfo")
    checkErr(err)

    for rows.Next() {
        var uid int
        var username string
        var department string
        var created time.Time
        err = rows.Scan(&uid, &username, &department, &created)
        checkErr(err)
        fmt.Println("uid | username | department | created ")
        fmt.Printf("%3v | %8v | %6v | %6v\n", uid, username, department, created)
    }

    fmt.Println("# Deleting")
    stmt, err = db.Prepare("delete from userinfo where uid=$1")
    checkErr(err)

    res, err = stmt.Exec(lastInsertId)
    checkErr(err)

    affect, err = res.RowsAffected()
    checkErr(err)

    fmt.Println(affect, "rows changed")
}

func checkErr(err error) {
    if err != nil {
        panic(err)
    }
}

```
위의 코드는 PostgreSQL가 `$1` 과 `$2` 같은 방법으로 인수를 전달하는 것을 알 수 있습니다.   
MySQL과 같이 `?`를 사용하지  않습니다. 또한 sql.Open에서 dsn 정보의 구문은 MySQL 드라이버에서 dsn 구문과 
다르므로,사용할 때 이점을 주의하시기 바랍니다.

또한 pg는 `LastInsertId` 함수를 지원하지 않습니다. 
PostgreSQL의 내부에서 MySQL의 증분 ID를 반환하는 구현이 없기 때문입니다. 기타 코드는 거의 동일합니다.

