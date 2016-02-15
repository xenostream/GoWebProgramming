# database/sql interface

Go는 PHP와는 다르게 공식적으로 데이터베이스 드라이버가 제공되지 않습니다.   
단지 개발자가 개발하기 위한  데이터베이스 드라이버의  표준 인터페이스를 제공 합니다. 개발자는 정의 된   
인터페이스에 따라 원하는 데이터베이스 드라이버를 직접 개발할 수 있습니다. 이것은 장점이 있습니다.   
표준 인터페이스를 참조하기만 하면 어떠한 코드도  개발할 수 있습니다. 이후 데이터베이스를  마이그레이션 할 때,   
어떤 수정도 하지 않아도 되는 이식성의 장점이 있습니다. Go는 어떤 표준 인터페이스를 정의하고있는 것일까요? 
자세히 분석해 보기로 하겠습니다. 

## sql.Register
database/sql에 존재하는 함수로  데이터베이스 드라이버를 등록하는 함수입니다. 타사 데이터베이스 드라이버를    
개발할 때는 모든 init 함수를 구현 합니다. init 함수는이 `Register(name string, driver driver.Driver)`를 
호출해서, 드라이버의 등록을 완료 합니다.

mymysql, sqlite3 드라이버는 어떻게 호출하는지 살펴 보기로 하겠습니다.   
``` Go
//https://github.com/mattn/go-sqlite3 driver
func init() {
    sql.Register("sqlite3", &SQLiteDriver{})
}

//https://github.com/mikespook/mymysql driver
// Driver automatically registered in database/sql
var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
func init() {
    Register("SET NAMES utf8")
    sql.Register("mymysql", &d)
}
```
각가의  데이터베이스 드라이버는 모두 이 함수를 호출해서  자신의 데이터베이스 드라이버의 이름과 목적을 명시하고,       
해당 driver를 등록하는 작업을 수행합니다. database/sql 내부에서는 하나의 map을 통해 사용자가 정의한   
드라이버를 저장 합니다.
```
var drivers = make(map[string]driver.Driver)

drivers[name] = driver
```

왜냐하면 database/sql을 이용해서 등록 할 때 여러개의  데이터베이스 드라이버를 등록 할 수 있기 때문 입니다.  
이때, 중복되지 않도록 하는 것이 좋습니다. 

database/sql 인터페이스를 이용해서  타사 라이브러리를 사용할 때 다음과 같이 사용하면 됩니다.   
```
import (
    "database/sql"
    _ "github.com/mattn/go-sqlite3"
)
```
초보자의 경우는 `_` 문자때문에 혼란스러울 수 있습니다. 이 것은 Go의 장점중 하나인 기능입니다. 이미 이 기호를,  
변수를 무시할 때 사용한 적이 있습니다. 즉, 값을 사용하지 않고 버리는 기능을 합니다. (컴파일러 에러 방지)   

import 문에서 사용되면 그 기능이 약간 다르게 작동합니다. 2.3 절에서  init 함수의 초기화 과정을 설명 했습니다.   
패키지를 import 할때 패키지의 init 함수가 자동으로 호출되고, 패키지에 대한 초기화가 완료 됩니다.   
따라서 위의 데이터베이스 드라이버 패키지를 가져 오면 init 함수가 자동으로 호출 됩니다. 다음은 init 함수에서   
이 데이터베이스 드라이버를 등록한 후, 코드에서 직접이 데이터베이스 드라이버를 명시해서  사용할 수 있습니다.

## driver.Driver
Driver는 데이터베이스 드라이버의 인터페이스 입니다. method가 하나만  정의되어 있습니다.
Open(name string) 메소드는 데이터베이스 Conn 인터페이스를 반환 합니다.
```
type Driver interface {
    Open(name string) (Conn, error)
}
```
반환되는 Conn 값은, 한번의 goroutine의 작업을 수행 할 수있다는 것을 주의 하시기 바랍니다.    
Conn 객체를 다른 goroutine에서는  사용할 수 없습니다. 즉, 다음 코드는 오류가 발생 합니다.   

```
...
go goroutineA (Conn) // 검색 작업을 수행
go goroutineB (Conn) // 삽입 조작의 실행
...
```

상기의 코드는 Go에게 어떤 작업이 어떤 goroutine 의해 만들어진 것인지 구별 할 수 없기 때문에 데이터의 혼란을   
초래하게 됩니다. 예를 들어 goroutineA에서 실행 된 검색결과를 goroutineB에 돌려주는 경우 B는 이 결과가  자신이   
실행 한 삽입 데이터라고 판단해 버리게 됩니다.   

타사 드라이버는 모두 이 함수를 정의하고 있습니다. 이것은 name 인수를 분석하여 원하는 데이터베이스 연결 정보를   
얻을 수 있습니다. 분석이 완료되면 이 정보를 사용해서 하나의 Conn 객체를 초기화한 후 그 값을 반환 합니다.

## driver.Conn
Conn은 데이터베이스 연결 인터페이스 정의 입니다. 여기에는 몇 가지 방법이 정의되어 있습니다.   
이 Conn는 하나의 goroutine에서만 사용할 수 있고, 여러 goroutine에서 사용할 수 없습니다. 자세한 내용은 위의    
설명을 확인하시기 바랍니다.
```
type Conn interface {
    Prepare(query string) (Stmt, error)
    Close() error
    Begin() (Tx, error)
}
```
Prepare 함수는 현재 연결과 관련된 실행되는 SQL 문장의 준비상태를 반환 합니다. 검색, 삭제 등의 작업을 수행 할   
수 있습니다.

Close 함수는 현재의 연결을 닫습니다. 연결이 가지고있는 자원을 해제하는 등 정리 작업을 수행합니다.    
드라이버는 database/sql 중 커넥션풀을(conn pool)을 구현하고 있기 때문에 문제가 발생할 수 있기 때문입니다.    

Begin 함수는 트랜잭션 처리를 나타내는 Tx를 반환 합니다. 이것을 이용하여 검색, 업데이트 등의 작업을 수행 할 수   
있습니다. 또는 트랜잭션에 대해서  롤백이나 커밋 작업을 수행 합니다.

## driver.Stmt
Stmt는 모든 준비가 된 상태입니다. Conn의 속성과 같이 하나의 goroutine에서만 사용할 수 있습니다.   
여러 goroutine에서 동시에 사용할 수 없습니다.
```
type Stmt interface {
    Close() error
    NumInput() int
    Exec(args []Value) (Result, error)
    Query(args []Value) (Rows, error)
}
```
Close 함수는 현재의 연결 상태를 닫습니다. 그러나 만약 현재 실행되는 query가 있다면  rows 데이터를 반환 합니다.  

NumInput 함수는 현재 예약 된 인수의 개수를 반환합니다. >= 0 값이 반환되었을 때는 데이터베이스 드라이버가   
자동으로 인수를 검사 합니다. 데이터베이스 드라이버 패키지가 예약 된 인수를 모르는 경우는 -1을 돌려 줍니다.

Exec 함수는 Prepare에서 준비된 sql을 실행 합니다. 인수를 전달하거나  update/insert 등의 작업을 수행합니다.   
Result 데이터를 반환 합니다.

Query 함수는 Prepare에서 준비된 sql을 실행 합니다. 필요한 인수를 전달 또는  select 작업을 수행 합니다.    
Rows 결과 셋을 반환 합니다.


## driver.Tx
트랜잭션 처리는 일반적으로 2 개의 프로세스가 존재 합니다. 커밋 또는 롤백 입니다.  
데이터베이스 드라이버에서 이 두 함수를 구현하면 이 두가지 기능을 사용할 수 있습니다. 
```
type Tx interface {
    Commit() error
    Rollback() error
}
```
이 두 가지 기능 중 하나는 커밋에 사용되고 다른 하나는 롤백에 사용 됩니다.

## driver.Execer
이것은 Conn 객체를 구현할 수있는 인터페이스입니다.
```
type Execer interface {
    Exec(query string, args []Value) (Result, error)
}
```
만약이 인터페이스의 정의가 없으면 DB.Exec가 호출 됩니다. 
즉, 먼저 Prepare가 호출 된 Stmt을 반환 후 Stmt의 Exec가  실행되고 Stmt이 닫힙니다.

## driver.Result
이것은 Update/Insert 등의 작업을 수행 한 결과를 돌려주는 인터페이스의 정의입니다.
```
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```
LastInsertId 함수는 데이터베이스에 의해 수행 된 삽입 조작에 의해 얻어지는 증가 ID를 반환 합니다.

RowsAffected 함수는 query 작업에 영향을 받은  데이터의 수를 돌려줍니다.

## driver.Rows
Rows는 실행 된 검색 결과 셋 인터페이스의 정의 입니다
```
type Rows interface {
    Columns() []string
    Close() error
    Next(dest []Value) error
}
```
Columns 함수는 데이터베이스 검색에서 필드 정보를 반환 합니다. 
이것이 반환 slice와 sql 검색 필드는 하나 하나가 직접 대응되며, 모든 테이블의 필드를 돌려주는 것은 아닙니다.

Close 함수는 Rows 반복자를 닫기 위해 사용됩니다.

Next 함수는 하나의 데이터를 반환하는 데 사용됩니다. 
데이터는 dest에 할당되어 dest 중의 요소는 string을 제외하고 driver.Value 값 이어야 합니다. 
반환되는 데이터 중의 모든 string은 []byte로 변환 될 필요가 있습니다. 
만약 마지막에 데이터가없는 경우 Next 함수는 io.EOF을 반환 합니다.


## driver.RowsAffected
RowsAffected는 사실 int64의 별칭 입니다. 그러나 Result 인터페이스를 구현하고 있기 때문에,   
낮은 계층에서 Result의 표시 방법을 구현하기 위해 사용됩니다.
```
type RowsAffected int64

func (RowsAffected) LastInsertId() (int64, error)

func (v RowsAffected) RowsAffected() (int64, error)
```
## driver.Value
Value는 사실 빈 인터페이스 입니다. 어떤 데이터도 저장할 수 있습니다.
```
type Value interface{}
```
driver.Value는 드라이버가 반드시 조작 할 수 Value 입니다. Value가 nil이 아니라면 다음 중 하나 입니다.
```
int64
float64
bool
[]byte
string   [*] Except Rows.Next which cannot return string
time.Time
```
## driver.ValueConverter
ValueConverter 인터페이스는 보통 값을 driver.Value 인터페이스 형식으로 변환하는 방법이 정의되어 있습니다.
```
type ValueConverter interface {
    ConvertValue(v interface{}) (Value, error)
}
```
개발중인 데이터베이스 드라이버 패키지에서는이 인터페이스의 함수가 많은 부분에서 이용되고 있습니다. 
이 ValueConverter는 장점이 많이 있습니다 :

- driver.value는 데이터베이스 테이블의 해당 필드에 특화되어 있습니다.   
  예를 들어 int64 데이터가 어떻게 데이터베이스 테이블의 unit16 필드로 변환되는지 등 입니다.
- 데이터베이스의 검색 결과를 driver.Value 값으로 변환 합니다.
- scan 함수를 이용해서 driver.Value 값을 사용자가 정의한 값으로 변환할 수 있습니다.

## driver.Valuer
Valuer 인터페이스는 driver.Value 한개 값을  돌려주는 메소드가 정의되어 있습니다.
```
type Valuer interface {
    Value() (Value, error)
}
```
많은 유형이 Value 메소드를 구현하고 있습니다. 자기 자신과 driver.Value 에서 사용하고 있습니다.   

위의 설명에 따라 드라이버 개발에 대한 기본적인 것을 이해할 수 있다고 생각합니다. 
이 드라이버는 단지이 인터페이스를 구현하여 추가 · 삭제 · 검색 · 수정 등의 기본 조작을 가능하게 할 뿐입니다.   
이후에는 해당 데이터베이스에 데이터를 교환하는 등 세세한 문제가 남아 있습니다만 여기에서는 자세히 언급하지 않습니다.  

## database/sql
database/sql은  database/sql/driver에서 제공되는 인터페이스의 기초 한 더 상위 계층의 메소드를 정의하고 있습니다.  
이것은 데이터베이스 조작을 용이하게 하고 내부에서 conn pool을 구현하고 있습니다.
```
type DB struct {
    driver   driver.Driver
    dsn      string
    mu       sync.Mutex // protects freeConn and closed
    freeConn []driver.Conn
    closed   bool
}   
```
Open 함수가 DB 객체를 반환 합니다. 이 중에는 freeConn이 있고, 이것이 바로 간단한 연결 풀 입니다. 
이 구현은 매우 간단하고  간단 합니다. Db.prepare을 실행할 때 `defer db.putConn(ci, err)`를 실행 합니다.   
즉이 연결을 연결 풀에 자동으로 등록합니다. 매번 conn을 호출 할 때는 먼저 freeConn의 길이가 0보다 큰지 확인하고  
0보다 큰 경우는 conn 객체를 재사용 할 수있다 것을 보장 합니다. 곧바로 직접 사용해도  괜찮습니다. 
만약 0보다 작은 경우 conn 객체를  생성해서 이값을 반환 합니다.


