# Interface

## interface
Go는 매우 섬세하게 설계된 interface가 있습니다. 이것은 객체지향과 내용 구성시에 매우 편리하게 사용됩니다.   
이 장 끝냈을 때, interface의 성공적인 디자인 방식에 감탄하게 될 것입니다.

### interface 란 무엇인가
간단하게 말하면, interface는 method의 조합 입니다. interface를 통해 개체의 행동을 정의 할 수 있습니다.   
이전 장의 마지막 예제에서 Student와 Employee 모두 SayHi 메소드를 가지고 있었습니다.    
그들의 내부 처리 방식은 다르지만, 그것은 중요하지 않습니다. 중요한 것은 그들 모두 `say hi`라고 말한다는 것입니다.  

여기에, 계속 더 확장 해 보겠습니다. Student와 Employee에서 또 다른 메소드 `Sing`을 작성합니다.    
Student에는 BorrowMoney 메소드를 추가하고,  Employee는 SpendSalary를 추가 하겠습니다.    

Student에는 총 3가지 메소드가 존재합니다. SayHi, Sing, BorrowMoney 입니다. 
Employee는 SayHi, Sing, SpendSalary의 3가지 메소드 입니다. 

위와 같은 조합을 interface(개체 Student와 Employee에 추가)라고 합니다. 
예를 들어,  Student와 Employee에서 모두 interface인 SayHi와 Sing을 구현하고 있습니다. 이 두 개체는 interface 형입니다.  
Employee에서 interface는  SayHi, Sing 입니다. BorrowMoney는 구현하지 않습니다. 
Employee는 BorrowMoney 메소드를 구현하지 않기 때문입니다.

### interface 형
interface 형이 메소드 세트를 정의 합니다. 만약 객체가 인터페이스가되는 모든 메소드를 구현한다고하면,  
이 오브젝트는 이 인터페이스를 구현하는 것입니다. 자세한 문법은 다음이 예제를 참고 하십시오.   
``` Go
type Human struct {
    name string
    age int
    phone string
}

type Student struct {
    Human            // 익명필드 Human
    school string
    loan float32
}

type Employee struct {
    Human            // 익명 필드 Human
    company string
    money float32
}

// Human 객체에 SayHi 메서드를 구현합니다.
func (h *Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s \n", h.name, h.phone)
}

// Human 객체에 Sing 메소드를 구현합니다.
func (h *Human) Sing(lyrics string) {
    fmt.Println("La la, la la la, la la la la la ...", lyrics)
}

// Human 객체에 Guzzle 메소드를 구현합니다.
func (h *Human) Guzzle(beerStein string) {
    fmt.Println("Guzzle Guzzle Guzzle ...", beerStein)
}

// Employee는 Human의 SayHi 메서드를 오버로드합니다.
func (e *Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s \n", e.name,
        e.company, e.phone) //여러 줄에 기록해도 괜찮습니다.
}

// Student는 BorrowMoney 메소드를 구현합니다.
func (s *Student) BorrowMoney(amount float32) {
    s.loan += amount       // (again and again and ...)
}

// Employee는 SpendSalary 메소드를 구현합니다.
func (e *Employee) SpendSalary(amount float32) {
    e.money -= amount      // More vodka please !!! Get me through the day!
}

// interface를 정의합니다.
type Men interface {
    SayHi()
    Sing(lyrics string)
    Guzzle(beerStein string)
}

type YoungChap interface {
    SayHi()
    Sing(song string)
    BorrowMoney(amount float32)
}

type ElderlyGent interface {
    SayHi()
    Sing(song string)
    SpendSalary(amount float32)
}

위의 코드를 통해서 알수 있듯이,  interface는 모든 객체에서 구현할 기능임을 알 수 있습니다.    
Men interface는 Human, Student 및 Employee에 의해 구현 됩니다. 예를 들어 Student는 Men과 YoungChap    
두 interface를 구현하게 됩니다.   

마지막으로, 모든 종류를 처리할 수 있는 빈interface (여기에서는 interface{}라고 정의합시다)를 구현하고 있습니다.   
빈 인터페이스는 0개의 메소드가 포함 된 interface 입니다.   

### interface 값
interface에는 도대체 어떤 값이 존재 할까요? interface 변수를 선언하면,  이 변수는 interface 유형의 모든 객체를    
저장할 수 있습니다. 위의 예제로 표현하면, Men interface 변수 m을 선언 했습니다.   
이 m 변수는 Human, Student 또는 Employee 값을 저장할 수 있게 된다는 의미 입니다. 

m 은 3 가지 형태를 가질 수있는 객체이므로,  Men형의 요소를 포함하는 slice를 정의 할 수 있습니다.    
이 slice는 Men 인터페이스의 모든 구조의 객체를 할당 할 수 있습니다. 이 slice와 원래의 slice는 차이가 있습니다.   
다음의 예를 살펴 보겠습니다.   
``` Go
package main
import "fmt"

type Human struct {
    name string
    age int
    phone string
}

type Student struct {
    Human         // 익명 필드
    school string
    loan float32
}

type Employee struct {
    Human         // 익명 필드
    company string
    money float32
}

// Human에 SayHi 메서드를 구현합니다.
func (h Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s \n", h.name, h.phone)
}

// Human에 Sing 메소드를 구현합니다.
func (h Human) Sing(lyrics string) {
    fmt.Println("La la la la ...", lyrics)
}

// Employee는 Human의 SayHi 메서드를 오버로드합니다.
func (e Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s \n", e.name,
        e.company, e.phone)
}

// Interface Men은 Human, Student 및 Employee로 구현 됩니다.
// 이 3 가지 형태는 다음의 두 가지 방법을 구현하기 때문입니다.
type Men interface {
    SayHi()
    Sing(lyrics string)
}

func main() {
    mike := Student{Human{"Mike", 25 "222-222-XXX"}, "MIT"0.00}
    paul := Student{Human{"Paul", 26 "111-222-XXX"}, "Harvard"100}
    sam := Employee{Human{"Sam", 36 "444-222-XXX"}, "Golang Inc."1000}
    tom := Employee{Human{"Tom"37 "222-444-XXX"}, "Things Ltd."5000}

    // Men 유형의 i 변수를 선언 합니다.
    var i Men

    // i는 Student를 저장할 수 있습니다.
    i = mike
    fmt.Println("This is Mike, a Student :")
    i.SayHi()
    i.Sing("November rain")

    // i는 Employee를 저장할 수도 있습니다.
    i = tom
    fmt.Println("This is Tom, an Employee :")
    i.SayHi()
    i.Sing("Born to be wild")

    // slice의 Men을 정의합니다.
    fmt.Println("Let 's use a slice of Men and see what happens")
    x := make([]Men 3)
    //이 3 개는 모두 다른 요소이지만, 같은 인터페이스를 구현하고 있습니다.
    x[0], x[1], x[2] = paul, sam, mike

    for _,  value := range x {
        value.SayHi()
    }
}
```

위의 코드에서 interface 메서드의 집합을 추상화 한 것을 알 수 있다고 생각합니다.    
다른 interface가 아닌 형태로 구현되어야하며, 자신이 구현할 수 없습니다.    
Go는 interface를 통해 `duck-typing`을 제공 합니다. 즉 "오리처럼 달리는 모습과 오리처럼 수영하고, 오리같이 울고,     
오리처럼 소리를 낸다면, 이 새는 오리다" 라는 것입니다.   

### 빈 interface
빈 interface (interface {})는 아무런 메소드도 포함되어 있지 않습니다. 이 거리 모든 형태는 하늘의 interface를 구현하고 있습니다. 빈 interface 자체는 아무 의미도 없습니다 (어떤 메소드도 포함되어 있지 않기 때문에)이 어떤 형태의 수치를 저장할 때 꽤 도움이들 수 있습니다. 이것은 어떤 형태의 수치를 저장할 수 있기 때문에 C 언어의 void * 형태와 비슷합니다.

// a를 빈 인터페이스로 정의
var a interface {}
var i int = 5
s : = "Hello world"
// a는 모든 유형의 값을 저장할 수 있습니다.
a = i
a = s

함수가 interface {}를 인수에 취하면 모든 형식의 값을 인수로 취할 수 있습니다. 만약 함수가 interface {}를 반환하면 모든 형식의 값을 반환 할 수 있습니다. 매우 편리하네요!
### interface 함수의 인수
interface 변수는이 interface 형식의 개체를 가질 수 있습니다. 그러면 함수 (메소드 포함)를 쓰는 경우 생각지도 못한 생각을 제공합니다. interface 인수를 정의하여 함수에 모든 형태의 인수를 받게 할 수 있습니다.

예를 들어 보자 : fmt.Println은 우리가 가장 자주 사용하는 함수입니다. 하지만 어떤 형태의 데이터를받을 수있는 점에주의습니까? fmt 소스 파일을 열면 이러한 정의를 담고 있습니다 :

type Stringer interface {
String () string
}
즉, String 메소드를 가지는 모든 형태가 fmt.Println 의해 호출 될 수있는 것입니다. 시험해 봅시다.

package main
import (
"fmt"
"strconv"
)

type Human struct {
name string
age int
phone string
}

//이 메소드를 사용해 Human에 fmt.Stringer을 구현합니다.
func (h Human) String () string {
return "❰"+ h.name + "-"+ strconv.Itoa (h.age) + "years - ✆"+ h.phone + "❱"
}

func main () {
Bob : = Human { "Bob", 39 "000-7777-XXX"}
fmt.Println ( "This Human is :"Bob)
}
이전 Box의 예를 생각해 봅시다. Color 구조체도 메소드를 하나 정의하고 있습니다 : String입니다. 사실 이것도 fmt.Stringer라는 interface를 구현하고있는 것입니다. 즉, 만약 어느 형태를 fmt 패키지로 특별한 형식으로 출력 시키려고 한 경우 Stringer 인터페이스를 구현해야합니다. 만약이 인터페이스를 구현하지 않으면 fmt는 기본 방식으로 출력합니다.

// 같은 기능을 구현합니다.
fmt.Println ( "The biggest one is", boxes.BiggestsColor () String ())
fmt.Println ( "The biggest one is", boxes.BiggestsColor ())

주 : error 인터페이스 객체 (Error () string 객체를 구현)를 구현합니다. fmt를 사용하여 출력을 할 경우 Error () 메소드가 호출됩니다. 따라서 String () 메소드를 재정의 할 필요가 없습니다.
### interface 변수를 저장하는 형식

interface 변수 중 일부는 모든 형태의 수치를 저장할 수있는 것을 배웠습니다 (이 형태는 interface를 구현하고 있습니다). 에서는이 변수에 실제로 저장되어있는 것은 어느 형태의 객체인지 어떻게 반대로 알 수 있을까요? 현재 두 가지 방법이 있습니다 :

- Comma-ok 주장

Go 언어의 문법은 변수가 어떤 형태인가 직접 판단하는 방법이 있습니다 : value, ok = element (T) 여기서 value는 변수의 값을 가리 킵니다. ok는 bool 형식입니다. element는 interface 변수입니다. T는 주장의 형태입니다.

만약 element에 T 형의 수치가 존재하고 있으면 ok는 true가 반환됩니다. 그렇지 않으면 false가 반환됩니다.

예를 상세하게 이해하고 갑시다.

package main

import (
"fmt"
"strconv"
)

type Element interface {}
type List [] Element

type Person struct {
name string
age int
}

// String 메소드를 정의합니다. fmt.Stringer을 구현합니다.
func (p Person) String () string {
return "(name :"+ p.name + "- age :"+ strconv.Itoa (p.age) + "years)"
}

func main () {
list : = make (List 3)
list [0] = 1 // an int
list [1] = "Hello"// a string
list [2] = Person { "Dennis", 70}

for index, element : = range list {
if value, ok : = element (int); ok {
fmt.Printf ( "list [% d] is an int and its value is % d \ n", index, value)
} else if value, ok : = element (string); ok {
fmt.Printf ( "list [% d] is a string and its value is % s \ n", index, value)
} else if value, ok : = element (Person); ok {
fmt.Printf ( "list [% d] is a Person and its value is % s \ n", index, value)
} else {
fmt.Println ( "list [% d] is of a different type", index)
}
}
}

매우 간단하네요. 이전 흐름의 항목 중에 소개 한대로 여러 if에서 변수의 초기화가 허용되고 있는데 기분 나무입니까?

또한 주장의 형태가 증가하면 증가할수록, ifelse의 수도 증가하는데 기억할지 모릅니다. 아래에서는 switch를 소개합니다.
- switch 테스트

코드의 예를 보여 편이 빠르 겠죠. 위의 구현을 다시 쓰고 다시 시도합니다.

package main

import (
"fmt"
"strconv"
)

type Element interface {}
type List [] Element

type Person struct {
name string
age int
}

// 프린트
func (p Person) String () string {
return "(name :"+ p.name + "- age :"+ strconv.Itoa (p.age) + "years)"
}

func main () {
list : = make (List 3)
list [0] = 1 // an int
list [1] = "Hello"// a string
list [2] = Person { "Dennis", 70}

for index, element : = range list {
switch value : = element (type) {
case int :
fmt.Printf ( "list [% d] is an int and its value is % d \ n", index, value)
case string :
fmt.Printf ( "list [% d] is a string and its value is % s \ n", index, value)
case Person :
fmt.Printf ( "list [% d] is a Person and its value is % s \ n", index, value)
default :
fmt.Println ( "list [% d] is of a different type", index)
}
}
}

여기서 강조하고 싶은 것은`element (type)`라는 문법은 switch의 외부 로직에서 사용할 수 없다는 것입니다. 만약 switch 밖에서 형태를 판단하려면`comma-ok`을 사용하십시오.

### 내장 interface
Go 정말 매력적인 것은 기본 논리 문법입니다. Struct를 배운 때 익명 필드는 그렇게 우아했다. 에서는 유사한 논리를 interface에 도입하면 더 완벽하게됩니다. 만약 interface1이 interface2 내장 필드이면 interface2는 암묵적으로 interface1의 메소드를 포함 할 수 있습니다.

소스 패키지 container / heap에 이러한 정의가있는 것을 확인할 수 있다고 생각합니다.

type Interface interface {
sort.Interface // 기본 제공 필드 sort.Interface
Push (x interface {}) // a Push method to push elements into the heap
Pop () interface {} // a Pop elements that pops elements from the heap
}

sort.Interface는 사실 기본 제공 필드입니다. sort.Interface의 모든 메소드를 암시 적으로 포함합니다. 즉 다음의 세 가지 방법입니다.

type Interface interface {
// Len is the number of elements in the collection.
Len () int
// Less returns whether the element with index i should sort
// before the element with index j.
Less (i, j int) bool
// Swap swaps the elements with indexes i and j.
Swap (i, j int)
}

또 다른 예는 io 패키지 안에있는 io.ReadWriter입니다. 이 가운데는 io 패키지의 Reader와 Writer 두 interface를 포함하고 있습니다 :

// io.ReadWriter
type ReadWriter interface {
Reader
Writer
}

### 리플렉션
Go는 리플렉션을 구현하고 있습니다. 리플렉션은 프로그램의 런타임 상태를 검사 할 수 있습니다. 우리가 일반적으로 사용하고있는 패키지는 reflect 패키지입니다. 어떻게 reflect 패키지를 사용하는지는 공식 문서에 자세한 원리가 설명되어 있습니다. [laws of reflection (http://golang.org/doc/articles/laws_of_reflection.html)

reflect를 사용하려면 3 단계로 나눌 수 있습니다. 아래에 간단하게 설명합니다 : 반사 형태의 값 (이 값은 빈 인터페이스를 구현하고 있습니다.) 우선 이것을 reflect 객체로 변환해야합니다 (reflect.Type 또는 reflect.Value입니다. 다른 상황에 따라 다른 함수를 호출합니다.)이 두 가지를 얻을 수있는 방법은 :

t : = reflect.TypeOf (i) // 원래 데이터를 가져옵니다. t 통해 형식 정의에서 모든 요소를​​ 검색 할 수 있습니다.
v : = reflect.ValueOf (i) // 실제 값을 가져옵니다. v 통해 저장되고있는 가운데 값을 얻을 수 있습니다. 값을 변경할 수 있습니다.

reflect 객체로 변환 한 후 뭔가 작업을 할 수 있습니다. 즉, reflect 객체를 대응하는 값으로 변환하는 것입니다. 예

tag : = t.Elem () Field (0) .Tag // struct에서 정의 된 태그를 취득한다.
name : = v.Elem () Field (0) .String () // 첫 번째 필드에 저장되어있는 값을 얻을 수 있습니다.

reflect 값을 검색하여 해당 형식과 값을 반환 할 수 있습니다.

var x float64 = 3.4
v : = reflect.ValueOf (x)
fmt.Println ( "type :"v.Type ())
fmt.Println ( "kind is float64 :"v.Kind () == reflect.Float64)
fmt.Println ( "value :"v.Float ())

마지막으로 반사 한 필드는 수정할 수 있어야합니다. 앞에서 배운 값 전달과 참조에 같은 이치입니다. 리플렉션 필드를 반드시 읽고 쓸 수있는 것은 다음과 같이 쓸 경우 오류가 발생하는 것입니다.

var x float64 = 3.4
v : = reflect.ValueOf (x)
v.SetFloat (7.1)

만약 해당하는 값을 변경하려면 이렇게 써야합니다.

var x float64 = 3.4
p : = reflect.ValueOf (& x)
v : = p.Elem ()
v.SetFloat (7.1)

위는 리플렉션에 대한 간단한 설명이 있지만 더 깊은 이해는 실제 프로그래밍에서 실천해 나가는 다른 없습니다.
