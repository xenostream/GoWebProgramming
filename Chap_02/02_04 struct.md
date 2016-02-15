---
layout: post
title: 'Go Web Programming: [02/04] Struct'
tags: goweb
---  


# Struct

Go 언어는 C나 다른 언어와 마찬가지로 서로 다른 형태의 속성이나 필드의 컨테이너로 새로운 형태를   
선언 할 수 있습니다. 예를 들어, 개인의 엔티티를 나타내는 `person` 형을 만들 수 있습니다.    
이 엔티티는 속성을 가지고 있습니다.  성별과 나이 입니다. 이러한 형식은`struct`이라고 합니다.  
다음은 예제 코드입니다.   
```go
type person struct {
    name string
    age int
}
```
struct 선언은 이렇게 간단히 끝이 납니다. 위의 struct형은 2개의 필드를 가지고 있습니다.   
- string 형의 name 필드는 사용자의 이름을 저장하는 속성 입니다.
- int 형 필드 age는 사용자의 나이를 저장하는 속성 입니다.

이제 struct이 어떻게 사용되는지를 코드를 통해서 알아보도록 하겠습니다.   
```go
type person struct {
    name string
    age int
}

var P person            // P는 person형 변수입니다.

P.name = "Astaxie"      // "Astaxie"를 변수 P의 name 속성에 할당 합니다.
P.age = 25              // "25"를 변수 P의 age 속성에 할당 합니다.
fmt.Printf("The person 's name is %s", P.name)   // P의 name 속성에 액세스 합니다.
```

위와 같은 방법외에도, 다른 여러 가지 선언 방법이 있습니다.   

- 1. 순서에 따라 초기화한다.   
  P := person{"Tom", 25}

- 2.`field : value` 방법으로 초기화합니다. 이 경우 순서는 임의로 설정됩니다.   
  P := person{age: 24, name: "Tom"}

- 3. 물론`new` 함수를 통해서 포인터로 만들 수 있습니다. 이 P의 형태는 *person 입니다.  
  P := new(person)

다음의 예제를 통해서 일반적인 struct의 사용법에 대해서 설명합니다.     
```go
package main
import "fmt"

// 새로운 형태를 선언합니다.
type person struct {
    name string
    age int
}

// 두 사람의 나이를 비교합니다. 나이가 큰 사람을 돌려주고, 또한 나이 차이도 반환합니다.
// struct도 값전달 방식으로 전달 합니다.
func Older(p1, p2 person) (person, int) {
    if p1.age > p2.age {       // p1과 p2의 두 사람의 나이를 비교합니다.
    return p1, p1.age - p2.age
    }
    return p2, p2.age - p1.age
}

func main() {
    var tom person

    // 초기 값을 할당 합니다.
    tom.name, tom.age = "Tom", 18

    // 2 개의 필드를 명확히 초기화 합니다.
    bob := person{age: 25, name: "Bob"}

    // struct 정의의 순서대로 초기화 합니다.
    paul := person{"Paul", 43}

    tb_Older, tb_diff := Older(tom, bob)
    tp_Older, tp_diff := Older(tom, paul)
    bp_Older, bp_diff := Older(bob, paul)

    fmt.Printf("Of %s and %s %s is older by %d years \n"
        tom.name, bob.name, tb_Older.name, tb_diff)

    fmt.Printf("Of %s and %s %s is older by %d years \n"
        tom.name, paul.name, tp_Older.name, tp_diff)

    fmt.Printf("Of %s and %s %s is older by %d years \n"
        bob.name, paul.name, bp_Older.name, bp_diff)
}
```


### struct 익명 필드
위에서 struct를 어떻게 선언하는지 설명 했습니다. 선언 할 때 필드 이름과 형태가 하나 하나 대응하고 있습니다.   
사실 Go는 형태만 정의하는 것도 지원하고 있습니다. 이것은 필드 이름을 쓰지 않는 방법으로, 익명 필드라고 합니다.     
기본 제공 필드라고도 합니다.

익명 필드가 struct 인 경우,이 struct가 가진 모든 필드는 숨겨져 현재 정의하고 있는 struct에 포함됩니다.  
예를 들어서 설명하겠습니다.    
```go
package main
import "fmt"

type Human struct {
    name string
    age int
    weight int
}

type Student struct {
    Human          // 익명필드, 기본적으로 Student는 Human 모든 필드를 포함 할 수 있습니다.
    speciality string
}

func main() {
    // 학생 한 명을 초기화 합니다.
    mark := Student{Human{"Mark", 25, 120}, "Computer Science"}

    // 해당 필드에 액세스합니다.
    fmt.Println("His name is", mark.name)
    fmt.Println("His age is", mark.age)
    fmt.Println("His weight is", mark.weight)
    fmt.Println("His speciality is", mark.speciality)
    // 해당 메모 정보를 수정합니다.
    mark.speciality = "AI"
    fmt.Println("Mark changed his speciality")
    fmt.Println("His speciality is", mark.speciality)
    // 나이 정보를 수정합니다.
    fmt.Println("Mark become old")
    mark.age = 46
    fmt.Println("His age is", mark.age)
    // 체중 정보를 수정합니다.
    fmt.Println( "Mark is not an athlet anymore")
    mark.weight += 60
    fmt.Println("His weight is", mark.weight)
}
```

![](/images/2.4.student_struct.png)   
그림 2.7 Student와 Human 상속 

Student가 age와 name 속성에 액세스 할 때,  마치 자신의 필드 인 것처럼 접근 한 것을 볼 수 있습니다.    
그렇습니다. 이것이 바로 익명 필드란 것입니다. 필드의 상속을 실현할 수 있습니다. 
이건 OOP언어보다 간단하지 않습니까? 여기에 더 멋진 방법도 있습니다. student는 Human 필드 이름에서 액세스    
할 수 있습니다. 아래의 코드를 참조 하십시오.   
```go
mark.Human = Human{"Marcus", 55, 220}
mark.Human.age -= 1
```
익명필드의 액세스 및 수정은 매우 간단합니다. 하지만 단순한 struct 필드이기 때문에 Go언어의 모든 내장형과   
자신이 정의한 모든형을 익명 필드로 사용할 수 있습니다. 아래의 예를 참조하시기 바랍니다.       
```go
package main
import "fmt"

type Skills[] string

type Human struct {
    name string
    age int
    weight int
}

type Student struct {
    Human            // 익명 필드 struct
    Skills           // 익명 필드 내에서 정의 된 형. string slice
    int              // 내장형 익명 필드
    speciality string
}

func main() {
    // 학생 Jane을 초기화 합니다.
    jane := Student{Human: Human{"Jane", 35, 100}, speciality: "Biology"}
    // 해당 필드에 액세스 
    fmt.Println("Her name is", jane.name)
    fmt.Println("Her age is", jane.age)
    fmt.Println("Her weight is", jane.weight)
    fmt.Println("Her speciality is", jane.speciality)
    // 그의 skill 기능 필드를 수정합니다.
    jane.Skills = []string{ "anatomy"}
    fmt.Println("Her skills are",jane.Skills)
    fmt.Println("She acquired two new ones")
    jane.Skills = append(jane.Skills, "physics", "golang")
    fmt.Println("Her skills now are"jane.Skills)
    // 익명 내장 필드를 수정합니다.
    jane.int = 3
    fmt.Println("Her preferred number is", jane.int)
}
```

위의 같이, struct는 struct를 익명필드뿐 만 아니라, 직접 정의한 형(Type)또는,  내장형도 익명필드로 처리    
할 수 있습니다. 또한 해당 필드에서 직접  함수를 실행 할 수 있습니다 (예를 들어 상기의  append 입니다).

여기에는 한 가지 의문사항이 있습니다. 만약 human에 phone이라는 필드가 있다면 student에도 phone 이라는    
필드가 있습니다. 이것은 어떻게 처리 해야 할까요?  

Go는 쉽게 이 문제를 해결할 수 있습니다. 우선 외부가 먼저 액세스되기 때문에 `student.phone`에 액세스 한 경우   
st​​udent 내의 필드에 액세스하고,  human 필드에 액세스하지 않습니다.   

이처럼 익명 필드를 통해 필드상속을 쉽게 처리 할 수 있습니다. 익명형의 필드에 접근하고 싶다면 익명필드의 이름으로   
액세스 할 수 있습니다. 아래의 예를 참조 하시기 바랍니다.    
```go
package main
import "fmt"

type Human struct {
    name string
    age int
    phone string         // Human 형이 가지는 필드
}

type Employee struct {
    Human               // 익명 필드 Human
    speciality string
    phone string        // 직원 phone 필드
}

func main() {
    Bob := Employee{Human{"Bob", 34 "777-444-XXXX"}, "Designer", "333-222"}
    fmt.Println("Bob 's work phone is :", Bob.phone)
    // 만약 Human의 phone 필드에 액세스하는 경우
    fmt.Println("Bob 's personal phone is :", Bob.Human.phone)
}
```

