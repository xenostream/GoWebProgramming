# NOSQL

NoSQL(Not Only SQL)은 관계형 데이터베이스가 아닌 것을 의미 합니다.   
Web2.0의 증가함에도 전통적인 관계형 데이터베이스가 Web2.0 페이지에 사용되고 있습니다.   
특히 매우 큰 규모와 고급 멀티 스레드를 사용하는 SNS형태의 Web2.0의 동적 홈페이지에서는   
전통적인 관계형 데이터베이스로는 기능이 많이 부족한 현상이 발생하고 있습니다.   

이러한 해결하기 어려운 문제가 노출되고, 비 관계형데이터베이스는 그러한 특징으로 매우 빠르게 발전하고 있습니다.

Go언어는 `21세기의 C언어`로서 `NOSQL`도 아주 잘 지원하고 있습니다.   
현재 유행하고있는 NOSQL에는 주로 redis, mongoDB, Cassandra와 Membase 등이 있습니다.    
이런  데이터베이스는 모두 고성능 멀티스레드 특징을 가지고 있으며, 이미 널리 모든 응용 프로그램에서   
사용되고 있습니다. 여기에서는 주로 redis와 mongoDB 에 대해서  설명 합니다.

## redis
redis는 key-value를 저장하는 시스템 입니다. Memcached와 비슷하며, 차이점은 저장되는 value형이  더 많으며,   
string(문자열), list(목록) set(세트)와 zset(정렬 된 set)을 포함 합니다.

현재 redis가 가장 많이 사용되고있는 곳은 마이크로 블로그 플랫폼 것입니다.    
그 다음에 Facebook에 인수 된 이미지 포럼인 instagram이 있습니다.   
기타 유명한 인터넷 기업 (http://redis.io/topics/whos-using-redis)도 그렇습니다.

Go는 현재 redis 드라이버에서 다음을 지원합니다
- https://github.com/alphazero/Go-Redis
- http://code.google.com/p/tideland-rdc/
- https://github.com/simonz05/godis
- https://github.com/hoisie/redis.go

필자가 개발한 최신 드라이버에서는 일부 bug가 수정되어 있습니다.    
이 드라이버는 필자의 단축 도메인 이름 서비스 프로젝트에서 사용되고 있습니다. (매일 200W정도의 PV 수가 있습니다.)

https://github.com/astaxie/goredis

다음 예제는 직접 개발한  redis 드라이버를 이용해서 데이터를 처리하는 예제입니다. 
``` Go
package main

import (
    "github.com/astaxie/goredis"
    "fmt"
)

func main() {
    var client goredis.Client

    // Set the default port in Redis
    client.Addr = "127.0.0.1:6379"

    // string manipulation
    client.Set("a", []byte("hello"))
    val, _ := client.Get("a")
    fmt.Println(string(val))
    client.Del("a")

    // list operation
    vals := []string{"a", "b", "c", "d", "e"}
    for _, v := range vals {
        client.Rpush("l", []byte(v))
    }
    dbvals,_ := client.Lrange("l", 0, 4)
    for i, v := range dbvals {
        println(i,":",string(v))
    }
    client.Del("l")
}

```
redis의 조작이 매우 간단함을  알 수 있을거라 생각 합니다. 이 코드는 실제 프로젝트에서 사용하고 있고, 성능도  
매우 높습니다. client 명령과 redis 명령은 기본적으로 동일 합니다. 그래서 원래 redis 작업과 매우 비슷하게 처리 합니다.  

## mongoDB


MongoDB는 고성능 오픈 소스 모덜리스 문서형 데이터베이스입니다. 이것은 관계형 데이터베이스와 비 관계형 데이터베이스   
사이의 제품 입니다. 비 관계형 데이터베이스 내에서도  기능이 가장 풍부한 관계형 데이터베이스와 가장 비슷합니다.   
지원되는 데이터 형식은 json과 비슷한 bjson 형식으로 데이터를 저장 합니다. 
따라서 상대적으로 복잡한 데이터를 저장할 수 있습니다. Mongo의 가장 큰 특징은 검색 언어가 매우 강력한  문법을   
지원하며 객체 지향 방식의 검색 문장을 사용하는 것입니다. 데이터베이스에 인덱스를 설정할 수도 있습니다.

아래의 그림은 mysql과 mongoDB 사이의 대응 관계를 보여줍니다. 
매우 간단합니다만, mongoDB의 성능은 매우 좋습니다.
![](5.6.mongodb.png)   
그림 5.1 MongoDB와 Mysql 작업의 대응도

현재 Go에서 지원되는 mongoDB의 가장 좋은 드라이버는 mgo (http://labix.org/mgo)입니다.   
이 드라이버는 현재 가장 많이 사용하는 패키지에 속합니다. 

다음 예제는 Go에서 mongoDB를 조작하는 방법에 대해서 설명 합니다.
``` Go
package main

import (
    "fmt"
    "labix.org/v2/mgo"
    "labix.org/v2/mgo/bson"
)

type Person struct {
    Name string
    Phone string
}

func main() {
    session, err := mgo.Dial("server1.example.com,server2.example.com")
    if err != nil {
        panic(err)
    }
    defer session.Close()

    session.SetMode(mgo.Monotonic, true)

    c := session.DB("test").C("people")
    err = c.Insert(&Person{"Ale", "+55 53 8116 9639"},
        &Person{"Cla", "+55 53 8402 8510"})
    if err != nil {
        panic(err)
    }

    result := Person{}
    err = c.Find(bson.M{"name": "Ale"}).One(&result)
    if err != nil {
        panic(err)
    }

    fmt.Println("Phone:", result.Phone)
}

```
mgo의 조작 방법과 beedb의 사용법과 매우 비슷하다는 것을 알 수 있습니다. 
struct를 이용해서 데이터를 조작하는 방법 입니다. 이것이바로  Go Style 입니다.

