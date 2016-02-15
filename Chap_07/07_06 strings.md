# Strings

문자열은 우리가 일반적으로 Web 개발에서 자주 사용하는 것입니다. 사용자의 입력이나 데이터베이스에서 데이터의 읽기 등을 포함하여 우리는 자주 문자열을 분할 연결, 변환 등을 수행합니다. 이 절에서는 Go 표준 라이브러리에있는 strings와 strconv 두 가지 패키지 함수를 사용하여 어떻게 신속하게 작업을 수행 바구니 설명합니다.
## 문자열 조작
다음 함수는 strings 패키지에 들어 있습니다. 여기에 평소 자주 사용하는 기능을 일부 소개합니다. 자세한 내용은 공식 문서를 참조하십시오.

- func Contains (s, substr string) bool

문자열 s에 substr가 포함되는지 판단합니다. bool 값을 반환합니다.

fmt.Println (strings.Contains ( "seafood" "foo"))
fmt.Println (strings.Contains ( "seafood", "bar"))
fmt.Println (strings.Contains ( "seafood", ""))
fmt.Println (strings.Contains ( "", ""))
// Output :
// true
// false
// true
// true

- func Join (a [] string, sep string) string

문자열 연결. slice a에 sep에 연결합니다.

s : = [] string { "foo", "bar", "baz"}
fmt.Println (strings.Join (s, ""))
// Output : foo, bar, baz

- func Index (s, sep string) int

문자열 s에서 sep가 존재하는 위치입니다. 인덱스를 돌려줍니다. 발견되지 않으면 -1을 반환합니다.

fmt.Println (strings.Index ( "chicken", "ken"))
fmt.Println (strings.Index ( "chicken", "dmr"))
// Output : 4
// - 1

- func Repeat (s string, count int) string

s 문자열을 count 회 반복합니다. 마지막으로 반복 된 문자열을 반환합니다.

fmt.Println ( "ba"+ strings.Repeat ( "na", 2))
// Output : banana

- func Replace (s, old, new string, n int) string

s 문자열에서 old 문자열을 new 문자열로 대체합니다. n은 대체 횟수를 나타냅니다. 0 이하에서는 모든 대체합니다.

fmt.Println (strings.Replace ( "oink oink oink", "k", "ky", 2))
fmt.Println (strings.Replace ( "oink oink oink", "oink" "moo", -1))
// Output : oinky oinky oink
// moo moo moo

- func Split (s, sep string) [] string

sep에 의해 s 문자열을 분할합니다. slice를 반환합니다.

fmt.Printf ( "% q \ n", strings.Split ( "a, b, c", ""))
fmt.Printf ( "% q \ n", strings.Split ( "a man a plan a canal panama", "a"))
fmt.Printf ( "% q \ n", strings.Split ( "xyz", ""))
fmt.Printf ( "% q \ n", strings.Split ( "", "Bernardo O'Higgins"))
// Output : [ "a" "b" "c"]
// "" "man" "plan" "canal panama"]
// "" "x" "y" "z" ""]
// [ ""]

- func Trim (s string, cutset string) string

s 문자열의 시작과 끝에서 cutset에서 지정한 문자열을 제거한다.

fmt.Printf ( "[% q]"strings.Trim ( "!!! Achtung!" "!"))
// Output : "Achtung"]

- func Fields (s string) [] string

s 문자열의 공백을 제거하고 빈에 따라 분할 된 slice를 반환합니다.

fmt.Printf ( "Fields are : % q"strings.Fields ( "foo bar baz"))
// Output : Fields are : [ "foo" "bar" "baz"]


## 문자열 변환
문자열을 변환하는 함수는 strconv에 있습니다. 다음은 그 중 자주 사용되는 것들의 목록에 지나지 않습니다 :

- Append 시리즈의 함수는 정수 등을 문자열로 변환 한 후, 현재의 바이트 열에 추가합니다.

package main

import (
"fmt"
"strconv"
)

func main () {
str : = make ([] byte, 0, 100)
str = strconv.AppendInt (str, 4567, 10)
str = strconv.AppendBool (str, false)
str = strconv.AppendQuote (str, "abcdefg")
str = strconv.AppendQuoteRune (str, '单')
fmt.Println (string (str))
}

- Format 시리즈의 함수는 다른 형태를 문자열로 변환합니다.

package main

import (
"fmt"
"strconv"
)

func main () {
a : = strconv.FormatBool (false)
b : = strconv.FormatFloat (123.23 'g', 12, 64)
c : = strconv.FormatInt (1234, 10)
d : = strconv.FormatUint (12345 10)
e : = strconv.Itoa (1023)
fmt.Println (a, b, c, d, e)
}

- Parse 시리즈의 함수는 문자열을 다른 형식으로 변환합니다.

package main

import (
"fmt"
"strconv"
)
func checkError (e error) {
if e! = nil {
fmt.Println (e)
}
}
func main () {
a, err : = strconv.ParseBool ( "false")
checkError (err)
b, err : = strconv.ParseFloat ( "123.23"64)
checkError (err)
c, err : = strconv.ParseInt ( "1234", 10, 64)
checkError (err)
d, err : = strconv.ParseUint ( "12345", 10, 64)
checkError (err)
e, err : = strconv.Atoi ( "1023")
checkError (err)
fmt.Println (a, b, c, d, e)
}

