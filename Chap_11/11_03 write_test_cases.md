# Write Test Cases

프로그램의 개발에서 테스트는 매우 중요합니다. 어떻게 코드의 품질을 보증하거나 어떻게 각 함수가 실행될 수 있음을 보증하거나 또 쓴 코드의 성능이 좋은 것을 어떻게 보장 할 것인가입니다. 우리는 단위 테스트는 주로 프로그램의 설계 및 구현 로직 오류를 발견하는 것이라고 알고 있습니다. 문제를 조기에 발견하고 문제를 파악하고 해결 케 성능을 테스트하기 위해 프로그램 설계 문제의 일부를 발견하여 온라인 프로그램이 다중 처리하고있는 상황에서도 안정을 유지할 수 있도록 있습니다. 이 절에서는 이러한 일련의 문제에서 Go 언어에서 어떻게 단위 테스트와 성능 테스트를 제공하거나 소개합니다.

Go 언어는 미리 준비되어있는 가벼운 테스트 프레임 워크`testing`와`go test` 명령을 사용하여 단위 테스트와 성능 테스트를 제공합니다. `testing` 프레임 워크 및 기타 언어로 테스트 프레임 워크는 비슷합니다. 이 프레임 워크를 기반으로 해당 기능에 대한 테스트를 쓸 수 있습니다. 또한이 프레임 워크를 기반으로 대응하는 내구성 테스트를 작성할 수 있습니다. 는 어떻게 만드는가 하나 하나 살펴 보도록합시다.

## 어떻게 테스트를 쓰는가?
`go test` 명령은 해당 디렉토리 아래의 모든 파일을 실행 할 수 밖에 없습니다. 따라서`gotest`라는 디렉토리를 새로 생성하여 모든 코드와 테스트 코드를이 디렉토리에 배치 할 수 있습니다.

다음 디렉토리 아래에 2 개의 파일을 새로 만듭니다 : gotest.go과 gotest_test.go

1. gotest.go :이 파일에는 패키지를 하나 씁니다. 내용은 나눗셈을 수행하는 함수가 하나 있습니다 :

package gotest

import (
"errors"
)

func Division (a, b float64) (float64, error) {
if b == 0 {
return 0, errors.New ( "제수는 0 이외가 아니면 안됩니다")
}

return a / b, nil
}

2. gotest_test.go : 이것은 단위 테스트 파일이지만 다음과 같은 원칙을 기억하십시오 :

- 파일 이름은 반드시`_test.go`가 마지막에 띄게하십시오. 이에 따라`go test`을 실행했을 때 해당 코드가 실행되게됩니다.
-`testing`라는 패키지를 import해야합니다.
- 모든 테스트 함수 이름은`Test`에서 시작됩니다.
- 테스트는 소스 코드에 적힌 순서대로 실행됩니다.
- 테스트 함수`TestXxx ()`의 매개 변수는`testing.T`입니다. 이 형식을 사용하여 오류 및 테스트의 상태를 기록 할 수 있습니다.
- 테스트 형식 :`func TestXxx (t * testing.T)``Xxx` 부분은 임의의 숫자의 조합입니다. 그러나 글자는 소문자 [a-z]는 안됩니다 예를 들어`Testintdiv`하는 것은 잘못된 함수 이름입니다.
- 함수는`testing.T`의`Error``Errorf``FailNow``Fatal``FatalIf` 메소드를 호출하여 테스트가 통과하지 못하는 것을 설명합니다. `Log` 메소드를 호출하여 테스트 정보를 기록합니다.

다음은 우리의 테스트 코드입니다 :

package gotest

import (
"testing"
)

func Test_Division_1 (t * testing.T) {
if i e : = Division (6, 2); i! = 3 || e! = nil {// try a unit test on function
t.Error ( "나누기 함수의 테스트를 통해 전달되지 않음") // 만약 예정된 것이어야 오류를 발생시킵니다.
} else {
t.Log ( "시작 테스트에 통과했습니다") // 기록하고 싶은 정보를 기록합니다
}
}

func Test_Division_2 (t * testing.T) {
t.Error ( "경로하지 않습니다")
}

프로젝트 디렉토리에서`go test`을 실행하면 다음과 같은 정보가 표시됩니다 :

--- FAIL : Test_Division_2 (0.00 seconds)
gotest_test.go : 16 : 패스하지 않습니다
FAIL
exit status 1
FAIL gotest 0.013s
이 결과에서 알 수 있듯이 테스트를 통과하지 못한 것은 두 번째 테스트 함수 테스트가 통하지 않는 코드`t.Error`을 쓰고 있었기 때문입니다. 는 첫 번째 함수가 실행 상황은 어떻습니까? 기본적으로`go test`를 실행하면 테스트에 통과하는 정보는 표시되지 않습니다. `go test -v`라는 옵션을 추가해야합니다. 이렇게하면 다음과 같은 정보가 표시됩니다 :

=== RUN Test_Division_1
--- PASS : Test_Division_1 (0.00 seconds)
gotest_test.go : 11 : 첫 번째 테스트에 통과
=== RUN Test_Division_2
--- FAIL : Test_Division_2 (0.00 seconds)
gotest_test.go : 16 : 패스하지 않습니다
FAIL
exit status 1
FAIL gotest 0.012s
위의 출력이 테스트 과정을 상세하게 표시하고 있습니다. 테스트 함수 1`Test_Division_1`에서는 테스트가 같이했습니다. 그러나 함수 2`Test_Division_2` 테스트는 실패했습니다. 마지막으로 테스트가 통과한다는 결론을 얻었습니다. 이상에서 테스트 함수 2를 다음과 같은 코드로 수정합니다 :

func Test_Division_2 (t * testing.T) {
if _ e : = Division (6, 0); e == nil {// try a unit test on function
t.Error ( "Division did not work as expected") // 예상 한 것이어야 오류 발생
} else {
t.Log ( "one test passed.", e) // 기록하고 싶은 정보를 기록
}
}
그 후`go test -v`을 실행하면 다음과 같은 정보를 표시하고 테스트에 통과합니다 :

=== RUN Test_Division_1
--- PASS : Test_Division_1 (0.00 seconds)
gotest_test.go : 11 : 첫 번째 테스트에 통과
=== RUN Test_Division_2
--- PASS : Test_Division_2 (0.00 seconds)
gotest_test.go : 20 : one test passed. 제수는 0이 아닌
PASS
ok gotest 0.013s

## 어떻게하고 내구성 테스트를 쓰는가?
내구성 테스트 함수 (메소드)의 성능을 측정하는 데 사용됩니다. 여기에 재게하지 않지만, 단위 테스트를 작성하는 것과 같은 것입니다. 그러나 다음의 몇 가지에주의해야합니다 :

- 내구성 테스트는 다음 루프의 형식으로 이루어져야합니다. 이 중 XXX는 임의의 숫자의 조합입니다. 그러나 글자는 소문자로는 안됩니다.

func BenchmarkXXX (b * testing.B) {...}

-`go test`는 기본적으로 내구성 테스트 기능을 수행하지 않습니다. 만약 내구성 테스트를 실행하려면 옵션`-test.bench`을 추가합니다. 문법 :`-test.bench = "test_name_regex"`. 예를 들어`go test -test.bench = ". *"`모든 내구성 테스트 함수를 테스트하는 것을 나타냅니다
- 내구성 테스트에서는 테스트가 성공적으로 실행되도록 루프 속에서`testing.B.N`를 사용하는 것을 기억하십시오
- 파일 이름은 반드시`_test.go`로 끝납니다

이하에서는 webbench_test.go라는 내구성 테스트 파일을 만듭니다. 코드는 다음과 같습니다 :

package gotest

import (
"testing"
)

func Benchmark_Division (b * testing.B) {
for i : = 0; i <b.N; i ++ {// use b.N for looping
Division (4, 5)
}
}

func Benchmark_TimeConsumingFunction (b * testing.B) {
b.StopTimer () // 调用 该函 수 정지 压力 测试으로 시간 计数

// 做 한 些初 始化으로 공작 예 如读 取文 건수 据 몇 据库 连接 之 类的,
// 这样 这些 시간 不影 响我 们测 试函 몇 개의 몸으로 성능

b.StartTimer () // 무게 새로운 개시 시간
for i : = 0; i <b.N; i ++ {
Division (4, 5)
}
}


`go test -file webbench_test.go -test.bench = ". *"`라는 명령을 실행하면 다음과 같은 결과가 나타납니다 :

PASS
Benchmark_Division 500000000 7.76 ns / op
Benchmark_TimeConsumingFunction 500000000 7.80 ns / op
ok gotest 9.364s

위의 결과는 우리가 어떤`TestXXX` 단위 테스트 함수를 실행하지 않는 것을 보여줍니다. 표시되는 결과는 스트레스 테스트 함수 만 실행했을뿐입니다. 첫째 줄에는`Benchmark_Division`가 500000000 번 실행됩니다 보여 매번 실행이 평균 7.76 밀리 초 였음을 보여줍니다. 둘째 줄은`Benchmark_TimeConsumingFunctin`가 500000000 번 실행됩니다 매회 평균 실행 시간이 7.80 밀리 초 였음을 보여줍니다. 마지막 행은 전체 실행 시간을 보여줍니다.

## 정리
위의 단위 테스트와 스트레스 테스트의 학습을 통해`testing` 패키지가 매우 가볍고, 단위 테스트와 스트레스 테스트를 쓰는 것은 매우 간단하다 알았습니다. 내장의`go test` 명령을 결합하여 매우 편리하게 테스트를 할 수 있습니다. 이처럼 우리가 매번 코드를 수정 끝날 때마다 go test를 실행하는 것만으로 간단하게 회귀 테스트를 할 수 있습니다.


