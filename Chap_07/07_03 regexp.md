# Regexp

정규 표현식은 패턴 매치와 텍스트 조작 복잡하고 강력한 도구입니다. 정규 표현식은 순수 텍스트 매칭에 비해 효율은 떨어지지 만보다 유연 성이 풍부합니다. 이 문법 규칙에 따라 생성되는 패턴은 기존의 텍스트에서 당신이 필요로하는 거의 모든 문자열의 조합을 필터링 할 수 있습니다. 만약 Web 개발에서 뭔가의 텍스트 데이터 소스에서 데이터를 꺼낼 필요가 있다면,이 문법 규칙에 따라 정확한 패턴 문자열을 만들기위한 의미있는 텍스트 정보를 데이터 소스에서 검색 할 수 있습니다 .

Go 언어는`regexp` 표준 패키지를 사용하여 공식에 정규 표현식을 지원합니다. 만약 당신이 다른 프로그래밍 언어에서 제공되는 정규 표현식과 동일한 기능을 사용한 적이있는 것이면, Go 언어 버전에서도 그렇게 문외한이라는 것은 없을 것입니다. 그러나 이들 사이에서도 약간의 차이가 있습니다. 왜냐하면 Go가 구현하고있는 것은 RE2 표준에서 \ C를 제외하고 자세한 문법 설명은 다음을 참조하십시오 :`http : // code.google.com / p / re2 / wiki / Syntax`

문자열 처리는 원래`strings` 패키지를 사용하여 검색 (Contains, Index) 대체 (Replace)와 가이세키 (Split, Join) 등 작업을 할 수있었습니다. 그러나 이들은 모두 간단한 문자열 조작에 지나지 않습니다. 이러한 검색은 모두 대문자와 소문자를 구별하고 고정 된 문자열입니다. 만약 가변의 이러한 매칭을 할 필요가 있다면, 실현 방법이 없습니다. 당연히 만약`strings` 패키지가 당신의 문제를 해결할 수 있다면 가능한 한이를 사용하여 해결해야합니다. 왜냐하면이 간단하고 성능과 가독성도 정규식에 비해 좋기 때문입니다.

이전 폼 검증 절에서 이미 정규 표현식을 만진 것을 기억하고 계실지도 모릅니다. 그때는 이것을 사용해 입력 된 정보가 어떠한 미리 설정된 조건을 만족하는지 검증하는 데 사용했습니다. 사용시주의해야 할 것은 : 어떤 문자열 역시 UTF-8로 인코딩되어 있다는 것입니다. 이후에 더 깊이 Go 언어의`regexp` 패키지 관련 지식을 배우고 갑시다.

## 정규 표현식을 사용하여 일치하는지 판단하는
`regexp` 패키지는 3 개의 함수를 사용하여 매칭을 판단합니다. 만약 일치하면 true를 반환하고 그렇지 않으면 false를 반환합니다.

func Match (pattern string, b [] byte) (matched bool, error error)
func MatchReader (pattern string, r io.RuneReader) (matched bool, error error)
func MatchString (pattern string, s string) (matched bool, error error)

위의 3 가지 기능 같은 기능을 실현하고 있습니다. 즉,`pattern`이 입력 소스와 일치 하는지를 판단하고 있습니다. 일치하면 true를 반환하거나 정규 표현식 파싱 오류가 나오면 error를 반환합니다. 3 개의 함수의 입력 소스는 각각 byte slice, RuneReader과 string입니다.

입력이 IP 주소인지 확인하려면 어떻게 판단해야할까요? 다음을 참조하십시오

func IsIP (ip string) (b bool) {
if m _ : = regexp.MatchString ( "^ [0-9] {1,3} \\. [0-9] {1,3} \\. [0-9] {1,3} \\ [0-9] {1,3} $ ", ip);! m {
return false
}
return true
}

보시다시피`regexp`의 pattern과 우리가 일반적으로 사용하는 정규 표현식은 똑입니다. 또 다른 예를 살펴 봅시다 : 사용자가 문자열을 입력하고이 입력이 올바른지 알고 싶은 것으로합니다 :

func main () {
if len (os.Args) == 1 {
fmt.Println ( "Usage : regexp [string]")
os.Exit (1)
} else if m _ : = regexp.MatchString ( "^ [0-9] + $", os.Args [1]); m {
fmt.Println ( "숫자입니다.")
} else {
fmt.Println ( "숫자가 아닙니다.")
}
}

위의 두 예제에서는 Match (Reader | String)를 사용하여 문자열이 우리의 요구에 부합하는지 판단하고 있습니다. 이들은 매우 편리합니다.

## 정규식을 사용하여 내용을 얻을
Match 패턴은 문자열의 판단에 대해서만 사용할 수 문자열의 일부분을 잘라내거나 문자열을 필터링하거나 일치하는 조건의 문자열을 추출 할 수 없습니다. 이러한 수요를 만족 싶다면 정규 표현식의 복잡한 패턴을 사용해야합니다.

우리가 잘 일종의 스크래핑 프로그램이 필요합니다. 아래에서는 스크래핑를 예로 어떻게 정규 표현식을 사용하여 얻은 데이터에 대해 필터링 또는 잘라내기를 할 바구니 설명합니다 :

package main

import (
"fmt"
"io / ioutil"
"net / http"
"regexp"
"strings"
)

func main () {
resp, err : = http.Get ( "http://www.baidu.com")
if err! = nil {
fmt.Println ( "http get error")
}
defer resp.Body.Close ()
body, err : = ioutil.ReadAll (resp.Body)
if err! = nil {
fmt.Println ( "http read error")
return
}

src : = string (body)

// HTML 태그를 소문자로 변환합니다
re _ : = regexp.Compile ( "\\ <​​\\ S \\ s +? \\>")
src = re.ReplaceAllStringFunc (src, strings.ToLower)

// <style> 태그를 제거합니다
re _ = regexp.Compile ( "\\ <​​style \\ S \\ s +? \\ </ style \\>")
src = re.ReplaceAllString (src, "")

// <script> 태그를 제거
re _ = regexp.Compile ( "\\ <​​script [\\ S \\ s +? \\ </ script \\>")
src = re.ReplaceAllString (src, "")

// <> 내의 모든 HTML 코드를 삭제하고 개행 문자로 바꿉니다
re _ = regexp.Compile ( "\\ <​​\\ S \\ s +? \\>")
src = re.ReplaceAllString (src, "\ n")

// 일련의 분리를 제거합니다
re _ = regexp.Compile ( "\\ s {2}")
src = re.ReplaceAllString (src, "\ n")

fmt.Println (strings.TrimSpace (src))
}

이 예에서 볼 수 있듯이 복잡한 정규 표현식을 사용하는 경우 먼저 Compile합니다. 이것은 정규식이 맞는지 여부를 분석하고 만약 맞다면 Regexp를 반환합니다. 반환 된 Regexp는 임의의 문자열에서 필요한 작업을 수행 할 수 있습니다.

정규 표현식의 분석은 다음의 몇 가지 방법이 있습니다 :

func Compile (expr string) (* Regexp, error)
func CompilePOSIX (expr string) (* Regexp, error)
func MustCompile (str string) * Regexp
func MustCompilePOSIX (str string) * Regexp

CompilePOSIX과 Compile의 차이는 POSIX에는 반드시 POSIX 문법을 사용할 필요가 있다는 것입니다. 이것은 최장 일치 방식을 사용하여 검색을 수행하고 Compile는 단지 최장 일치 방식이 채용되고 있습니다. (예 : [az] {2,4}과 같은 정규 표현식을 "aa09aaa88aaaa"라는 같은 텍스트에 적용 할 때 CompilePOSIX은 aaaa를 반환하며 Compile가 반환 정규식은 aa됩니다) 전에 Must 로 점화 함수는 정규 표현식의 문법을 분석 할 때 만약 패턴이 정확한 문법 않으면 직접 panic이되지 않고 Must 할 수없는 것은 단지 오류를 반환합니다.

어떻게 Regexp을 만들거나 이해했는데,이 struct가 어떤 방법으로 우리의 문자열 조작을 제공하고 있는지 다시 한번 살펴보기로하자. 우선 아래 검색을 할 수있는 함수를 살펴 보겠습니다 :

func (re * Regexp) Find (b [] byte) [] byte
func (re * Regexp) FindAll (b [] byte, n int) [] [] byte
func (re * Regexp) FindAllIndex (b [] byte, n int) [] [] int
func (re * Regexp) FindAllString (s string, n int) [] string
func (re * Regexp) FindAllStringIndex (s string, n int) [] [] int
func (re * Regexp) FindAllStringSubmatch (s string, n int) [] [] string
func (re * Regexp) FindAllStringSubmatchIndex (s string, n int) [] [] int
func (re * Regexp) FindAllSubmatch (b [] byte, n int) [] [] [] byte
func (re * Regexp) FindAllSubmatchIndex (b [] byte, n int) [] [] int
func (re * Regexp) FindIndex (b [] byte) (loc [] int)
func (re * Regexp) FindReaderIndex (r io.RuneReader) (loc [] int)
func (re * Regexp) FindReaderSubmatchIndex (r io.RuneReader) [] int
func (re * Regexp) FindString (s string) string
func (re * Regexp) FindStringIndex (s string) (loc [] int)
func (re * Regexp) FindStringSubmatch (s string) [] string
func (re * Regexp) FindStringSubmatchIndex (s string) [] int
func (re * Regexp) FindSubmatch (b [] byte) [] [] byte
func (re * Regexp) FindSubmatchIndex (b [] byte) [] int

위의 18 개의 함수는 입력 소스 (byte slice, string 및 io.RuneReader)의 차이에 따라 아래의 일부처럼 단순화 할 수 있습니다. 기타는 단지 입력 소스가 다른 것만으로 다른 기능은 기본적으로 동일합니다 :

func (re * Regexp) Find (b [] byte) [] byte
func (re * Regexp) FindAll (b [] byte, n int) [] [] byte
func (re * Regexp) FindAllIndex (b [] byte, n int) [] [] int
func (re * Regexp) FindAllSubmatch (b [] byte, n int) [] [] [] byte
func (re * Regexp) FindAllSubmatchIndex (b [] byte, n int) [] [] int
func (re * Regexp) FindIndex (b [] byte) (loc [] int)
func (re * Regexp) FindSubmatch (b [] byte) [] [] byte
func (re * Regexp) FindSubmatchIndex (b [] byte) [] int

이 함수의 사용에 대해 다음의 예를 살펴 보자

package main

import (
"fmt"
"regexp"
)

func main () {
a : = "I am learning Go language"

re _ : = regexp.Compile ( "[a-z] {2,4}")

// 정규식과 일치하는 첫 번째를 찾아
one : = re.Find ([] byte (a))
fmt.Println ( "Find :"string (one))

// 정규식에 일치하는 모든 slice를 찾아 낸다. n이 0보다 작 았던 경우는 모든 일치하는 문자열을 반환합니다. 그렇지 않으면 지정된 길이가 반환됩니다.
all : = re.FindAll ([] byte (a) -1)
fmt.Println ( "FindAll", all)

// 조건에 일치하는 index의 위치를​​ 찾아 낸다. 시작 위치와 종료 위치.
index : = re.FindIndex ([] byte (a))
fmt.Println ( "FindIndex"index)

// 조건에 일치하는 모든 index의 위치를​​ 찾아 n은 전술
allindex : = re.FindAllIndex ([] byte (a) -1)
fmt.Println ( "FindAllIndex"allindex)

re2 _ : = regexp.Compile ( "am (*) lang (*)")

// Submatch을 찾아 배열을 돌려줍니다. 처음 요소는 일치하는 모든 요소입니다. 두 번째 요소는 처음 ()에서 세 번째는 두 번째 () 중입니다.
// 다음 출력에서​​는 처음 요소는 "am learning Go language"입니다.
// 두 번째 요소는 "learning Go"입니다. 공백을 포함 출력한다는 점에 유의하십시오.
// 세 번째 요소는 "uage"입니다.
submatch : = re2.FindSubmatch ([] byte (a))
fmt.Println ( "FindSubmatch"submatch)
for _ v : = range submatch {
fmt.Println (string (v))
}

// 정의와 위 FindIndex은 동일합니다.
submatchindex : = re2.FindSubmatchIndex ([] byte (a))
fmt.Println (submatchindex)

// FindAllSubmatch 조건에 일치하는 모든 하위 일치 항목을 찾습니다.
submatchall : = re2.FindAllSubmatch ([] byte (a) -1)
fmt.Println (submatchall)

// FindAllSubmatchIndex 모든 하위 일치 항목의 index를 찾습니다.
submatchallindex : = re2.FindAllSubmatchIndex ([] byte (a) -1)
fmt.Println (submatchallindex)
}

지금까지 매치 함수를 소개했습니다. Regexp도 3 개의 함수를 정의하고 있습니다. 이들은 동맹의 외부 함수 기능은 전혀 함께합니다. 사실 외부 함수는이 Regexp의 3 개의 함수를 호출함으로써 실현하고 있습니다.

func (re * Regexp) Match (b [] byte) bool
func (re * Regexp) MatchReader (r io.RuneReader) bool
func (re * Regexp) MatchString (s string) bool

다음 대체 함수가 어떻게 동작하는지 이해하고 갑시다.

func (re * Regexp) ReplaceAll (src, repl [] byte) [] byte
func (re * Regexp) ReplaceAllFunc (src [] byte, repl func ([] byte) [] byte) [] byte
func (re * Regexp) ReplaceAllLiteral (src, repl [] byte) [] byte
func (re * Regexp) ReplaceAllLiteralString (src, repl string) string
func (re * Regexp) ReplaceAllString (src, repl string) string
func (re * Regexp) ReplaceAllStringFunc (src string, repl func (string) string) string

이러한 대체 함수는 위된다고 예에 자세한 응용 사례가 있습니다.

다음 Expand의 설명을 살펴 보자 :

func (re * Regexp) Expand (dst [] byte, template [] byte, src [] byte (match) [] int) [] byte
func (re * Regexp) ExpandString (dst [] byte, template string, src string (match) [] int) [] byte

Expand 도대체 무엇에 사용되는 것일까 요? 아래의 예를 참조하십시오 :

func main () {
src : = [] byte (`
call hello alice
hello bob
call hello eve
`)
pat : = regexp.MustCompile (`(? m) (call) \ s + (? P <cmd> \ w +) \ s + (? P <arg> +) \ s * $`)
res : = [] byte {}
for _ s : = range pat.FindAllSubmatchIndex (src, -1) {
res = pat.Expand (res [] byte ( "$ cmd ( '$ arg') \ n") src, s)
}
fmt.Println (string (res))
}

지금까지 이미 Go 언어의`regexp` 패키지의 모든 것을 소개했습니다. 이에 대한 주요 기능 소개와 예제를 통해 여러분은 Go 언어의 정규 표현식 패키지를 사용하여 기본적인 정규 표현식의 조작이 가능하게 된 것으로 믿고 있습니다.

