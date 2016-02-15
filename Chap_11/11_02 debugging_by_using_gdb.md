# Debugging by using GDB

프로그램을 개발함에있어 개발자는 종종 디버깅 코드를 작성해야합니다. Go 언어는 PHP 나 Python 같은 동적 언어 컴파일러를 필요로하지 않고 수정하는 것만으로 직접 출력하여 동적으로 실행 환경에서 데이터를 출력 할 수있는 것은 아닙니다. 당연히 Go 언어도 Println처럼 데이터를 출력하여 디버깅 할 수 있지만 매번 재 컴파일해야합니다. 이것은 매우 귀찮은 것입니다. Python에서 pdb / ipdb 같은 도구를 통해 디버깅을 할 수 있으며, Javascript에도 비슷한 도구가 있습니다. 이러한 도구는 모두 동적으로 변수 정보를 표시하는 것이나, 단계 실행 수 있습니다. 우리는 GDB를 이용한 디버깅 할 수 있습니다. 은이 절에서는 어떻게하고 GDB에 의해 Go 프로그램을 디버깅 할 것인가 소개합시다.

## GDB 디버깅에 대한 간략한 소개
GDB는 FSF (자유 소프트웨어 재단)이 배포하고있는 강력한 UNIX 시스템의 프로그램 디버깅 도구입니다. GDB를 사용하여 다음과 같은 수 있습니다 :

1. 프로그램을 시작하고 개발자가 정의한 요구에 따라 프로그램을 실행할 수 있습니다.
2. 디버그되는 프로그램은 개발자 설정 한 중단 점에서 중지 할 수 있습니다. (중단 점은 조건식 수 있습니다.)
3. 프로그램이 정지했을 때, 이때 프로그램에서 발생하는 일들을 검사 할 수 있습니다.
4. 동적으로 현재 프로그램의 실행 환경을 변경할 수 있습니다.

현재 Go 프로그램의 디버깅을 지원하는 GDB 버전은 7.1 이상입니다.

Go 프로그램을 컴파일 할 때 다음 몇 가지에주의하십시오

1. 매개 변수는 -ldflags "-s"는 debug 정보의 출력을 생략합니다.
2. -gcflags "-N -l"매개 변수는 Go의 내부에서 행해지는 몇 가지 최적화를 무시할 수 있습니다. 집성 체형 변수와 함수의 최적화입니다. 이들은 GDB 디버깅이 매우 어렵습니다 때문에 컴파일 할 때이 두 가지 매개 변수를 추가하여 최적화를 피합니다.

## 자주 사용하는 명령
GDB에서 자주 사용하는 명령의 일부는 다음과 같습니다

- list

`l`과 생략됩니다. 소스 코드를 표시하는 데 사용됩니다. 기본적으로 10 줄의 코드를 표시합니다. 뒤에 표시하는 구체적인 행 매개 변수로 전달할 수 있습니다. 예를 들면 :`list 15`는 10 줄의 코드를 표시하고 다음과 같이 15 행 10 행 중 중심에 표시됩니다.

10 time.Sleep (2 * time.Second)
11 c <- i
12}
13 close (c)
14}
15
16 func main () {
17 msg : = "Starting main"
18 fmt.Println (msg)
19 bus : = make (chan int)


- break

`b`와 생략됩니다. 중단 점을 설정하는 데 사용됩니다. 뒤에 브레이크 포인트를 줄을 매개 변수로 추가합니다. 예를 들어`b 10`에서는 10 번째 줄에 중단 점을두고 있습니다.

- delete
`d`와 생략됩니다. 중단 점을 제거하는 데 사용됩니다. 뒤에 브레이크 포인트의 번호가 붙습니다. 이 번호는`info breakpoints` 의해 해당 설정된 중단 점 번호를 얻을 수 있습니다. 다음은 설정된 브레이크 포인트의 번호를 표시합니다.

Num Type Disp Enb Address What
2 breakpoint keep y 0x0000000000400dc3 in main.main at /home/xiemengjun/gdb.go:23
breakpoint already hit 1 time

- backtrace

`bt`과 생략됩니다. 다음과 같이 실행중인 코드의 과정을 출력하기 위해 사용됩니다 :

# 0 main.main () at /home/xiemengjun/gdb.go:23
# 1 0x000000000040d61e in runtime.main () at /home/xiemengjun/go/src/pkg/runtime/proc.c:244
# 2 0x000000000040d6c1 in schedunlock () at /home/xiemengjun/go/src/pkg/runtime/proc.c:267
# 3 0x0000000000000000 in ?? ()
- info

info 명령은 정보를 표시합니다. 뒤에 몇 가지 매개 변수가 있습니다. 자주 사용되는 것은 다음의 몇 가지 있습니다 :

-`info locals`

현재 실행중인 프로그램의 변수 값을 표시합니다.
-`info breakpoints`

현재 설정되어있는 브레이크 포인트 목록을 표시합니다.
-`info goroutines`

현재 실행중인 goroutine의 목록을 표시합니다. 다음 코드에서 알 수 있듯이 *가 붙어있는 것은 현재 실행하고있는 것입니다.

* 1 running runtime.gosched
* 2 syscall runtime.entersyscall
3 waiting runtime.gosched
4 runnable runtime.gosched
- print

`p`와 생략됩니다. 변수 또는 기타 정보를 표시하는 데 사용됩니다. 뒤에 출력해야하는 변수 이름이 추가됩니다. 당연히 매우 사용하기 쉬운 함수 $ len ()와 $ cap ()도 있습니다. 현재 string, slices 또는 maps의 길이와 용량을 돌려 주는데 사용됩니다.

- whatis

현재 변수의 형태를 표시하는 데 사용됩니다. 뒤에 변수 이름이 붙습니다. 예를 들어`whatis msg`는 다음과 같이 나타납니다 :

type = struct string
- next

`n`과 생략됩니다. 단계 실행에 사용됩니다. 다음 단계로 이동합니다. 중단 점이 있으면`n`을 입력하여 다음 단계까지 계속 수행 할 수 있습니다.
- coutinue

`c`로 축약됩니다. 현재 브레이크 포인트에서 빠져 있습니다. 뒤에 파라미터를 붙이는 것으로, 여러 번 중단 점을 뛰어 넘을 수 있습니다.

- set variable

이 명령은 실행중인 변수의 값을 변경하는 데 사용됩니다. 형식은 다음과 같습니다 :`set variable <var> = <value>`

## 디버깅 과정
다음 코드는 어떻게 GDB를 사용하여 Go 프로그램을 디버깅 할 것인가 데모를 실시합니다. 다음은 데모 코드입니다 :

package main

import (
"fmt"
"time"
)

func counting (c chan <- int) {
for i : = 0; i <10; i ++ {
time.Sleep (2 * time.Second)
c <- i
}
close (c)
}

func main () {
msg : = "Starting main"
fmt.Println (msg)
bus : = make (chan int)
msg = "starting a gofunc"
go counting (bus)
for count : = range bus {
fmt.Println ( "count :", count)
}
}

파일을 컴파일하고 실행 파일 gdbfile을 생성합니다 :

go build -gcflags "-N -l"gdbfile.go

gdb 명령으로 디버깅을 시작합니다 :

gdb gdbfile

시작하면 우선이 프로그램을 실행할 수 있을지 살펴 보자. `run` 명령을 입력하고 엔터 키를 누르면 프로그램이 실행됩니다. 프로그램이 성공적이면, 프로그램은 다음과 같이 출력합니다. 명령 줄에서 직접 프로그램을 실행 한 것과 동일합니다 :

(gdb) run
Starting program : / home / xiemengjun / gdbfile
Starting main
count : 0
count : 1
count : 2
count : 3
count : 4
count : 5
count : 6
count : 7
count : 8
count : 9
[LWP 2771 exited]
[Inferior 1 (process 2771) exited normally]
좋아, 프로그램을 어떻게 시작하는지 알 수있었습니다. 다음 프로그램에 중단 점을 설정합니다 :

(gdb) b 23
Breakpoint 1 at 0x400d8d : file /home/xiemengjun/gdbfile.go, line 23.
(gdb) run
Starting program : / home / xiemengjun / gdbfile
Starting main
[New LWP 3284]
[Switching to LWP 3284]

Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
23 fmt.Println ( "count :", count)

위의 예에서`b 23`에서 23 번째 줄에 중단 점을 설정했습니다. 그 후`run`를 입력하면 프로그램이 시작됩니다. 현재 프로그램은 이전에 설정된 중단 점에서 중지합니다. 브레이크 포인트에 대응하는 컨텍스트의 소스 코드를 알기 위해서는`list`를 입력하여 소스 코드가 현재 정지하고있는 행의 이전 5 줄에서 표시 할 수 있습니다 :

(gdb) list
18 fmt.Println (msg)
19 bus : = make (chan int)
20 msg = "starting a gofunc"
21 go counting (bus)
22 for count : = range bus {
23 fmt.Println ( "count :", count)
24}
25}

GDB가 실행중인 프로그램의 현재 환경에 몇 가지 유용한 디버깅 정보를 가지고 있습니다. 해당 변수를 출력하기 만하면 해당 변수의 타입과 값을 확인 할 수 있습니다 :

(gdb) info locals
count = 0
bus = 0xf840001a50
(gdb) p count
$ 1 = 0
(gdb) p bus
$ 2 = (chan int) 0xf840001a50
(gdb) whatis bus
type = chan int

다음에이 프로그램을 계속 실행시키고 유지해야합니다. 다음 명령을 참조하십시오

(gdb) c
Continuing.
count : 0
[New LWP 3303]
[Switching to LWP 3303]

Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
23 fmt.Println ( "count :", count)
(gdb) c
Continuing.
count : 1
[Switching to LWP 3302]

Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
23 fmt.Println ( "count :", count)

매번`c`를 입력 할 때마다 한번의 코드가 실행됩니다. 다음 for 루프 점프하여 계속 해당 정보를 출력합니다.

현재 컨텍스트 관련 변수의 정보를 변경하고자합니다. 일부 프로세스를 뛰어 넘어 계속 다음 단계를 수행하여 수정을 한 후에 원하는 결과를 얻을 수 있습니다 :

(gdb) info locals
count = 2
bus = 0xf840001a50
(gdb) set variable count = 9
(gdb) info locals
count = 9
bus = 0xf840001a50
(gdb) c
Continuing.
count : 9
[Switching to LWP 3302]

Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
23 fmt.Println ( "count :", count)

마지막에 약간 생각하자. 이전 프로그램의 실행의 전 과정에서 몇의 gorutine가 만든 있을까요? 각 goroutine은 무엇을하고있는 것일까 요 :

(gdb) info goroutines
* 1 running runtime.gosched
* 2 syscall runtime.entersyscall
3 waiting runtime.gosched
4 runnable runtime.gosched
(gdb) goroutine 1 bt
# 0 0x000000000040e33b in runtime.gosched () at /home/xiemengjun/go/src/pkg/runtime/proc.c:927
# 1 0x0000000000403091 in runtime.chanrecv (c = void, ep = void, selected = void, received = void)
at /home/xiemengjun/go/src/pkg/runtime/chan.c:327
# 2 0x000000000040316f in runtime.chanrecv2 (t = void, c = void)
at /home/xiemengjun/go/src/pkg/runtime/chan.c:420
# 3 0x0000000000400d6f in main.main () at /home/xiemengjun/gdbfile.go:22
# 4 0x000000000040d0c7 in runtime.main () at /home/xiemengjun/go/src/pkg/runtime/proc.c:244
# 5 0x000000000040d16a in schedunlock () at /home/xiemengjun/go/src/pkg/runtime/proc.c:267
# 6 0x0000000000000000 in ?? ()

goroutines 명령을 확인하여 goroutine의 내부가 어떻게 실행되고 있는지 더 잘 이해할 수 있습니다. 각 함수 호출되는 순서는 이미 명확하게 표시되어 있습니다.

## 정리
이 장에서는 GDB 디버깅의 Go 프로그램의 기본 명령의 일부를 소개했습니다. `run``print``info``set variable``continue``list``break` 등 자주 사용하는 디버그 명령을 포함하여 위의 데모에서 그랬던 것처럼 독자는 이미 Go 프로그램에 대해 GDB를 이용한 디버깅을 기본적으로 이해 한 것으로 믿고 있습니다. 만약 더 많은 디버깅 기법을 알고 싶다면 공식 페이지 GDB 디버깅 항목을 참조하십시오.

