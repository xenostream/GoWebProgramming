# Control Statements and Functions

이 절에서는 Go 언어의 흐름 제어 및 함수에 대해 소개 합니다.

## 흐름 제어
흐름 제어는 프로그램 언어 중 가장 중요한 기능 중 하나입니다. 왜냐하면 흐름제어로 매우 간단한 로직에서 부터      
매우 복잡한 로직을 표현 할 수 있기 때문 입니다. Go언어의 흐름 제어는 크게 세 부분으로 구성되어 있습니다.   
 조건문, 판단문, 루프 제어 및 무조건 점프 입니다.   

### if
`if`는 모든 프로그래밍 언어에서 가장 자주 사용하는 구문중 하나일 것입니다.    
이 문법을 대략적으로 설명하면, 만약 조건을 만족하면 무엇을하고 만족하지 않으면 어떤행동을 할지를 결정하는 것입니다.     

Go 언어에서는 `if` 분기문의 특징은 괄호로 비교조건을 묶을 필요가 없습니다. 다음 코드를 참고 하시기 바랍니다.   
``` Go
if x> 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is less than 10")
}
```

Go의`if`문의 장점중 하나는 조건 분기에서 변수를 선언 할 수 있습니다.    
이 이렇게 선언한 변수의 범위는 선언된 블록내에만 존재하며 다른 곳에서는 사용할 수 없습니다.  

``` Go
// x 값을 계산한 후 x의 크기를 반환 합니다. 예제에서는 10 이상 여부를 판단 합니다.
if x := computedValue(); x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is less than 10")
}

// 여기에서 만약 이렇게 호출하면 컴파일 오류 입니다. x는 조건문 내에서만 사용하는 변수이기 때문입니다.    
fmt.Println(x)
```
또한, 여러가지 조건별로 검사를 할 수도 있습니다. 예제는 다음과 같습니다.    
``` Go
if integer == 3 {
    fmt.Println("The integer is equal to 3")
} else if integer < 3 {
    fmt.Println("The integer is less than 3")
} else {
    fmt.Println("The integer is greater than 3")
}
```

### goto

Go언어에서는 `goto` 키워드가 있습니다 

> 반드시 사용할 곳에만 현명하게 사용하시기 바랍니다. 

 `goto` 문은 반드시 함수 내에서 미리 정의 된 태그(레이블)로 이동 합니다.   
예를 들어 이러한 루프가 있다고 가정합니다.    
``` Go
func myFunc() {
    i := 0
Here :           //행의 처음에서 시작하고 콜론을 마지막에 사용하여 태그로 표시합니다.     
    println(i)
    i++
    goto Here   // Here로 이동합니다.
}
```
> 태그 이름은 대소문자를 구별 합니다.

### for
Go언어에서 가장 강력한 컨트롤 문장이라면 단연, `for` 문장입니다. 

> Go언어의 반복문은 `for`문 하나 뿐입니다. 
   
이것은 반복문 내에서, 데이터를 모두 읽어 내는 용도로 사용할 수 있습니다.     
일반적인 언어에서 `while`문장을 대신해서 사용한다고 생각하시면 됩니다.  문법은 다음과 같습니다.   

```
for expression1; expression2; expression3 {
// ...
}
```
`expression1`, `expression2`, `expression3`은 식 입니다.    
`expression1`과 `expression3`은 변수 선언이나 함수 호출의 반환 값을 사용할 수 있습니다.    
`expression2`는 반복 의사 결정에 사용 됩니다. 이 문장의 흐름은 다음과 같습니다.     
`expression1`문장은 루프가 시작되기 전에 호출 됩니다. `expression3` 반복 할 때 마지막에  호출 됩니다.  
 
이렇게 얘기하는 것보다, 예제를 보는 편이 빠르겠습니다.     

``` Go
package main
import "fmt"

func main() {
    sum := 0;
    for index := 0; index < 10; index++ {
    sum += index
    }
    
    fmt.Println("sum is equal to", sum)
}
```

`sum is equal to 45`


때로는 여러개의 변수할당을 할 경우가 있습니다. Go언어에는  콤마연산자(,)가 없기 때문에 병행할당을 이용할 수 있습니다.   

`i, j = i + 1, j-1`


또한, `expression1`과 `expression3`를 생략 할 수 도 있습니다.     
``` Go
sum := 1
for ; sum <1000 ; {
    sum += sum
}
```
이 예제에서는 `;`을 생략 할 수 있습니다. 다음과 같은 코드는 동일한 기능을 수행 합니다.    
어디선가 낯이 익지 않습니까? 바로 `while` 문의 기능입니다.      
``` Go
sum := 1
for sum < 1000 {
    sum += sum
}
```

반복문에서는 `break`와`continue` 두 가지 명령을 사용할 수 있습니다.    
`break`는 현재 루프에서 탈출하며, `continue`는 조건 검사로 점프 합니다.    
중첩 된 반복문의  경우`break` 명령으로, 한번에 중첩 반복문을 탈출 할 수 있습니다.     
다음은 그 사용 예제입니다.       
``` Go
for index := 10; index > 0; index-- {
    if index == 5 {
        break      // 또는 continue
    }
    fmt.Println(index)
}
// break문이라면  10,9,8,7,6이 출력 됩니다.
// continue문의 경우 10,9,8,7,6,4,3,2,1이  출력 됩니다.
```

`break`와`continue` 는 이름(태그)을 지정해서 사용할 수 있습니다.    
여러번 중첩된 루프에서 바깥쪽 루프로 한번에 점프하는 데 사용 됩니다. (break goto_label)

`for`문장의 주요 용도는 `range` 키워드와 함께 `slice`와`map`의  데이터를 액세스 할 경우 입니다.     
```
for k, v := range map {
    fmt.Println("map 's key :"k)
    fmt.Println("map 's val :", v)
}
```

Go는 "여러개의 반환 값"을 반환하는 기능을 제공합니다. 하지만, 선언만하고 사용하지 않는 변수는 컴파일러 오류를    
출력 합니다. 이런 상황에서 `_`를  사용하여 필요없는 반환 값을 무시할 수 있습니다.    

``` Go
for _ , v := range map {
    fmt.Println("map 's val :", v)
}
```
상기의 코드에서는 `_`문자로 값을 무시했으므로, 컴파일 오류가 발생하지 않습니다.   


### switch
수 많은 `if-else`를 작성하여 로직 처리해야하는 경우가 종종 있습니다.     
이런 많은류의 분기 코드는 보기에 혹은 읽기에 매우 좋지 않습니다. 또한 유지 보수도 쉽지 않게 되므로,   
`switch`문을 사용하면 간단히 해결 할 수 있습니다. 이 문법은 다음과 같습니다.    

``` Go
switch sExpr {
    case expr1:
        some instructions
    case expr2:
        some other instructions
    case expr3:
        some other instructions
    default:
        other code
}
```
`sExpr`과 `expr1``expr2``expr3`의 유형은 정확히 일치해야 합니다.    
Go의`switch`문은 매우 사용하기 편리하며, 식(expression)은 반드시 정수만 사용할 수 있는것이 아닙니다.    
실행 과정은 위에서 아래까지 case문장중 일치하는 항목을 찾을 때까지 수행됩니다.    
만약 `switch`에 일치하는 식이 없으면, `default` 문장을 수행합니다.    
``` Go 
i := 10
switch i {
    case 1:
        fmt.Println("i is equal to 1")
    case 2, 3, 4:
        fmt.Println("i is equal to 2, 3 or 4")
    case 10:
        fmt.Println("i is equal to 10")
    default:
        fmt.Println( "All I know is that i is an integer")
}
```
상기 예제에서는 7번째 줄의 case문이 수행 됩니다.  
Go의 `switch`은 기본적으로`case`의 마지막에 `break`문이 있는 것으로 간주하기 때문에,    
case를 실행한 후에 곧바로 switch문을 벗어나게 됩니다. 만약, `fallthrough` 키워드를 사용한다면,        
해당 case 코드를 수행한 후 다음 case문을 수행할 수도 있습니다. 다음은 그 사용 예제입니다.    
```
integer := 6
switch integer {
    case 4:
        fmt.Println("The integer was <= 4")
        fallthrough
    case 5:
        fmt.Println("The integer was <= 5")
        fallthrough
    case 6:
        fmt.Println("The integer was <= 6")
        fallthrough
    case 7:
        fmt.Println("The integer was <= 7")
        fallthrough
    case 8:
        fmt.Println("The integer was <= 8")
        fallthrough
    default:
        fmt.Println( "default case")
}
```

상기의 코드는 다음과 같이 출력 됩니다.   
```
The integer was <= 6
The integer was <= 7
The integer was <= 8
default case
```


## 함수
함수는 Go언어의 가장 핵심 중의 핵심입니다. 키워드 `func` 으로 함수를 선언 합니다. 선언 형식은 다음과 같습니다.    
```
func funcName(input1 type1, input2 type2) (output1 type1, output2 type2) {
    // 여기 로직 처리 코드입니다.
    // 여러 값을 반환합니다.
    return value1, value2
}
```
위의 코드외에 다음과 같은 것을 알 수 있습니다. 

- 키워드`func`을 이용해서 `funcName`라는 함수를 선언 합니다.   
- 함수는 하나 이상의 인수를 취할 수 있고, 인수의 이름 뒤에 인수의 형식을 지정합니다. 콤마(,)를 구분자로 사용 합니다.      
- 함수는 한번에 여러개의  반환 값을 반환 할 수 있습니다.    
- 예제에서는 두 변수 `output1`과 `output2`에 반환 됩니다. 반환 변수의 이름은 생략해도 무관 합니다.       
- 만약 하나의 반환 값만 존재한다면,  반환 값의 괄호를 생략 할 수 있습니다.   
- 만약 반환 값이 없다면, 반환 값의 정보도 생략 할 수 있습니다.   
- 만약 반환 값이 있으면 함수에서 return 문을 추가해야 합니다.   

다음은 실제로, 함수를 사용하는 예제 입니다. (Max 값을 계산 합니다)
``` Go
package main
import "fmt"

// a, b 중 최대 값을 반환 합니다.
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

func main() {
    x := 3
    y := 4
    z := 5

    max_xy := max(x, y) // 함수 max(x, y)를 호출
    max_xz := max(x, z) // 함수 max(x, z)를 호출

    fmt.Printf("max(% d % d) = % d \n", x, y max_xy)
    fmt.Printf("max(% d % d) = % d \n", x, z, max_xz)
    fmt.Printf("max(% d % d) = % d \n", y, z, max(y, z))  // 직접 호출해도 괜찮습니다.
}
```

위의 `max` 함수에는 2개의 인수가 있습니다. 인수의 형식은 모두 `int`형 입니다.    
첫 번째 인수 형식은 생략 할 수 있습니다.(즉 a, b int,나 a int, b int는 동일합니다.)    
선택은 개발자의 취향입니다. 2개 이상의 동일한 형식의 반환값도 마찬가지 입니다.    

### 여러개의  반환 값
Go 언어는 C에 비해 향상적인 특징을 가지고 있습니다. 

> 함수가 한번에 여러개의  반환 값을 가질 수 있습니다.

코드를 통해서 알아보도록 하겠습니다.     
``` Go
package main
import "fmt"

// A + B와 A * B를 반환 합니다
func SumAndProduct(A, B int) (int, int) {
    return A + B, A * B
}

func main() {
    x := 3
    y := 4

    xPLUSy, xTIMESy := SumAndProduct(x, y)

    fmt.Printf("%d + %d = %d \n", x, y xPLUSy)
    fmt.Printf("%d * %d = %d \n", x, y xTIMESy)
}
```

예제에서는 직접 2 개의 값을 반환 했습니다. 당연히 인수명으로 반환해도 상관없습니다.        
만약 함수가 export되는 함수라면(대문자로 시작) 가능한 한 반환 값에 이름을 붙이는 것을 권장합니다.     
왜냐면 이름없는 반환 코드보다 반환변수명으로 반환하는 것이 코드의 가독성이 좋기 때문 입니다.    
```
func SumAndProduct(A, B int) (add int, Multiplied int) {
    add = A + B
    Multiplied = A * B
    return
}
```


### 가변 인자
Go 함수는 가변 인자 기능을 지원하고 있습니다. 가변 인자를 받아들이는 함수는 불특정개의 인수를 처리합니다.    
 이를 위해서 함수가 가변 인자를 처리할 수 있도록하는 특별한 선언을 해야 합니다.     
``` Go
func myfunc(arg ...int) {}
```
`arg ...int`는 Go에게 이 함수가 다수개의 인수를 받아들이는 함수라는 것을 알려 줍니다. 

> 주의할점은, 이렇게 선언한 다수개의 인수형식은 모두 `int` 입니다.

실제 가변인수는(`args`) 함수 블록 내에서는 `int`의 `slice` 입니다.
```
for _,  n := range arg {
    fmt.Printf("And the number is : %d \n", n)
}
```

### 값 전달과 참조
인수는 호출되는 함수에 전달될 때 실제값의 복사본이 전달 됩니다.    
호출되는 함수에서 인수에 어떠한 수정을 가해도 함수호출시의 실제 인수는 아무런 변화가 없습니다.    
값의 변화 상황은 단지 복사된 값 내에서 변할 뿐입니다. (값에 의한 전달)   

이것을 확인하는 예제를 하나 보겠습니다.     
```
package main
import "fmt"

// 인수 + 1 을 하는 간단한 함수
func add1(a int) int {
    a = a + 1     // a의 값을 변경합니다.
    return a      // 새로운 값을 반환합니다.
}

func main() {
    x := 3

    fmt.Println("x =", x) // "x = 3"으로 출력

    x1 := add1(x)        // add1(x)를 호출

    fmt.Println("x + 1 =", x1) // "x + 1 = 4"로 출력   
    fmt.Println("x =", x)      // "x = 3"으로 출력   
}
```

실행해 보면, `add1` 함수를 호출한 후, `add1`에서 `a = a + 1`을 수행하더라도, `x`변수는    
아무런 변화가 발생하지 않습니다. 이유는 매우 간단 합니다. `add1`함수가 호출될 때, `add1` 함수가 받는     
인수는 `x` 자체가  아니라 `x`의 복사본이기 때문 입니다.

만약 정말 `x`자체를 전달 싶다면 어떻게 처리할까요?  이런 경우는 자주 발생 합니다.     

이 경우, 이른바 포인터가 등장하게 됩니다. 변수는 메모리의 특정 위치에 존재하고 있다는 것을 이미 알고 있습니다.   
실제로 변수를 수정한다는 것은 변수가 위치한 주소의 메모리의 내용을 수정하는 것입니다.    
`add1` 함수가 `x` 변수의 주소를 알고 있다면 `x` 변수의 값을 직접 변경할 수 있습니다. 따라서 `x`변수가    
실제로 존재하는 주소값을, `&x`형식으로 사용해서  함수에 전달하고, 함수의 매개변수 형식을 `int`에서 포인터 변수인   
`*int`로 변경 합니다. 이제는 함수에서 `x`의 값을 직접 변경할 수 있습니다.    
이때 함수​​는 여전히 복사에 의한 인수를 전달합니다!!! 하지만 복사하는 대상이 포인터인 것입니다.    
다음의 예제를 참조해서 보시기 바랍니다.    
```
package main
import "fmt"

// 인수 + 1 처리 함수
func add1(a *int) int {           // 주의하시기 바랍니다.
    *a = *a + 1                   // a의 값을 수정하고 있습니다.
    return *a                     // 새로운 값을 반환합니다.
}

func main() {
    x := 3

    fmt.Println("x =", x)         // "x = 3"으로 출력  

    x1 := add1(&x)                // add1(&x)를 호출하여 x의 주소를 전달

    fmt.Println("x + 1 =", x1)    // "x + 1 = 4"를 출력   
    fmt.Println("x =", x)         // "x = 4"를 출력
}
```

이처럼 `x`변수를 직접 수정할 수 있게 되었습니다.이처럼 포인터를 전달해서 얻는 장점은 무엇일까요?  
 
- 포인터를 전달해서 여러 함수가 같은 객체에 대해 작업을 수행 할 수 있습니다.
- 포인터 전달은 비교적 가볍습니다. (8 바이트만 전달)메모리 주소만 전달하면 그만입니다.    
  포인터를 사용해서 큰 구조체를 빠르게 전달할 수 있습니다. 만약 값으로 전달했다면, 상대적으로 더 많은 시스템 리소스    
  (메모리와 시간)를 매번 함수 호출할때마다 소비하게 됩니다. 따라서 구조체와 같은 큰 자료형을 전달할 때 포인터를   
  사용하는 것이 현명한 선택입니다.      
- Go 언어의`string`, `slice`, `map` 3가지 형식은 바로 포인터를 사용하는 방식입니다.(이미 포인터 참조형입니다)          
  그래서 변수를 직접 전달할 수 있기 때문에 주소를 구해서 포인터를 전달할 필요가 없습니다.
  만약 함수가 `slice`의 길이를 변경하려면 주소를 구한 후 포인터를 전달해야 합니다.   

### defer
Go 언어의 훌륭한 디자인 중 하나로 지연(defer) 문법이 있습니다.    
함수에서 defer문을 여러 개 추가해서 사용할 수 있습니다.  함수가 끝까지 실행되었을 때 , defer문이 역순으로 실행된 후,       
함수가 반환됩니다. 이것은 특히 리소스를 오픈하는 작업을 하거나, 오류 발생에 대비한  롤백기능을 지원할때 요긴하게,    
사용될 수 있습니다. 이렇게 처리하지 않는다면, 리소스 및 메모리  누수 등의 문제를 일으킬 수 있습니다.    
즉, 리소스를 다루는 프로그램의 경우는 다음의 코드와 같이 처리해야만 합니다.     

``` Go
func ReadWrite() bool {
    file.Open("file")
    
    if failureX {
        file.Close ()
    return false
    }

    if failureY {
        file.Close ()
        return false
    }

    file.Close ()
    return true
}
```

위의 코드는 많은 중복 부분이 보입니다. Go의 `defer`는 바로 이러한 문제를 해결 합니다.    
이 기능으로, 코드는 중복부분을 줄이는 기능뿐만 아니라 프로그램을 더 읽기 좋게 만들어 줍니다. `defer`키워드 사용 후   
지정된 함수가 함수를 종료하기 직전에 호출되는 것을 보장해 주게 됩니다.     
``` Go
func ReadWrite() bool {
    file.Open("file")
    defer file.Close()
    if failureX {
        return false
    }
    if failureY {
        return false
    }
    return true
}
```

만약 여러개의 `defer`을 사용하는 경우는 `defer`는 LIFO 형식으로 수행됩니다.    
따라서 다음의 코드는 `4 3 2 1 0`을 출력 합니다.
``` Go
for i := 0; i < 5; i++ {
    defer fmt.Printf("% d", i)
}
```


### 값 형태로 함수사용 

Go언에서 함수또한 변수처럼 타입으로 처리할 수 있습니다. `type` 키워드를 통해서 함수 변수로 정의 합니다.     
이것은 모두 같은 인수와 같은 반환 값을 가지는 하나의 형태입니다.
```
type typeName func(input1 inputType1, input2 inputType2[...]) (result1 resultType1[...])
```
함수를 변수로 다룸으로써 어떠한 장점이 있을까요? 다음의 예를 참조하시기 바랍니다.    
``` Go
package main
import "fmt"

type testInt func(int) bool     // 함수의 형식을 선언합니다.

func isOdd(integer int) bool {
    if integer % 2 == 0 {
        return false
    }
    return true
}

func isEven(integer int) bool {
    if integer % 2 == 0 {
        return true
    }
    return false
}

// 여기에서 선언 함수의 형태를 인수 중 하나로 간주 합니다.

func filter(slice []int, f testInt) []int {
    var result []int
    for _, value := range slice {
        if f (value) {
            result = append (result, value)
        }
    }
    return result
}

func main() {
    slice := []int{1, 2, 3, 4, 5, 7}
    fmt.Println("slice =", slice)
    odd := filter(slice, isOdd) // 함수 값 전달
    fmt.Println("Odd elements of slice are :", odd)
    even := filter(slice, isEven) // 함수 값 전달
    fmt.Println("Even elements of slice are :", even)
}
```

공유 인터페이스를 쓸 때 함수 값과 타입으로 처리하면 매우 편리합니다.    
위의 예에서 `testInt`라는 형식은 함수의 형태 중 하나였습니다. 두 `filter` 함수의 인수와 반환 값은 `testInt`의  
형태와 동일하지만, 더 많은 로직을 제공 할 수 있습니다. 따라서 프로그램을 더 좋게 개선할 수 있습니다.  

### Panic와 Recover

Go언어는 Java와 같은 예외처리를 하지 않습니다. 즉, 예외를 던지지 않는 것입니다.     
대신 `panic`과 `recover` 기능을 사용 합니다. 반드시 기억 하시기 바랍니다. 이것은 최후의 수단으로 사용해야 한다는 것을.    
`panic`을 사용을 최대한 줄이시기 바랍니다. 이것은 매우 강력한 도구이나, 현명하게 사용하시기 바랍니다.       

`Panic`  
> 내장 함수 입니다. 원래의 처리 흐름을 중단시킬 수 있습니다.   
  패닉이 발생한 로직에 들어간 후  함수 `F`가 `panic`을  호출 합니다. 이 프로세스는 계속적으로 실행됩니다.    
  일단 `panic`이 `goroutine`에서 발생하면 호출 된 함수가 모두 반환 됩니다. 이 것은 프로그램을 종료 합니다.    
  직접`panic`를 호출 합니다. 실행시 오류가 발생해도 처리할 수 있게 됩니다.   
  예를 들어 배열의 경계를 넘어 액세스를 할 경우 등입니다.     

`Recover`  
> 내장 함수 입니다. 에러 상황을 발생시키는 `goroutine`을  복원 할 수 있습니다.   
  `recover` 지연 함수 내에서만 유효 합니다. 일반적으로 실행중에 `recover`를 호출하면 `nil`이  반환 됩니다.     
  즉, 다른 아무런 효과도 없습니다. 만약 현재 `goroutine`에서 패닉이 발생하면, `recover`를 호출하여 `panic`의     
  입력 값을 보존한 후 정상적인 실행상태로 복원 할 수 있습니다.  

다음 예제를 통해서 어떻게`panic`이 작동하는지 알아보도록 하겠습니다.       
``` Go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

이 함수는 인수가 함수 실행시`panic`를 발생하는지 확인합니다 :
``` Go 
func throwsPanic(f func()) (b bool) {
    defer func() {
        if x := recover(); x! = nil {
            b = true
        }
    }()
    f() // 함수 f를 실행 합니다. 만약 f에서 panic이 발생되면 복원을 할 수 있습니다.
    return
}
```



### `main` 함수와`init` 함수

Go는 이미 2개의 함수가 예약되어 있습니다.  
`init` 함수(모든 `package`에서 사용할 수 있습니다)와 `main` 함수(`package main`에서만 사용할 수 없습니다)입니다.  
이 두 함수는 선언할때, 어떠한 인수나 반환 값을 가지지 않습니다. `package` 속에서 여러개의 `init` 함수를 사용해도,   
아무런 문제가 없습니다. 물론 가독성은 저해됩니다. `package`에서 하나의 `init` 함수를 작성하길 강력히 추천 합니다.   

Go 프로그램은 자동으로 `init()`함수와 `main()`함수를 호출하기 때문에, 별도로 이 두 함수를 직접 호출 할 필요는 없습니다.   
`package`의 `init` 함수의 사용은 옵션입니다만, `package main`은 반드시 `main` 함수를 포함해야 합니다.  

프로그램 초기화 및 실행은 모두 `main` 패키지에서 시작 됩니다. 만약`main` 패키지가 다른 패키지를 가져 오는 경우에,   
컴파일시에 종속된 패키지를 가져옵니다. 패키지가 여러 패키지를 동시에 가져올 수있는 경우에는 먼저 다른 패키지를 가져온 후,  
이 패키지 안에있는 패키지 클래스 상수와 변수가 초기화 됩니다.    
다음으로,  init 함수(있을 경우)가 실행되고, 마지막에 `main` 함수가 실행되는 구조 입니다.   
다음 그림에서 실행 과정을 자세히 설명하고 있습니다.   

![](images/2.3.init.png)   
그림 2.6 main 함수를 사용하여 패키지 가져 오기 및 초기화 과정 그림  


### import
Go 코드를 쓸 때는 import 명령으로 패키지 파일을 가져 오는 경우가 종종 있습니다.    
대부분 사용하는 방법은 다음과 같습니다.    
``` GO
import (
    "fmt"
)
```

상기처럼 `fmt` 패키지를 가져오면, 코드에서 다음과 같은 방법으로 패키지내의 함수를 호출 할 수 있습니다.    
```
fmt.Println("hello world")
```
fmt 패키지는 Go 언어의 표준 라이브러리 입니다. 사실 `$GOROOT` 환경 변수에 지정된 디렉토리 아래에 이 모듈이    
실제로 존재합니다. 당연히 Go 가져 오기는 다음과 같은 두 가지 방법으로 자신이 쓴 모듈을 직접 추가 할 수 있습니다.  

1. 상대 경로   
   import "./model"
   // 현재 파일과 같은 디렉토리에 있는 model 디렉토리. 그러나 이 방법으로 import는 추천하지 않습니다.

2. 절대 경로   
   import "shorturl/model" 
   // $gopath/src/shorturl/model 모듈을 추가합니다.


여기에서는 import에 대하여 일반적인 몇 가지 방법을 설명했습니다.      
그 밖에도 특수한 import가 몇가지 있습니다. 초보자에게는 낯설게 보이는 몇가지에 대해서 알아보겠습니다.    


1. 도트   
   간혹 다음과 같이 패키지를 가져 오는 방법을 볼 수 있습니다
```
    import (
      . "fmt"
    )
```
   이런 방식의 의미는 해당 패키지를 가져온 후에,  패키지의 함수를 호출 할 때 패키지 이름을 생략 할 수 있습니다.   
   즉, fmt.Println("hello world")로 호출하는 것을 Println("hello world")와 같이 패키지. 을 생략할 수 있습니다.

2. 별칭  
   별칭은 이름에 대하여 동일한 별명을 사용해서 읽기 편하게 사용할 수 있습니다.   
```
   import (
        f "fmt"
    )
```
   별칭으로 사용할 경우 패키지 함수 호출 할 때 접두사를 별칭으로 사용할 수 있습니다. 즉 f.Println("hello world")

3. _    
  일반적으로 이해하기 어려운 방법에 속합니다. 다음의 import를 참조 하십시오.
```
import (
    "database / sql"
    _ "github.com/ziutek/mymysql/godrv"
)
```
_(언더바)는 패키지를 가져 오기만 하는 것으로 패키지내의 함수를 직접 사용하는 것이 아니라,  
이 패키지 안에 있는 init 함수만 호출 합니다.  


