# Concurrency

Go를 21 세기의 C 언어라는 사람도 있습니다. Go 언어는 설계가 간단하고, 21 세기에서 가장 중요한 것은 멀티 스레드이기 때문입니다. Go 언어 수준에서 멀티 스레드를 지원하고 있습니다.

## goroutine

goroutine은 Go 멀티 스레드의 핵심입니다. goroutine은 사실 처음부터 끝까지 스레드입니다. 그러나 스레드보다 작은 수십 개의 goroutine은 낮은 계층에서 5,6 개의 스레드를 실현하고있을뿐입니다. Go 언어의 내부에서는 이러한 goroutine 사이에서는 메모리 공유를 실현하고 있습니다. goroutine을 실행하려면 아주 작은 스택 메모리 (대략 4 ~ 5KB입니다.)을 필요로 할뿐입니다. 당연히 해당 데이터가 늘어납니다 만, 바로이 때문에 여러 다중 스레드 작업을 수행 할 수 있습니다. goroutine은 thread에 비해보다 사용하기 쉽고, 효과적이고 편리합니다.

goroutine은 Go의 runtime 관리를 이용한 스레드 컨트롤러입니다. goroutine은`go` 키워드에 의해 실현합니다. 사실 그냥 보통의 함수입니다.

go hello (a, b, c)

키워드 go 통해 goroutine을 시작합니다. 예를 살펴 보자.

package main

import (
"fmt"
"runtime"
)

func say (s string) {
for i : = 0; i <5; i ++ {
runtime.Gosched ()
fmt.Println (s)
}
}

func main () {
go say ( "world") // 새로운 Goroutines를 실행한다.
say ( "hello") // 현재 Goroutines 실행
}

// 위의 프로그램을 실행하면 다음과 같이 출력됩니다 :
// hello
// world
// hello
// world
// hello
// world
// hello
// world
// hello

go 키워드로 아주 쉽게 멀티 스레드 프로그래밍을 실현할 수 아실 것입니다.
위의 여러 goroutine은 동일한 프로세스에서 실행되고 있습니다. 메모리의 데이터를 공유하고 있지만 디자인에 공유를 이용하여 통신을하거나하지 않고 통신을 통해 공유 할 수 있도록합시다.

> runtime.Gosched ()에서는 CPU 시간을 다른 사람에게 전달합니다. 다음의 단계에서 계속이 goroutine을 실행합니다.

> 기본적으로 배차 프로세스를 사용만으로 멀티 스레드를 제공 할뿐입니다. 멀티 코어 프로세서의 멀티 스레드를 실현하려면 우리의 프로그램에서 runtime.GOMAXPROCS (n)를 호출함으로써 발송자에 동시에 여러 프로세스를 사용해야합니다. GOMAXPROCS 동시에 실행하는 로직 코드 시스템 프로세스의 최대 개수를 설정하고 이전 설정을 반환합니다. 만약 n <1 인 경우 현재 설정은 변경되지 않습니다. Go의 새로운 버전으로 배차가 수정되면, 이것은 삭제 될 것입니다. Rob 멀티 스레드의 개발은 여기에 문장이 있습니다. http://concur.rspace.googlecode.com/hg/talk/concur.html#landing-slide

## channels
goroutine은 동일한 주소 공간에서 실행됩니다. 따라서 공유 된 메모리에 대한 액세스는 반드시 동기화되어 있지 않으면 안됩니다. 는 goroutine 사이에서 어떻게 데이터 통신을 할 것입니까. Go는 채널이라는 아주 좋은 통신 메커니즘을 제공합니다. 채널은 Unix shell과의 양방향 파이프를 만듭니다. 이를 통해 값을 보내거나받을 수 있습니다. 이 값은 특정 형태 만 허용됩니다. 채널 형. channel을 정의하면 채널에 전송하는 값의 형식도 정의해야합니다. 주의하시기 바랍니다. 반드시 make를 사용해 channel을 만듭니다.

ci : = make (chan int)
cs : = make (chan string)
cf : = make (chan interface {})

channel은`<-` 연산자를 사용하여 데이터를 보내거나 받거나합니다.

ch <- v // v를 channel ch에 보낸다.
v : = <-ch // ch 속에서 데이터를 받아 v에 대입한다.

이것을 우리의 예에 적용시켜 봅시다 :

package main

import "fmt"

func sum (a [] int, c chan int) {
total : = 0
for _ v : = range a {
total + = v
}
c <- total // send total to c
}

func main () {
a : = [] int {7, 2, 8, -9, 4, 0}

c : = make (chan int)
go sum (a [: len (a) / 2, c)
go sum (a [len (a) / 2 :, c)
x, y : = <-c <-c // receive from c

fmt.Println (x, y, x + y)
}

기본적으로 channel이주고받는 데이터는 차단되어 있습니다. 다른 하나는 준비되어 있지 않으면 Goroutines 동기화는 더 쉬워집니다. lock을 표시 할 필요가 없습니다. 이른바 블록은 만약 (value : = <-ch)를 읽은 경우,이 차단됩니다. 데이터를받은 단계에서 (ch <-5)를 보낼 어느 것도 데이터가 읽힐 때까지 블록됩니다. 버퍼링이없는 channel은 여러 goroutine 간의 동기화를 할 매우 훌륭한 도구입니다.

## Buffered Channels
위에서는 기본적으로 버퍼링 형이없는 channel을 소개했습니다. 그러나 Go는 channel 버퍼링 대소 지정하는 것을 허락합니다. 매우 간단합니다. 즉 channel은 여러 가지 요소를 저장할 수 있습니다. ch : = make (chan book 4)는 4 개의 bool 형의 요소를 가질 channel을 만듭니다. 이 channel에서 이전의 4 가지 요소는 블록되지 않고 입력 할 수 있습니다. 다섯 번째 요소가 입력 된 경우 코드는 차단 된 다른 goroutine이 channel에서 요소를 꺼낼 때까지 공간을 대피합니다.

ch : = make (chan type, value)

value == 0! 버퍼링 없음 (블록)
value> 0! 버퍼링 (블록 없음, value 개의 요소까지)

아래의 예를 참조하십시오. 로컬에서 테스트 해 볼 수 있습니다. 대응하는 value 값을 변경하십시오


package main

import "fmt"

func main () {
c : = make (chan int 2) // 2를 1로 수정하면 오류가 발생합니다. 2를 3으로 수정하면 정상적으로 실행됩니다.
c <- 1
c <- 2
fmt.Println (<- c)
fmt.Println (<- c)
}

## Range와 Close
위의 예에서는 2 번 c의 값을 가져와야했습니다. 이제 그다지 유용하지 않습니다. Go는이 점을 고려하여 range 의해 slice 나 map을 조작하는 것과 같은 감각으로 버퍼링 형의 channel을 조작 할 수 있습니다. 아래의 예를 참조하십시오.

package main

import (
"fmt"
)

func fibonacci (n int, c chan int) {
x, y : = 1 1
for i : = 0; i <n; i ++ {
c <- x
x, y = y, x + y
}
close (c)
}

func main () {
c : = make (chan int 10)
go fibonacci (cap (c) c)
for i : = range c {
fmt.Println (i)
}
}

`for i : = range c`에서이 channel이 닫혀을 명시 될 때까지 연속해서 channel의 데이터를로드 할 수 있습니다. 위의 코드에서 channel의 폐쇄가 명시되어있는 것을 확인할 수있다라고 생각합니다. 생산자는`close` 내장 함수에 의해 channel을 닫습니다. channel을 닫은 후에는 어떠한 데이터도 보낼 수 없습니다. 소비 측은`v, ok : = <-ch`는 식으로 channel이 이미 닫혀 있는지 테스트 할 수 있습니다. 만약 ok false가 돌아 오면 channel은 이미 어떤 데이터도없고, 닫혀 있다는 것입니다.

> 생산자 쪽에서 channel이 닫힐 점에주의하십시오. 소비 측면에서 없습니다. 이것은 쉽게 panic이 발생합니다.

> 또한 channel은 파일과 같은 것이 아님에 유의하십시오. 자주 닫을 필요가 없습니다. 어떤 데이터도 보낼 수없는 경우 또는 range 루프를 종료시키고 싶은 경우 등 괜찮아요.

## Select
여기에서 하나 뿐인 channel이있는 상황에 대해 소개했습니다. 에서는 여러 channel이 존재하면 어떻게 조작해야할까요? Go는 키워드`select`을 제공합니다. `select` 통해 channel의 데이터를 모니터링 할 수 있습니다.

`select`은 기본적으로 차단됩니다. channel에서 교환되는 데이터를 모니터링 할 때만 실행합니다. 여러 channel 준비가되었을 때, select는 무작위로 하나 선택하고 실행합니다.

package main

import "fmt"

func fibonacci (c, quit chan int) {
x, y : = 1 1
for {
select {
case c <- x :
x, y = y, x + y
case <-quit :
fmt.Println ( "quit")
return
}
}
}

func main () {
c : = make (chan int)
quit : = make (chan int)
go func () {
for i : = 0; i <10; i ++ {
fmt.Println (<- c)
}
quit <- 0
} ()
fibonacci (c, quit)
}

`select` 중에도 default 문이 있습니다. `select`은 사실 switch의 기능과 유사합니다. default는 감시하고있는 channel이 모두 준비가되어 있지 않을 때 기본적으로 실행됩니다. (select는 channel을 기다리고 차단하지 않습니다.)

select {
case i : = <-c :
// use i
default :
// c가 차단 된 때 여기가 실행됩니다.
}

## 시간
가끔 goroutine이 차단되는 상황에 마주합니다. 는 프로그램 전체가 차단되는 상황을 어떻게 피할 수 있을까요? select를 사용하여 시간을 설정 할 수 있습니다. 아래와 같은 방법으로 제공합니다 :

func main () {
c : = make (chan int)
o : = make (chan bool)
go func () {
for {
select {
case v : = <- c :
println (v)
case <- time.After (5 * time.Second) :
println ( "timeout")
o <- true
break
}
}
} ()
<- o
}


## runtime goroutine
runtime 패키지는 goroutine을 처리하는 몇 가지 기능이 포함되어 있습니다 :

- Goexit

사전에 실행 된 goroutine에서 빠져 있습니다. 그러나 defer 함수는 계속해서 호출됩니다.

- Gosched

사전 goroutine의 실행 권한을 제네 레이트합니다. 디스패처가 다른 대기중인 작업의 실행을 계획하고 다음의 어느 시점에서이 위치에서 실행을 복원합니다.

- NumCPU

CPU의 코어 수를 돌려줍니다.

- NumGoroutine

현재 실행중인 행과 대기 작업의 수를 돌려줍니다.

- GOMAXPROCS

실행할 수있는 CPU 코어 수의 최대 값을 설정하고 이전의 코어 수를 돌려줍니다.



