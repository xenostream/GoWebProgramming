---
layout: post
title: 'Go Web Programming: [02/05] Object Oriented'
tags: goweb
---  

# Object Oriented

앞의 두 장에서 함수와 struct를 설명 했습니다. 함수를 struct의 필드로 처리할 수 있습니다.   
이번장에서는 함수의 또 다른 형태에 대해서  설명 합니다.
다른 객체지향프로그래밍 언어에서는 이 기능을 `method`이라고 부르고 있는 기능입니다. 

## method
사각형이라는 struct를 정의하고 이 사각형의 넓이는 구하는 프로그램을 작성하고 있다고 가정하겠습니다.     
먼저, 일반적인 사고방식으로 접근하면 다음과 같이 프로그래밍 하게 됩니다.     
```go
package main
import "fmt"

type Rectangle struct {
    width, height float64
}

func area(r Rectangle) float64 {
    return r.width * r.height
}

func main() {
    r1 := Rectangle{12, 2}
    r2 := Rectangle{9, 4}
    fmt.Println("Area of​​ r1 is :", area(r1))
    fmt.Println("Area of​​ r2 is :", area (r2))
}
```
상기의 코드는 직사각형의 면적을 구할 수 있겠지만, area() 함수는 Rectangle의 구조체와 연관된 함수라고 볼 수 없습니다.   
Rectangle 객체(여기서는 r1, r2)를  면적을 계산하는 함수에 그저 인수로 전달하고 있을 뿐입니다.   

이렇게 구현해도 상관 없지만 여러가지 면적을 구할려면, 예를들어 사각형, 오각형 , 다각형에 대한 면적을 구해야하는    
상황이 발생하면 어떻게 처리해야 할까요? 이런 경우의 해결책은 해당 처리 함수를 늘릴 수밖에 없습니다.   
또한 함수 이름도 개별적으로 준비해야 합니다. `area_rectangle, area_circle, area_triangle ...` 식으로...   

아래의 그림은 타원 함수를 나타내고 있습니다. 이 함수는 struct에 속하지 않기(객체지향 용어로, 즉 class에 속하지 않는)    
때문에 struct 외부에 별도로 존재하고,  개념적으로도  어떤 struct과도 연결되지 않는 일반적인 함수 입니다.    

![](/images/2.5.rect_func_without_receiver.png)  
그림 2.8 메소드와 struct의 관계도  

분명 이런 구현 방법에는 무리가 있습니다. 또한 개념적으로도, "면적"은 "형상"의 한 속성 입니다.    
이 속성은 특정 형상과 연관되어져 있습니다. 직사각형의 가로와 세로와 같이 서로 뗄레야 뗄수 없는 존재인 것 입니다.   

이런 이유로 `method`라는 개념이 생겨 났습니다. 즉, `method`은 어느 형태와 연관되어 존재한다 라는 의미 입니다.        
`method` 선언 문법과 함수 선언 문법은 거의 동일 합니다.   
단지, `func` 키워드 뒤에 receiver(method가 어디에 속한지를 표시)를 추가해서 선언 합니다.    

위에서 언급 한 형상 예제로 보면 `area()` 함수(methos)는 형상(Rectangle)에 연관되어서 작동한다는 것에서 유래 합니다.    
Rectangle.area()의 의미는 Rectangle형상에 속한 area() 함수라는 의미입니다. 
Rectangle에 속하는 메소드는 구조체 외부에 선언하고, 리시버를 통해서 해당 `구조체의 함수`로 연관짓게 됩니다.     

더 구체적으로 말하면, Rectangle은 length와 width 필드만 존재 합니다.또한, 동시에 area() 메소드가 존재 합니다.  
이런 필드 및 메소드는 모두 Rectangle에 속한다 라는 의미입니다. 

Go언어의 개발자 중 하나인 `Rob Pike`의 말을 잠시 빌리면 다음과 같습니다.       

> "A method is a function with an implicit first argument, called a receiver."

method의 문법은 다음과 같습니다. 
```go
func (r ReceiverType) funcName(parameters) (results)
```
이전에 설명한 예제를 가지고 method를 설명하겠습니다.     
```go
package main

import (
    "fmt"
    "math"
)

type Rectangle struct {
    width, height float64
}

type Circle struct {
    radius float64
}

func (r Rectangle) area() float64 {
    return r.width * r.height
}

func (c Circle) area() float64 {
    return c.radius * c.radius * math.Pi
}


func main() {
    r1 := Rectangle{12, 2}
    r2 := Rectangle{9, 4}
    c1 := Circle{10}
    c2 := Circle{25}

    fmt.Println("Area of​​ r1 is :", r1.area())
    fmt.Println("Area of​​ r2 is :", r2.area())
    fmt.Println("Area of​​ c1 is :", c1.area())
    fmt.Println("Area of​​ c2 is :", c2.area())
}
```

method를 사용할 때는 다음의  몇 가지를 주의하시기 바랍니다.   

- method는 동일한 이름이라도 수신기가 다르면 method도 다릅니다.
- method는 수신기의 필드에 액세스 할 수 있습니다.
- method의 호출은 `.` 통해 액세스 합니다. struct이 필드에 액세스하는 것과 동일 합니다.



![](/images/2.5.shapes_func_with_receiver_cp.png)   
그림 2.9 다른 struct의 method는 다르다.    

위의 예에서 method area()는 각각 Rectangle와 Circle에 속한 메소드가 됩니다. 이때 Rectangle과 Circle이   
리시버가 됩니다. 또한 area() 메소드는 Rectangle / Circle에게 속하게 되고, 사용되어 집니다.      

> method는 점선으로 표시하고 있습니다. 이것은 메소드의 리시버는 값에 의한 전달이며, 참조가 아닙니다.    

  리시버를 포인터로 처리해도 문제 없습니다. 차이점은 포인터는 리시버가 엔터티의 내용에 직접 수정할 수 있는 반면,       
  일반적인 방법은 리시버의 조작은 값의 복사본을 조작하는 것입니다. 원래 엔터티에 대한 조작이 발생하지 않는 것입니다.       
  
method는 struct 에서만 사용할 수 있는 것일까요? 당연히 아닙니다!!   
메소드는 사용자 정의형, 내장형, struct 등 모든 형으로도 선언할 수 있습니다. 조금 헷갈리시는가요?    
사용자 정의형이 struct 아닌가요? struct는 사용자 정의형 중에서도 비교적 특수한 형태의 타입일 뿐입니다.   
다음과 같은 선언을 제공하고 있습니다.    
```
type typeName typeLiteral
```
다음의 예제는 사용자 정의형을 선언하는 예제입니다.    
```go 
type ages int

type money float32

type months map [string] int

m := months {
    "January": 31
    "February": 28,
    ...
    "December": 31
}
```

사용법을 아시겠습니까? 코드에 형의 이름을 별도로 지정해서 가독성을 좋게 만드는것 일 뿐입니다.    
실제로는 별칭을 정의해서 사용하는 것 뿐입니다. C의 `typedef`과 유사한 것으로,     
상기의 예제에서는 ages는 int형의 동일키워드가 되는 것 이상의 의미는 없습니다.    

그럼다시, `method` 설명으로 돌아와서, 사용자 정의형을 임의의 `method`와 연관지을 수 있습니다.   
다음은 이러한 기능을 하는 조금 복잡한 예제를 살펴보겠습니다.     
```go
package main
import "fmt"

const (
    WHITE = iota
    BLACK
    BLUE
    RED
    YELLOW
)

type Color byte

type Box struct {
    width, height, depth float64
    color Color
}

type BoxList[] Box       // a slice of boxes

func (b Box) Volume() float64 {
    return b.width * b.height * b.depth
}

func (b * Box) SetColor(c Color) {
    b.color = c
}

func (bl BoxList) BiggestColor() Color {
    v := 0.00
    k := Color(WHITE)
    for _,  b := range bl {
        if bv := b.Volume(); bv > v {
            v = bv
            k = b.color
        }
    }
    return k
}

func (bl BoxList) PaintItBlack() {
    for i,  _ := range bl {
        bl[i].SetColor(BLACK)
    }
}

func (c Color) String() string {
    strings := []string{ "WHITE", "BLACK", "BLUE", "RED", "YELLOW"}
    return strings [c]
}

func main() {
    boxes := BoxList {
                Box{4, 4, 4, RED}
                Box{10, 10, 1, YELLOW}
                Box{1, 1, 20, BLACK}
                Box{10, 10, 1, BLUE}
                Box{10, 30, 1, WHITE}
                Box{20, 20, 20, YELLOW}
    }

    fmt.Printf("We have %d boxes in our set \n", len(boxes))
    fmt.Println("The volume of the first one is", boxes[0].Volume(),  "cm³")
    fmt.Println("The color of the last one is", boxes[len(boxes) -1].color.String())
    fmt.Println("The biggest one is", boxes.BiggestColor().String())

    fmt.Println("Let 's paint them all black")
    boxes.PaintItBlack()
    fmt.Println("The color of the second one is", boxes[1].color.String())

    fmt.Println("Obviously, now, the biggest one is", boxes.BiggestColor().String())
}
```

위의 코드는 const 키워드로 몇개의 정수상수를 정의한 후 몇가지의 사용자 정의형을 선언하고 있습니다.  

- Color는 byte의 별칭 입니다.
- struct : Box를 선언 합니다. 3 개의 가로, 세로, 높이 필드와 색상 속성을 가지고 있습니다.
- slice :  BoxList을 선언 합니다. Box를 가지고 있습니다.

다음으로 사용자 정의형을 수신기로 method를 선언 합니다.  

- Volume() 수신기를 Box로 선언 합니다. Box의 부피를 반환 합니다.
- SetColor(c Color)는 Box의 색을 c로 변경 합니다.
- BiggestColor()는 BoxList에 선언되어 있습니다.    
- PaintItBlack()는 BoxList의 모든 Box의 색을 모두 검정색으로 변경 합니다.
- String()는 Color로 정의되어 있으며, Color의 구체적인 색을 돌려줍니다 (문자열 형식)

위의 코드를 문자로 표현하면 쉽게 생각됩니다. 문제를 해결할 경우 문제를 묘사한 후 해당 코드를 작성하여 제공 합니다.   


### 포인터 receiver
그럼 여기서, SetColor 구현방법을 검토해 보겠습니다.   
이 receiver는 Box의 포인터를 사용하고 있습니다. *Box를 사용 하고 있습니다. 왜 Box가 아닌 포인터를 사용하는 것일까요?  
SetColor를 선언하는 진짜 목적은 실제 Box의 색을 변경하는 것입니다. 
만약 Box의 포인터를 전달하지 않았다면, SetColor가 받는 것은 사실 Box의 복사인 것입니다. 즉, 메소드 내에서 색상을     
변경하는 것은 Box의 복사본을 조작하는 것일뿐, 실제 Box에는 아무런 변화가 없습니다. 따라서 포인터를 전달해야 합니다.   

여기서는 receiver를 함수선언의 첫번째로 기술했습니다. 이것은 이전의 함수 설명에서 값전달과 참조도 쉽게 처리됩니다.    
혹시 SetColor 함수에서 다음과 같이 사용해야 하는게 아닐까라는 생각이 생길지도 모릅니다. `*b.Color = c`   
그런데 실제로는 `b.Color = c`로 사용했습니다. 포인터에 대응하는 값을 읽을 수 있어야하기 때문입니다. 

Go에서는 이 두 가지 방법 모두 맞습니다.    
포인터를 사용하여 해당 필드에 액세스 한 경우(포인터에 아무런 필드가 없었다고해도) Go는 포인터를 통해 그 값에   
액세스하려는 것을 이미 알고 있습니다. 주의 깊은 개발자는 이렇게 생각할지도 모릅니다. PointItBlack에서 SetColor를    
호출했을 때, 혹시 `(&bl[i]).SetColor(BLACK)`로 써야 하지 않을까? 라고 말입니다.     
SetColor의 receiver는 *Box이지 Box가 아니니까요. 하지만, 이 두 가지 방법은 어떤것을 사용해도 모두 괜찮습니다.    
Go는 receiver가 포인터임을 이미 알고 있습니다. 그래서 내부적으로 자동으로 해석해서 처리해 주는 것입니다.

> 만약 메소드의 receiver가 `*T`라면 T형 엔티티의 변수 V 에서 이 메소드를 호출 할 수 있습니다. 
  이때 &V 의해 메소드를 호출 할 필요는 없습니다.

마찬가지로, 

> 만약 메소드의 receiver가 T면 `*T`형의 변수 P에서 이 메소드를 호출 할 수 있습니다.    
  *P를 사용하여 메소드를 호출 할 필요는 없습니다.    

결과적으로 호출하고있는 메소드가 포인터의 메소드인지 여부는 신경 쓸 필요가 없습니다. Go 당신이 시도하고있는 모든 것을  
이미 알고있는 것입니다. C/C++ 프로그램을 작성해본 경험 있다면, 아주 아주 큰 고통이 해결 된 것입니다.

### method 상속
이전 장에서는 필드의 상속에 대해서 배웠습니다. 마찬가지로!!!     
> method도 상속 할 수 있습니다.    
만약 익명필드가 한개의 메소드를 구현하는 경우이 익명 필드를 포함해서 sturct이 메소드를 호출 할 수 있습니다.      
다음의 예제를 통해서 알아보도록 하겠습니다. 
```go
package main
import "fmt"

type Human struct {
    name string
    age int
    phone string
}

type Student struct {
    Human            // 익명 필드
    school string
}

type Employee struct {
    Human             // 익명 필드
    company string
}

// human에서 메소드를 정의
func (h * Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s \n", h.name, h.phone)
}

func main() {
    mark := Student{Human{"Mark", 25 "222-222-YYYY"}, "MIT"}
    sam := Employee{Human{"Sam", 45 "111-888-XXXX"}, "Golang Inc"}

    mark.SayHi()
    sam.SayHi()
}
```


### method 재 작성
위의 예에서 만약 Employee에서 SayHi을 구현하려면 어떻게하면 좋을까요?  간단합니다.    
익명필드의 충돌처리와 같은 이치로, Employee에서도 메소드를 재 정의 할 수 있습니다. 익명 필드를 다시 작성하는    
방법은 아래의 예를 참조 하십시오.
```go
package main
import "fmt"

type Human struct {
    name string
    age int
    phone string
}

type Student struct {
    Human           // 익명 필드
    school string
}

type Employee struct {
    Human           // 익명 필드
    company string
}

// Human에 method를 정의
func (h *Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s \n", h.name, h.phone)
}

// Employee의 method로 Human의 method를 다시 작성
func (e * Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s \n", e.name,
            e.company, e.phone) // 2줄로 나눠서 작성해도 됩니다!!!
}

func main() {
    mark := Student{Human{"Mark", 25 "222-222-YYYY"}, "MIT"}
    sam := Employee{Human{"Sam", 45 "111-888-XXXX"}, "Golang Inc"}

    mark.SayHi()
    sam.SayHi()
}
```

위의 코드와 같이 Go언어는 아주 유연하게 디자인 되어 있습니다.    
이런 지식으로 기본적인 객체 지향 프로그램을 설계 할 수 있습니다. Go객체지향은 이렇게 간단히 구현됩니다.     

public / private 라는 키워드는 필요하지 않습니다. 대문자와 소문자를 사용해서 처리하고 있는 것입니다.    
즉, 대문자로 시작하면 외부에 공개하고, 소문자로 시작하면 비공개입니다. 메서드또한 동일한 규칙이 적용 됩니다.

