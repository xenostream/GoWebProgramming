---
layout: post
title: 'Go Web Programming: [02/02] Go Foundation'
tags: goweb
---  


# Go Foundation   

이 장에서는 변수, 상수, 기본타입과 Go 프로그램의 여러가지 기본적인 기법에 대해서 소개 합니다.  

## 변수

Go 언어에서 변수는 여러가지 방법으로 선언될 수 있습니다.   
가장 표준적인 방법으로 `var` 키워드를 사용하여 변수를 선언하는 방법 입니다.    

>`C 언어`와 달리 Go언어는 변수의 유형을 **변수 뒤**에 놓습니다!!!    


```go
  // "variableName"라는 이름의 변수를 선언 합니다. 변수 형식은 "type"입니다.   
  var variableName type  
```

여러 변수를 한번에 선언 합니다.   

```go
  // "type"형의 변수 3개를 선언 합니다.
  var vname1, vname2, vname3 type
```

변수를 선언함과 동시에 초기화 합니다.   

```go
  // "variableName"변수를 "value"로 초기화합니다. 형식은 "type"입니다.
  var variableName type = value
```

여러개의 변수를 동시에 선언한 후 초기화 합니다.

```go
/*
  모든 변수를 "type"형으로 선언하고,vname1은 v1, vname2는 v2, vname3은 v3으로 초기화 합니다. 
*/
  var vname1, vname2, vname3 type = v1, v2, v3
```

만약 위와 같은 방식으로 선언하는 방식이 불편하다고 생각 하십니까?   
이런 문제점은 , Go 언어의 설계자도 알고 있습니다. 좀 더 쉽게 변수를 선언할 수 있는 방법이 있습니다.  
초기값으로 해당 변수의 형식을 판가름 할 수  있으므로, 다음과 같은 방법으로 변수를 선언할 수 있습니다.  

```go
/*
   3 개의 변수를 선언하고 개별적으로 초기화 합니다. 
   vname1는 v1, vname2는 v2, vname3는 v3 과 같이 대입되는 값의 형식에 맞게 각각 형이    
   정해지고 초기화 됩니다.
*/
   var vname1, vname2, vname3 = v1, v2, v3
```

이런 방법도 여전히 번거로우신가요? 좀 더 나은 방법으로 선언해 보겠습니다. 

```go  
/*
  3개의 변수를 선언하고 개별적으로 초기화 합니다.
  vname1는 v1, vname2는 v2, vname3는 v3
  컴파일러는 초기화 값에 따라 자동으로 적합한 형식을 판단 합니다.   
*/
  vname1, vname2, vname3 :=  v1, v2, v3
```

이 방법이면 매우 간결하게 사용 할 수 있겠습니다. `:=`기호가 `var`와`type`을 대체 합니다.  
이러한 형식으로 변수를 선언하는 것을 `단축선언`이라고 합니다.    
그러나 이 단축선언 방식에는 한가지 제한이 있습니다. 바로 `함수 내부`에서만 사용할 수 있습니다.  
함수 밖에서 사용하면 컴파일러가 에러를 출력 합니다.   
따라서 일반적으로 `var` 형식으로 선언하는 변수들은 글로벌 변수로 선언됩니다.

`_` (밑줄)은 변수의 특별한 이름입니다. 어떤 값으로 초기화 해도 그 값을 모두 버려 버립니다.    
다음 예제에서 `35`라는 값을`b`에 부여하며, `34`는 버려  버립니다.

```go
_ , b := 34, 35
```

Go 컴파일러는 선언한 후 사용하지 않는 변수가 발견되면 컴파일 오류를 출력 합니다.   
즉, 다음과 같은 코드는 에러 입니다. 그래서 밑줄 변수가 존재 합니다. 

```go
package main

func main () {
    var i int
}
```

## 상수

이른바 상수는 프로그램이 `컴파일되는 단계`에서 값이 결정됩니다. 따라서 프로그램이 실행될 때 값의 변경은   
절대 허용되지 않습니다. 상수는 숫자, bool 또는 문자열 등의 형태로 선언 할 수 있습니다.

상수로 선언하는 문법은 다음과 같습니다.

```go
const constantName = value
    // 만약 필요하다면 상수의 타입을 명시 할 수 있습니다.  
const Pi float32 = 3.1415926
```
다음은 여러가지 상수 선언의 예제 입니다.  

```go
const Pi = 3.1415926
const i = 10000
const MaxThread = 10
const prefix = "astaxie_"
```
Go 상수는 일반적인 프로그래밍 언어와 달리 많은 소수 자릿수를 지정할 수 있습니다.(예: 소수점 200 자리 등)   


## 내장 기본형

### Boolean

Go언어에서 bool 값의 선언 키워드는 `bool`입니다. 값은`true` 혹은`false`이며, 기본값은 `false` 입니다.

```go
var isActive bool                   // 전역 변수 선언
var enabled, disabled = true, false // 형 생략 선언
func test () {
    var available bool             // 일반 선언
    valid := false                 // 단축 선언
    available = true                // 대입 
}
```

### 숫자

`정수`에는 부호와 무부호의 2가지가 있습니다. Go에서는 `int`와`uint`로 지원하고 있습니다.   
이 두 가지는 같은 길이로 표현되지만, 32/64비트 운영체제에 따라 실제길이는 다르게 표현 됩니다.   
Go에서는 직접 bit 수를 지정할 수있는 유형도 있습니다.   

```go
rune int8 int16 int32 int64 byte uint8 uint16 uint32 uint64
```

`rune`은 `int32`의 별칭이며, UTF-8 한 문자를 저장 합니다.   
`byte`는 `uint8`의 별칭이며, ASCII한 문자를 저장 합니다. 

주의해야 할 점은 한번 형태를 지정하면, 다른 형의 변수끼리 연산을 할 수 없으며, 컴파일시에 컴파일러는   
오류를 발생 시킵니다. 즉 다음의 코드는 오류가 발생합니다.

```go
 var a int8
 var b int32
 c := a + b
```

` invalid operation : a + b (mismatched types int8 and int32) `

32비트 운영체제에서 int의 길이는 32bit지만, int 및 int32도 서로 연산할 수 없습니다.(강타입 언어)

`부동 소수점` 형식은`float32`와`float64`의 두 종류만 있습니다 (`float` 형은 없습니다.)   
디폴트는 `float64` 입니다.

이것이 숫자의 모든것 아닙니다. Go 언어는 복소수 또한  지원하고 있습니다.   
기본 형식은 `complex128`(64bit 실수 + 64bit 허수)입니다. 만약 좀 더 작은 값이 필요하다면,   
`complex64`(32bit 실수 + 32bit 허수)도 있습니다.  
복소수의 형식은 `RE + IMi` 입니다. 이 중 `RE`가 실수 부분 `IM`이  허수 부분이며,   
마지막`i`는 허수 단위 입니다. 다음은 복소수 사용 예제 입니다.

```go
var c complex64 = 5 + 5i
fmt.Printf("Value is : % v", c)
```

`Value is : (5 + 5i)` 와 같이 출력 됩니다. 

### 문자열

앞 장에서 언급 한 바와 같이 Go 언어에서  문자열은 모두 `UTF-8` 코드로 사용 합니다.   
문자열은 쌍 따옴표("") 또는 백틱('')으로 감싸는 것으로 선언 됩니다. 이 형태는 "string" 입니다.

```go
var frenchHello string      // 문자열 변수 선언의 일반적인 방법
var emptyString string = "" // 문자열 변수를 하나 선언하고 빈 문자열로 초기화 한다.
func test() {
    no, yes maybe := "no" "yes" "maybe"   // 단축선언, 동시에 여러 변수를 선언 및 초기화    
    japaneseHello := "Konichiwa"          // 선언 및 초기화
    frenchHello = "Bonjour"               // 문자열 대입
}
```

>Go 언어에서 문자열은 변경할 수 없습니다.    

예를들어 다음 코드는 컴파일시 오류가 발생 합니다.   

```go
var s string = "hello"
s [0] = 'c'
```

오류내용 : `can not assign to s [0]`

하지만, 프로그래밍 중에 실제로 문자열 값을 바꿀려면 다음과 같이 사용하면 됩니다.    

```go
s := "hello"
c := []byte(s)   // 문자열 s를 []byte 형 배열로 변환
c [0] = 'c'      // 배열 인덱스로 값 변경 
s2 := string(c)  // 배열을 다시 string 형식으로 변환 
fmt.Printf( "%s \n", s2)
```

Go는 `+` 연산자를 사용하여 문자열을 연결할 수 있습니다.  

```go
s := "hello"
m := "world"
a := s + m
fmt.Printf ( "%s \n", a)
```

문자열을 수정할 경우 이런 방법도 사용할 수 있습니다.   

```go
s := "hello"
s = "c" +  s[1:]    // 문자열은 변경할 수 없지만 슬라이스를 이용할 순 있습니다.  
fmt.Printf( "%s \n", s)
```

만약 여러 줄에 걸쳐서 문자열을 선언하고 싶다면 `` 로 선언 할 수 있습니다.

```go
m := `hello
      world`
```

``로 둘러싸인 문자열은 `Raw 문자열` 입니다. 즉, 문자열에서 나열된  형식 그대로 인식하는 형식 입니다.  
문자열의 대체 치환 변경은 없습니다. 줄 바꿈문자(\n)는 그대로 출력 됩니다. 


### 오류 유형
Go에는 `error`형이라는 내장 형식이 있습니다. 오류 정보의 처리에 사용 됩니다.   
또한 Go언어에는 `package`에서 오류 처리를 할수 있는 `errors`라는 패키지도 존재 합니다.    

```go
err := errors.New( "emit macho dwarf : elf header corrupted")
if err != nil {
    fmt.Print(err)
}
```

### Go 데이터의 실제 저장 방식 

아래의 그림은 [Russ Cox Blog](http://research.swtch.com/) 에서 잘 설명되어 있습니다.    
[Go 데이터 구조](http://research.swtch.com/godata)에 대한 내용 입니다.    
이런 기본 형식은 낮은 계층에서 메모리를 분배하고 해당 값을 저장하고있는 것이 간파 할 수있다 생각합니다.

![](/images/2.2.basic.png)  
그림 2.1 Go 데이터 저장 형식  

## 기술

### 그룹화에 의한 선언

Go 언어에서는 여러개이  정수·변수를 선언하거나 여러 패키지에서 import할 경우 그룹화를 이용할 수 있습니다.  

예를 들어 다음과 같이 그룹화 선언을 사용할 수 있습니다. (괄호 사용)

```go
import "fmt"
import "os"

const i = 100
const pi = 3.1415
const prefix = "Go_"

var i int
var pi float32
var prefix string
```

상기의 코드를 그룹화를 이용하여 작성하면 다음과 같습니다.

```go
import(
    "fmt"
    "os"
)

const(
    i = 100
    pi = 3.1415
    prefix = "Go_"
)

var(
    i int
    pi float32
    prefix string
)
```

### iota 열거형

Go 언어에는 `iota`라는 키워드가 있습니다. 이 키워드는 `열거형(enum)`을 선언 할 때 사용 됩니다.   
기본값은 0부터 시작하여 순차적 1씩 증가 됩니다.

```go
const(
    x = iota // x == 0
    y = iota // y == 1
    z = iota // z == 2
    w        // 상수 선언에서 값을 생략하면 기본값은 이전 값과 동일 합니다.  
             // 여기에서 z = iota 로 선언하고 있으므로, 다음값인 w == 3 입니다.  
             // 사실 예제에서  y와 z 값도 사실 "= iota"는 생략 할 수 있습니다.
)

const v = iota // const 키워드를 사용할 때마다, iota 값은 리셋됩니다. 즉,v == 0 입니다.

const(
    e, f, g = iota, iota, iota  // e = 0, f = 0, g = 0 iota 와 동일 합니다
)
```

> 다른 값과 `iota`에 설정되어있는 것을 제외하고 각 `const` 그룹의 시작 상수는 기본적으로 0 입니다.  
두번째 이후의 상수는 이전 상수 값을 기본값으로 사용합니다. 이전 상수의 값이`iota`이면 후의 값도`iota` 입니다.   

### Go 프로그램의 디자인 규칙
Go 언어가 간결함을 유지하는 것은 다음과 같은 기본적인 규칙이 있기 때문입니다.    

- 대문자로 시작하는 변수는 외부에 공개됩니다.   
  즉, 다른 패키지에서 읽을 수있는 공용 변수라는 의미 입니다. 소문자로 시작하는 변수는 내보낼 수 없습니다.   
  즉, 내부 개인 변수 입니다. ( Public / Private 에 해당)
- 대문자로 시작하는 함수 또한 마찬가지 입니다.    
  대문자로 시작하는 변수/함수는 객체지향언어 에서 `class`의 `public` 키워드와 동일한 의미 입니다.    
  소문자로 시작하는 변수/함수는 `private` 키워드와 그 의미가 동일합니다. 

## array, slice, map

### array
`array` 는 배열 입니다. 배열형의 정의는 다음과 같습니다.

```go
var arr[n] type
```

> 배열은 반드시 [n]과 같이 크기가 지정됩니다.!!  

`[n] type`에서 `n`은 배열의 길이를 나타 냅니다. `type`은 배열 항목의 형식을 의미합니다.     
배열에 대한 조작 방식은 다른 언어와 비슷하며, `[]`을 통해서  값을 대입하거나 추출합니다.     

```go
var arr[10] int     // int 형의 배열을 선언합니다.
arr [0] = 42        // 배열의 인덱스는 0부터 시작합니다.
arr [1] = 13        // 대입
fmt.Printf("The first element is %d \n", arr[0]) // 데이터를 추출하여 42를  반환 합니다.
fmt.Printf("The last element is %d \n", arr[9]) 
// 값이 할당되지 않은 마지막 요소를 반환합니다. 초기화 하지 않았으므로, 0이 반환 됩니다.  
```

길이값은 배열의 일부이므로 `[3]int` 과 `[4]int` 다른 형식 입니다. 배열도 길이를 변경할 수 없습니다.       
배열 형식의 상호 대입은 값으로 전달 합니다. 즉, 하나의 배열이 함수의 인수로 사용 된  경우 전달 되는 값은,       
배열의 복사본이며, 포인터가 아닙니다. 만약 포인터형식으로 사용 하려면 뒤에 소개하는 `slice`를  이용 하십시오.       

배열또한 `:=`로 선언 할 수 있습니다.  

```go
a := [3]int{1, 2, 3}     // 길이가 3인 int 배열을 선언 합니다.  
b := [10]int{1, 2, 3}    // 길이가 10의 int 배열을 선언합니다.   
                         // 이 중 3 가지 요소의 초기 값은 1,2,3이고, 나머지는 0입니다.  
c := [...]int{4, 5, 6}   // 길이값에 `...`를 사용해서 크기를 자동계산 할 수 있습니다.    
```

만약 `배열의 배열`을 사용할 수도 있습니다. Go는 중첩배열을 지원 합니다.    
예를 들어 다음 코드는 이차원 배열을 선언하고 있습니다.   

```go
// 이차원 배열을 하나 선언 합니다. 이 배열은 두 개의 배열을 요소로하고 
// 각 배열에는 4 개의 int 형의 요소가 포함되어 있습니다.
doubleArray := [2][4] int{[4]int{1, 2, 3, 4}, [4]int{5, 6, 7, 8}}

// 위의 선언은 단순화 할 수 있습니다.
easyArray := [2][4] int{ {1, 2, 3, 4}, {5, 6, 7, 8}}
```

배열의 상태는 다음과 같습니다.

![](/images/2.2.array.png)   

그림 2.2 다차원 배열의 매핑 관계  


### slice

많은 응용 프로그램에서 배열만으로는 요구사항을 모두 충족할 수 없습니다.    
가끔 어느 정도 크기의 배열이 필요한지 모르는 경우도 존재하기 때문입니다. 따라서 "동적 배열"이 필요 합니다.        
Go는 이러한 동적 배열을 가능하게 만들어 주는 데이터 구조를 `slice`라고 합니다.       

`slice`는 진정한 의미의 동적 배열은 아닙니다. 단순한 참조형식 일뿐입니다.     
`slice`는 실제로는 `array`형태로 내부적으로 처리됩니다.   
>`slice` 선언은  `array`와같이 길이를 지정할 필요가 없습니다.  

```go
// array 선언과 동일하지만 길이를 지정하지 않습니다.   
var fslice[] int
```

`slice`를 선언함과 동시에 데이터를 초기화 합니다.  

```go
slice := []byte { 'a', 'b', 'c', 'd'}
```

`slice`는 배열 또는 이미 존재하는 `slice` 내부에서도 선언 할 수 있습니다.    
`slice`는 `array[i:j]`형식으로 그 값을 얻을 수 있습니다. `i`는 배열의 시작 위치이며, `j`는 끝위치 입니다.      
그러나 `array[j]`는 그 값에 포함되지 않습니다. 즉, 길이는 `j-i` 입니다.  

```go
// 10개의 요소를 선언 합니다. 요소의 형태는 byte의 배열 입니다.
var ar = [10] byte{ 'a', 'b', 'c', 'd', 'e',​​ 'f', 'g', 'h' 'i', 'j'}

// byte를 포함한 2 개의 slice를 선언합니다
var a, b []byte

// ar 포인터 배열의 세 번째 요소에서 시작하여 5 번째 요소까지
a = ar [2 : 5]

// 현재 a 변수가 가지는 값은  ar[2] ar[3]과 ar[4] 입니다.

// b는 배열 ar의 또 다른 slice입니다.
b = ar [3 : 5]

// b의 요소는 ar[3]과 ar[4]입니다.
```

>`slice`와 `array`는 선언시에 구분 되므로 주의 하시기 바랍니다.   

배열은 선언 할 때 괄호안에([]) 배열의 길이를 명시하거나 `...` 를 사용해서 자동으로 길이를 계산 합니다.      
한편 `slice`를 선언 할 때는 괄호([]) 안에 크기를 지정하는 문자가 없습니다.  

이러한 데이터 구조는 다음과 같이되어 있습니다.  

![](/images/2.2.slice.png)   
그림 2.3 slice와 array의 대응 관계도  

slice는 다음과 같이 사용됩니다.    

 -`slice`의 기본 시작 위치는 0 입니다. `ar[:n]`은 `ar[0:n]`과 같습니다.  
 -`slice`의 두 번째 값은 배열의 길이를 의미합니다. `ar[n:]` 은 `ar[n:len(ar)]`과 같습니다.  
 - 만약 `slice`의 모든값이 필요할 경우 경우에는, `ar[:]`라는 형태로 사용 할 수 있습니다.   
   왜냐하면 기본 시작 값은 0이고, 두 번째는 배열의 길이에 해당하기  때문입니다. 즉`ar[0:len(ar)]`과 같습니다.         

다음은 `slice`로 작업하는 몇가지 예제 입니다.    

```go
// 배열을 선언
var array = [10] byte{ 'a', 'b', 'c', 'd', 'e',​​ 'f', 'g', 'h' 'i', 'j'}
// slice를 2 개 선언
var aSlice, bSlice []byte

// 조작 
aSlice = array[3]  // aSlice = array[0:3]과 같다.  aSlice는 다음 요소가 포함됩니다 : a, b, c
aSlice = array[5:] // aSlice = array[5:10]과 같다. aSlice는 다음 요소가 포함됩니다 : f, g, h, i, j
aSlice = array[:]  // aSlice = array[0:10]과 같다. aSlice에는 모든 요소가 포함되어 있습니다.

// slice에서 slice를 추출 
aSlice = array[3:7]  // aSlice는 다음 요소가 포함됩니다 : d, e, f, g, len = 4, cap = 7
bSlice = aSlice[1:3] // bSlice에는 aSlice[1] aSlice[2]가 포함되며 각각의 요소는 다음과 같습니다 : e, f
bSlice = aSlice[3]   // bSlice에는 aSlice[0], aSlice[1] aSlice[2]가 포함됩니다. 각각 다음과 같습니다 : d, e, f
bSlice = aSlice[0:5] // slice의 cap의 범위 내에서 확장 할 수 있습니다. bSlice는 다음 요소가 포함됩니다 : d, e, f, g, h
bSlice = aSlice[:]   // bSlice에는 aSlice의 모든 요소를​​ 포함합니다 : d, e, f, g
```

>`slice`는 참조 형이므로,이 요소의 값을 변경하면 다른 모든 참조에서도 값이 변경 됩니다.   

예를 들어 위의 `aSlice`와 `bSlice`에서 `aSlice`의 요소를 변경하면 `bSlice`의 해당 값도 변경 됩니다.  

개념적으로는 `slice`는 구조체 입니다. 이 구조체에는 3 개의 요소가 포함되어 있습니다.  
- 첫번째는, 포인터 입니다. 배열의 `slice`가 나타내는 시작 위치를 가리 킵니다.  
- 두번째는, 길이, 즉 `slice`의 길이 입니다.  
- 세번째는, 최대 길이, `slice`의 시작 위치에서 배열의 마지막 위치까지의 길이 입니다.  

```go 
Array_a := [10]byte{ 'a', 'b', 'c', 'd', 'e',​​ 'f', 'g', 'h' 'i', 'j'}
Slice_a := Array_a[2:5]
```

위 코드의 데이터 저장 구조는 다음 그림과 같습니다.  

![](/images/2.2.slice2.png)  
그림 2.4 slice에 대응되는 배열정보    

`slice`에는 몇 가지 유용한 내장 함수가 있습니다.  

-`len()`     `slice`의 길이를 가져 옵니다.  
-`cap()`     `slice`의 최대 용량을 가져 옵니다.  
-`append()`  `slice`에 하나 이상의 요소를 추가합니다. 그 후`slice`와 같은 형태의`slice`을 반환 합니다.  
-`copy()`    원래`slice`의`src`를 `dst`요소에 복사하고 복사 된 요소의 개수를 반환 합니다.  

`append` 함수는 `slice`가 참조하는 배열의 내용을 변경하는 것입니다. 따라서 참조와 동일한 배열의 다른`slice`에도   
영향을 줍니다. 그러나`slice`에 공백이없는(`(cap-len) == 0`) 경우 동적 메모리에 새로운 배열 공간이 할당 됩니다.     
반환되는 `slice` 배열의 포인터는 이 공간을 가리키고 있습니다. 또한 원래 배열의 내용은 바뀌지 않습니다.     
이 배열을 참조하는 다른`slice`은 영향을받지 않게 됩니다. 

Go 1.2에서, slice는 세 인수의 slice를 지원하게 되었습니다. 
이전까지 우리는 다음과 같은 방법으로 slice 또는 array에서 slice를 제거 했습니다.

```go
var array [10]int
slice := array[2:4]
```

이 예에서는 slice의 요소 수는 8이고, 새 버전에서는 다음과 같이 요소 수를 지정할 수 있습니다.  

```
slice = array[2:4:7]
```

위의 요소 수는 `7-2`, 즉 5입니다. 이렇게 생성 된 새로운 slice는 마지막 3 가지 요소에 액세스하는 방법이 없습니다.  

만약 slice가 `array[:i:j]` 같은 형식이라면 첫 번째 인수를 기본으로 간주하며,  기본값은  0 입니다.  

### map

`map`의 개념은 Python 사전과 동일 합니다. 이 형식은 `map[keyType]valueType` 입니다.  

아래의 코드를 참조 하시기 바랍니다. `map`의 사용법은 `slice`와 유사합니다. 단지  `key` 통해 조작 합니다.      
다만 `slice`의 `index`는 `int`유형 이어야만 합니다. `map`의 `index`에는 많은 형태가 있습니다.    
`int`도 될 수 있으며, `string` , `==`와 `!=` 등 연산자가 정의되어 있는 어떤 형태라도 괜찮습니다.  

```go
// key를 문자열로 선언 합니다. 값은 int 인 사전입니다.   
// 이 방법은 사용되기 전에 `make로 초기화`되어야 합니다.
var numbers map[string] int

// 또 다른 map의 선언 방법
numbers := make(map[string] int)
numbers [ "one"] = 1           // 대입
numbers [ "ten"] = 10          // 대입
numbers [ "three"] = 3

fmt.Println( "세 번째 숫자는 :", numbers[ "three"])   // 데이터 검색
// "세 번째 숫자는 : 3"라는 형식으로 출력됩니다.
```

이 `map` 은 일반적으로 사용하는 표와 유사 합니다.     
왼쪽 열에 `key` 오른쪽 열에 `value` 값이 있습니다.  

map을 사용할 경우 몇가지 주의사항이 있습니다.       

-`map`은 순서가 없습니다.     
 `map`의 출력은 매번 다르게 출력될 수 있습니다.     
 `index`로 값을 얻을 수 없으며 항상 `key`를  사용해서 값을 추출해야 합니다.     
-`map`의 길이는 고정되지 않습니다.       
  이것은 `slice`와 같고, 참조 형의 일종 입니다.    
- 내장 `len` 함수를 `map`에 적용하면 `map`이 가지는`key`의 개수를 반환 합니다.    
-`map` 값은 쉽게 다룰 수 있습니다.      
 `numbers["one"] = 11`와 같이 key가 `one`인 사전의 값을 `11`로 변경합니다.      
-`map`은 다른 기본형과 달리, `thread-safe`하지 않습니다.     
  즉, 여러 go-routine을 다룰 때는 반드시 mutex lock 메커니즘을 사용해야만 합니다.      

`map` 초기화는 `key:val` 방법으로 초기 값을 줄 수 있습니다.    
또한 동시에`map`에는 기본적으로 `key`가 존재하는지 확인하는 방법이 존재 합니다.  

`delete`함수로 `map` 요소를 삭제 합니다.    

```go
// 사전을 초기화합니다.
rating := map[string] float32{ "C": 5, "Go": 4.5 "Python": 4.5 "C ++": 2}

// map은 2 개의 반환 값을 가집니다. 첫번째는 해당 키의 값이며, 두 번째 반환 값은 
// 만약 key가 존재하지 않으면 false가 존재하면 true 값을 반환 합니다.   
csharpRating, ok : = rating[ "C#"]
if ok {
    fmt.Println( "C# is in the map and its rating is", csharpRating)
} else {
    fmt.Println( "We have no rating associated with C# in the map")
}

delete(rating "C") // key가 C의 요소를 제거합니다.  
```

위에서 말한 것처럼 `map`은 참조 형의 일종이기 때문에 만약 2 개의 `map`이 동일한 포인터를 가리키는      
경우 하나를 변경한다면 또다른 맵의 값도 변경 됩니다.    

```go
m := make(map[string] string)
m ["Hello"] = "Bonjour"
m1 := m
m1["Hello"] = "Salut"      // 이때 m["hello"] 값도 이미 변경 됩니다. 
```

### make, new 메모리 조작

`make`는 내장형(`map`, `slice` 및`channel`)의 메모리를  할당 합니다.  
 `new`는 각 형태의 메모리를 할당합니다.  

내장 함수`new`는 본질적으로 다른 언어에서 사용되는 메모리할당 `new()` 함수와 동일 합니다.    
`new(T)`는 0으로 초기화된 `T` 형의 메모리 공간을 할당하고 그 주소를 돌려 줍니다 . 즉 `*T` 형 값입니다.     
Go 용어로 말하면, 포인터를 반환하는 것입니다. 새롭게 할당 된 형식 `T` 는 0 입니다.  
  
>중요한 점은 `new`는 포인터를 반환 합니다.

내장 함수`make(T, args)`는 `new(T)`와는 다른 기능을 가지고 있습니다.      
make는 `slice`, `map` 또는`channel`을 만들고 초기값(0이 아닌 값)을 가진`T` 형을 반환하므로, `*T`가 없습니다.     
기본적으로 이 3 가지 형태가 다른 점은 데이터 구조를 가리키는 참조가 사용되기 전에 초기화되느냐의 여부입니다.      
예를 들어, 데이터(내부`array`)를 가리키는 포인터, 길이, 용량에 따른 3 가지로 설명되는 `slice`의 각 항목이     
초기화되기 전에는 `slice`의 값은 `nil` 입니다. `slice`, `map`, `channel`에서, make는 내부 데이터 구조를     
초기화하고 적당한 값으로 대입됩니다.    

> `make`는 초기화를 수행한 후 값을 반환 합니다.

다음 그림은 `new`와 `make`의 차이점에 대해 자세히 설명하고 있습니다.  

![](/images/2.2.makenew.png)   
그림 2.5 make와 new 의 실제 메모리 할당


## 제로 값
"제로 값"은 값이 비어있다는 의미가 아니며, 초기값이라는 의미 입니다.   
이것은 일종의 "변수의 생성초기"의 기본값이며, 일반적으로 0 입니다.  
각 유형별 제로 값은 다음과 같습니다.   

```go
int          0
int8         0
int32        0
int64        0
uint         0x0
rune         0      // rune의 실제 형태는 int32입니다.
byte         0x0    // byte의 실제 형태는 uint8입니다.
float32      0      // 길이는 4 byte
float64      0      // 길이는 8 byte
bool         false
string       ""
```

