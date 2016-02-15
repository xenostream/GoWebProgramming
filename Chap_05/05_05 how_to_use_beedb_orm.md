# How to use beedb ORM

beedb 필자가 개발 한 Go 표준을 따르는 ORM 조작을 위한 라이브러리입니다.   
struct를 사용해서 테이블의 기록의 매핑기능을 제공 합니다. beedb는  충분히 가벼운 Go ORM 프레임워크 입니다.  
이 라이브러리를 개발 한 동기는 복잡한 ORM 학습 곡선 쉽게 학습하기 위해서 입니다. 
ORM의 실행 효율성과 기능 사이에서 균형을 잡는 것에 중점을 두고 개발하였습니다. beedb는 현재 오픈 소스 Go ORM   
프레임워크로서 비교적 완전한 라이브러리 중 하나입니다. 또한 실행 효율도 상당히 좋고 기능도 기본적으로 
수요를 만족시키고 있습니다. 그러나 현재는  연결 관계를 지원하지 않으며, 이것은 다음 버전의 주요 개발 포인트입니다.

beedb은 database/sql 표준 인터페이스를 지원하는  ORM 라이브러리입니다. 
따라서 이론적으로는 데이터베이스 드라이버가 database/sql 인터페이스를 지원만 한다면 beedb를 사용할 수 있습니다.   
현재까지 테스트 한 드라이버 패키지는 다음과 같습니다 :

Mysql : github.com/ziutek/mymysql/godrv [*]

Mysql : code.google.com/p/go-mysql-driver [*]

PostgreSQL : github.com/bmizerany/pq [*]

SQLite : github.com/mattn/go-sqlite3 [*]

MS ADODB : github.com/mattn/go-adodb[*]

ODBC : bitbucket.org/miquella/mgodbc[*]

## 설치

beedb는 go get 방식의 설치를 지원합니다. 이것은 Go Style의 방식에 완전히 부합되어  구현되었습니다.  
```
go get github.com/astaxie/beedb
```

## 초기화 방법
우선 해당 데이터베이스 드라이버 패키지를 import 해야합니다. 
database/sql 표준 인터페이스 패키지 및 beedb 패키지입니다 :
``` Go
import (
"database / sql"
"github.com/astaxie/beedb"
_ "github.com/ziutek/mymysql/godrv"
)
```
필요한 패키지를 가져온 후 데이터베이스에 대한 연결을 열어야 합니다. 
그후 beedb 객체(예를 들어 MySQL합니다)를  만듭니다
```
db, err := sql.Open("mymysql", "test/xiemengjun/123456")
if err != nil {
    panic(err)
}
orm := beedb.New(db)
```
beedb의 New 함수는 두 개의 인수를 필요로 합니다. 
첫 번째 인수는 표준 인터페이스 db이름이고, 두 번째 인수는 사용할 데이터베이스 엔진 입니다.   
만약 사용하는 데이터베이스 엔진이 MySQL/Sqlite 이면, 두 번째 인수는 생략해도 괜찮습니다.

만약 SQLServer를 사용한다면 다음과 같이 초기화해야 합니다.
```
orm = beedb.New(db "mssql")
```
만약 PostgreSQL을 사용한다면  초기화는 다음과 같습니다.
```
orm = beedb.New (db "pg")
```
현재 beedb는 프린트 디버깅을 지원하기 때문에, 아래의 코드처럼  디버깅을 할 수 있습니다.
```
beedb.OnDebug = true
```
다음 예에서는 이전 데이터베이스의 테이블 Userinfo을 사용합니다. 우선 목적 struct를 만듭니다.
```
type Userinfo struct {
    Uid int`PK` 
    // 만약 테이블의 기본 키가 id 아니면 pk 코멘트를 추가해야합니다. 이 필드가 기본 키임을 명시합니다.
    Username string
    Departname string
    Created time.Time
}
```
주의하시기 바랍니다. beedb에서는 낙타표기법을 사용하면 자동으로 언더바표기법으로 변환합니다.  
예를 들어 `UserInfo`라는 Struct를 정의하면 구현 될 때 `user_info`로 변환 됩니다. 필드 이름도 이  규칙을 따릅니다.  

## 데이터 삽입
아래의 코드는 데이터를 입력하는 방법을 보여줍니다. 
SQL문장을 직접사용해서 데이터를 입력하지 않으며, struct 개체를 이용해서 처리합니다.  
Save 인터페이스를 호출하면  struct 데이터를 데이터베이스에 실제로 저장 합니다.
```
var saveone Userinfo
saveone.Username = "Test Add User"
saveone.Departname = "Test Add Departname"
saveone.Created = time.Now()
orm.Save(&saveone)
```
데이터를 정상적으로 입력하면 증분 ID가 `saveone.Uid` 입니다. Save 인터페이스는 자동으로 저장 합니다.

beedb 인터페이스는 다른 종류의 데이터 입력방법을 제공합니다. 바로 map데이터 입력 기능입니다.
```
add := make(map[string]interface{})
add["username"] = "astaxie"
add["departname"] = "cloud develop"
add["created"] = "2012-12-02"
orm.SetTable("userinfo").Insert(add)
```
여러 데이터를 동시에 입력 
```
addslice := make([]map[string]interface{}, 10)
add:=make(map[string]interface{})
add2:=make(map[string]interface{})
add["username"] = "astaxie"
add["departname"] = "cloud develop"
add["created"] = "2012-12-02"
add2["username"] = "astaxie2"
add2["departname"] = "cloud develop2"
add2["created"] = "2012-12-02"
addslice = append(addslice, add, add2)
orm.SetTable("userinfo").InsertBatch(addslice)
```
위의 설명은 메소드 체인에 의한 검색방식과  조금 비슷 합니다. jquery 개발자라면 잘 알고있을 것입니다.  
매번 호출되는 method는 원래의 orm 개체를 반환하기 때문에 계속해서 개체의 다른 method를 호출 할 수 있습니다.

상기에서  호출 한 SetTable 함수는 앞으로 실행이 map에 사용되는 데이터베이스 테이블이 `userinfo` 이다! 라고
명시하고 있습니다.

## 데이터 업데이트
계속해서 상기의 예제로 데이터를 업데이트 합니다.   
현재 saveone의 기본 키 값이 이미 존재 합니다. 이때 save 인터페이스를 호출하면  beedb에서  자동으로 update를 
호출하여 데이터를 업데이트 합니다.이것은 입력 작업이 없습니다.
```
saveone.Username = "Update Username"
saveone.Departname = "Update Departname"
saveone.Created = time.Now()
orm.Save(&saveone) // 현재 saveone에는 기본 키가 있습니다. 업데이트를 수행합니다.
```
데이터의 업데이트는 map 조작의 직접적인 사용을 지원합니다.
```
t := make(map[string]interface{})
t["username"] = "astaxie"
orm.SetTable("userinfo").SetPK("uid").Where(2).Update(t)
```
여기에서  몇 가지 beedb 함수를 호출 해 봅니다.

SetPK : ORM에게 데이터베이스 테이블 `userinfo`의 기본 키가`uid`임을 명시 합니다.

Where : 조건을 설정하는 데 사용 됩니다. 여러 인수를 지원하는 첫 번째 인수가 만약 정수이면,  
        Where ( "기본 키 =?", 값)가 호출 된 것 입니다.
Update 함수는 map 형의 데이터를 수신 데이터로  업데이트 합니다.

## 데이터 검색
beedb의 검색 인터페이스는 사용하기 쉽고,  구체적인 사용 방법은 아래의 예를 참조 하십시오.

예1)  기본 키에 의해 데이터를 검색 :
```
var user Userinfo
// Where accepts two arguments, supports integers
orm.Where("uid=?", 27).Find(&user)
```

예2) 
```
var user2 Userinfo
orm.Where(3).Find(&user2) // short form that omits primary key.
```
예3) 주 키가 아닌 조건 :
```
var user3 Userinfo
// Where accepts two arguments, supports char type.
orm.Where("name = ?", "john").Find(&user3)
```

예4) 더 복잡한 조건 :
```
var user4 Userinfo
// Where accepts three arguments
orm.Where("name = ? and age < ?", "john", 88).Find(&user4)
```

아래의 인터페이스를 통해 여러 데이터를 얻을 수 있습니다. 예를 참조 하십시오.

예1) 조건 id > 3에 따라 20에서 시작되는 10 건의 데이터를 가져 옵니다.
```
var allusers []Userinfo
err := orm.Where("id > ?", "3").Limit(10,20).FindAll(&allusers)
```

예2) limit의 두 번째 인수는 선택 사항입니다. 기본값은 0부터 시작됩니다. 10 개의 데이터를 가져옵니다.
```
var tenusers []Userinfo
err := orm.Where("id > ?", "3").Limit(10).FindAll(&tenusers)
```
예3) 모든 데이터를 가져옵니다.
```
var everyone []Userinfo
err := orm.OrderBy("uid desc,username asc").FindAll(&everyone)
```
위에서는 Limit 기능이 있습니다. 이것은  검색 결과의 수를 제어하는​​ 데 사용 됩니다.

- Limit() : 2 개의 인수를 지원합니다.   
  첫 번째 인수는 검색 수를 나타내며, 두 번째 인수는 검색 할 데이터의 시작 위치를 나타냅니다. 기본값은 0입니다.

- OrderBy() :이 함수는 검색을 정렬하는 데 사용됩니다. 인수는 정렬 조건이어야합니다.

위의 예에서 얻을 데이터가 직접 struct 개체에 매핑됩니다. 
만약 데이터를 map으로 사용하고 싶을 뿐이라면 아래의 방법으로 실현 할 수 있습니다 :
```
a, _ := orm.SetTable("userinfo").SetPK("uid").Where(2).Select("uid,username").FindMap()
```

예에서 또한 새로운 인터페이스 함수 Select가 나왔습니다. 이 함수는 여러 필드를 찾고 있는지 정의하는 데   
사용됩니다. 기본적으로 모든 필드는 `*` 입니다.

FindMap() 함수는`[] map [string] [] byte` 형을 돌려줍니다. 따라서 자신의 형태 변환을 수행해야 합니다.

## 데이터 삭제
beedb는 풍부한 데이터 삭제 인터페이스를 제공합니다. 아래의 예를 참조하십시오.

예1) 단일 데이터 삭제
```
// saveone is the one in above example.
orm.Delete(&saveone)
```
예2) 여러 데이터를 삭제
```
// alluser is the slice which gets multiple records.
orm.DeleteAll(&alluser)
```

예3) sql에 따라 데이터를 삭제
```
orm.SetTable("userinfo").Where("uid>?", 3).DeleteRow()
```

## 관계 검색
현재 beedb은 struct 관계를 지원하지 않습니다. 그러나 일부 응용 프로그램은 관계에 의한 검색을 필요로하고 있습니다.   
따라서 현재 beedb에서는 간단한 솔루션을 제공 합니다.
```
a, _ := orm.SetTable("userinfo").Join("LEFT", "userdetail", "userinfo.uid=userdetail.uid")
    .Where("userinfo.uid=?", 1).Select("userinfo.uid,userinfo.username,userdetail.profile").FindMap()
```
위의 코드에서는 새로운 인터페이스의 Join 함수가 나오고 있습니다. 이 함수는 세 개의 인수가 있습니다.

- 첫 번째 인수는 : INNER, LEFT, OURTER, CROSS 등이 포함 됩니다
- 둘째 인수는 연결 테이블을 나타냅니다
- 세 번째 인수는 연결 조건을 나타냅니다


## Group By과 Having
일부 응용 프로그램이 group by 및 having 기능을 필요로하고 있기 때문에 beedb도 간단한 실현 방법을 제공 합니다.
```
a, _ := orm.SetTable("userinfo").GroupBy("username").Having("username='astaxie'").FindMap()
```
위의 코드에서 나타나는 2 개의 새로운 인터페이스 함수

GroupBy : groupby 필드를 수행하는 데 사용 됩니다

Having : having을 실행할 때의 조건을 지정하는 데 사용 됩니다

## 더 나아가
현재 beedb 이미 많은 국내외에서 사용자로 부터  피드백을 얻고 있습니다. 
현재 보류상태이며, 이후에 몇 가지 기능으로 성능 개선이 진행될 예정 입니다.

- interface 설계의 구현. database/sql/driver의 설계와 유사한 beedb 인터페이스를 설계합니다.   
  그 후 해당 데이터베이스 CRUD 작업을 제공합니다.
- 관계형 데이터베이스 설계의 실현.   
  일대일, 일대 다, 다 대다 지원을 제공합니다. 코드는 다음과 같습니다 :

``` Go
          type Profile struct {
              Nickname string
              Mobile   string
          }
          type Userinfo struct {
              Uid         int
              PK_Username string
              Departname  string
              Created     time.Time
              Profile     HasOne
          }
```
- 자동으로 데이터베이스 테이블, 인덱스를 작성
- 연결 풀의 구현에 goroutine을 사용.
