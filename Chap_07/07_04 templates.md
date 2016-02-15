# Templates

## 템플릿이란 무엇인가
아마 당신은 MVC 디자인 패턴에 대해 들어 본 적이 있다고 생각합니다. Model은 데이터를 처리를 View는 표시 결과를 Controller는 사용자의 요청을 제어합니다. View 레이어의 처리는 많은 동적 언어는 아무도 정적 HTML 안에 동적 언어가 생성 한 데이터를 삽입합니다. 예를 들어 JSP에서는`<% = .... = %>`을 삽입하여 PHP에서`<? php .....?>`을 삽입하는 것으로 실현합니다.

아래의 그림에서 템플릿 메커니즘을 소개합니다

! [] (images / 7.4.template.png? raw = true)

그림 7.1 템플릿 메커니즘도

Web 응용 프로그램이 클라이언트에 반환 피드백 정보 중 대부분의 내용은 정적 불변입니다. 또한 적은 부분​​에서 사용자의 요청에 의해 동적으로 생성되는 것이 있습니다. 예를 들어 사용자의 액세스 로그 목록을 표시하려면 사용자 사이에서는 로그 데이터가 다른만으로, 목록 스타일은 고정입니다. 이때 템플릿을 이용하여 많은 정적 코드를 사용 돌릴 수 있습니다.

## Go 템플릿 사용
Go 언어에서는`template` 패키지를 사용하여 템플릿을 수행합니다. `Parse``ParseFile``Execute` 같은 방법을 사용하여 파일이나 문자열에서 템플릿을로드합니다. 그 후 심어 그림에 표시된 템플릿 merge 작업 같은 것을 실행합니다. 아래의 예를 참조하십시오 :

func handler (w http.ResponseWriter, r * http.Request) {
t : = template.New ( "some template") // 템플릿을 새로 작성한다.
t _ = t.ParseFiles ( "tmpl / welcome.html", nil) // 템플릿 파일을 분석
user : = GetUser () // 현재 사용자의 정보를 얻을 수 있습니다.
t.Execute (w, user) // 템플릿 merger 작업을 수행한다.
}

위의 예에서 Go 언어의 템플릿 작업은 매우 간단하고 편리하다고 이해해 주실 것으로 생각합니다. 다른 언어의 템플릿 처리와 유사하여 먼저 데이터를 검색 한 후 데이터를 적용합니다.

데모 및 테스트 코드 간편 때문에 이후의 예에서는 다음과 같은 형식의 코드를 채용합니다.

- ParseFiles 대신 Parse를 사용합니다. Parse는 직접 문자열을 테스트 할 외부 파일을 필요로하지 않기 때문입니다.
- handler를 사용해 데모 코드를 작성하지 않고, 각각 하나의 main을 테스트합니다. 편리한 테스트입니다.
-`http.ResponseWriter` 대신`os.Stdout`을 사용합니다. `os.Stdout`는`io.Writer` 인터페이스를 구현하고 있기 때문입니다.

## 어떻게 템플릿에 데이터를 삽입 하는가?
위에서 어떻게 해석과 템플릿 적용하거나 시연했습니다. 이상에서 자세히 어떻게 데이터를 적용 가는지 이해하고 갑시다. 템플릿은 모든 Go 개체에 적용됩니다. Go 객체의 필드는 어떻게 템플릿에 삽입되는 것입니까?

### 필드 작업
Go 언어의 템플릿은`{{}}`을 통해 적용시 대체해야하는 필드를 포함합니다. `{{}}`는 현재 개체를 보여줍니다. 이것은 Java 및 C ++ 안의 this와 비슷한 것입니다. 만약 현재 개체의 필드에 액세스하려면`{{.FieldName}}`라고합니다. 그러나 참고 :이 필드는 반드시 내 보낸 것입니다 (글자가 대문자가됩니다), 그렇지 않으면 적용시 오류를 발생시킵니다. 아래의 예를 참조하십시오 :

package main

import (
"html / template"
"os"
)

type Person struct {
UserName string
}

func main () {
t : = template.New ( "fieldname example")
t _ = t.Parse ( "hello {{.UserName}}!")
p : = Person {UserName : "Astaxie"}
t.Execute (os.Stdout, p)
}

위의 코드는 제대로`hello Astaxie` 출력됩니다. 그러나 만약 코드에 수정을 템플릿에 수출되지 않은 필드를 포함하면 오류를 발생시킵니다.

type Person struct {
UserName string
email string // 내 않은 필드 글자가 소문자입니다.
}

t _ = t.Parse ( "hello {{.UserName}}! {{.email}}")

위의 코드는 오류를 발생시킵니다. 왜냐하면 내되지 않은 필드를 호출 한 것입니다. 그러나 만약 존재하지 않는 필드를 호출하면 오류를 발생시키지 않고 빈 문자열을 출력합니다.

템플릿에서`{{}}`를 출력하면 일반적으로 문자열 개체에 적용됩니다. 기본적으로 fmt 패키지가 호출 된 문자열의 내용이 출력됩니다.

### 중첩 된 필드의 내용 출력
위의 예에서 어떻게 하나의 개체의 필드를 출력하거나 나타냈다. 만약 필드에서 또한 객체가있는 경우는 어떻게 루프하고 이러한 내용을 출력하는 것입니까? 여기에서는`{{with ...}} ... {{end}}`와`{{range ...}} {{end}}`에 의해 데이터를 출력 할 수 있습니다.

- {{range}}은 Go 언어 중 range와 비슷합니다. 루프하고 데이터를 조작합니다
- {{with}} 조작은 현재 개체의 값을 말합니다. 컨텍스트의 개념과 비슷합니다.

자세한 사용 방법은 다음 예를 참조하십시오 :

package main

import (
"html / template"
"os"
)

type Friend struct {
Fname string
}

type Person struct {
UserName string
Emails [] string
Friends [] * Friend
}

func main () {
f1 : = Friend {Fname : "minux.ma"}
f2 : = Friend {Fname : "xushiwei"}
t : = template.New ( "fieldname example")
t _ = t.Parse (`hello {{.UserName}}!
{{range .Emails}}
an email {{}}
{{end}}
{{with .Friends}}
{{range}}
my friend name is {{.Fname}}
{{end}}
{{end}}
`)
p : = Person {UserName : "Astaxie"
Emails : [] string { "astaxie@beego.me", "astaxie@gmail.com"},
Friends : [] * Friend {& f1 & f2}}
t.Execute (os.Stdout, p)
}

### 조건 분기
Go 템플릿에서 만약 의사 결정이 필요할 경우, Go 언어의`if-else` 문장과 유사한 방법을 사용하여 처리 할 수​​ 있습니다. 만약 pipeline이 비어 있으면, if는 기본적으로 false이라고 생각합니다. 아래의 예에서 어떻게`if-else` 문장을 사용하거나 다음과 같습니다 :

package main

import (
"os"
"text / template"
)

func main () {
tEmpty : = template.New ( "template test")
tEmpty = template.Must (tEmpty.Parse ( "하늘의 pipeline if demo : {{if``}} 출력되지 않습니다. {{end}} \ n"))
tEmpty.Execute (os.Stdout, nil)

tWithValue : = template.New ( "template test")
tWithValue = template.Must (tWithValue.Parse ( "하늘이 아닌 pipeline if demo : {{if`anything`}} 내용이 있습니다. 출력합니다. {{end}} \ n"))
tWithValue.Execute (os.Stdout, nil)

tIfElse : = template.New ( "template test")
tIfElse = template.Must (tIfElse.Parse ( "if-else demo : {{if`anything`}} if 부분 {{else}} else 부분 {{end}} \ n"))
tIfElse.Execute (os.Stdout, nil)
}

위의 데모 코드를 통해`if-else` 문장이 상당히 간단하다는 것을 알 수있었습니다. 사용시 매우 쉽게 템플릿 코드에 통합됩니다.

>주의 : if에서 조건 판단을 사용할 수 없습니다. 예를 들어, Mail == "astaxie@gmail.com"같은 판단은 잘못된 것입니다. if 중에서는 bool 값 만 사용할 수 있습니다.

### pipelines
Unix 유저는`pipe` 대해 잘 아시죠. `ls | grep "beego"`같은 문법은 자주 사용되는 것이군요. 현재 디렉토리 아래의 파일을 필터링 "beego"를 포함한 데이터를 표시합니다. 이전 출력 후 입력한다는 의미가 있습니다. 마지막으로 필요한 데이터를 표시합니다. Go 언어의 템플릿의 가장 큰 장점은 데이터 pipe를 지원하는 것입니다. Go 언어에서 어떤`{{}}`중 모든 pipelines 데이터입니다. 예를 들어 위에서 출력 된 email 만일 XSS 인젝션을 일으킬 수 있다고하면 어떻게 변환하는 것일까 요?

{{. | html}}

email이 출력되는 장소에서는 위와 같은 방법으로 출력을 html 엔티티로 변환 할 수 있습니다. 위와 같은 방법은 우리가 평소 쓰고있는 Unix의 방법과 전혀 함께가 잖아요. 아주 간단하게 조작 할 수 있습니다. 다른 함수를 호출하는 경우도 비슷한 방법입니다.

### 템플릿 변수
가끔 템플릿을 사용하고 로컬 변수를 정의 할 수 있습니다. 작업 중 로컬 변수를 선언 할 수 있습니다. 예를 들어`with``range``if` 과정에서 지역 변수를 선언합니다. 이 변수의 범위는`{{end}}`의 전입니다. Go 언어에서 선언 된 로컬 변수의 형식은 다음과 같습니다 :

$ variable : = pipeline

자세한 예제는 다음을 참조하십시오 :

{{with $ x : = "output"| printf "% q"}} {{$ x}} {{end}}
{{with $ x : = "output"}} {{printf "% q"$ x}} {{end}}
{{with $ x : = "output"}} {{$ x | printf "% q"}} {{end}}
### 템플릿 함수
템플릿이 객체의 필드의 값을 출력 할 때`fmt` 패키지를 채용하여 객체를 문자열로 변환합니다. 그러나 때때로 우리는 이렇게하고 싶지 않을 때도 있습니다. 예를 들어 스팸 메일을 보낸 사람이 웹 페이지에서 주워 오는 방법으로 우리의 사서함에 정보를 전송하는 것을 방지하고 싶을 때가 있습니다. `@`를 at로 변환하고자하는 것입니다. 예 :`astaxie at beego.me`처럼. 이런 기능을 구현하려면 스스로 정의한 함수에서이 기능을 작성해야합니다.

각 템플릿 함수는 모두 하나의 이름을 가지고 있고, 하나의 Go 함수와 관련이 있습니다. 다음의 방법으로 관계를 갖게합니다.

type FuncMap map [string] interface {}

예를 들어, 만약 email 함수 템플릿 함수의 이름을`emailDeal`하고 싶은 경우, 이것은 관련 Go 함수의 이름은`EmailDealWith`입니다. 아래의 방법으로이 함수를 등록 할 수 있습니다.

t = t.Funcs (template.FuncMap { "emailDeal": EmailDealWith})

`EmailDealWith`라는 함수의 인수와 반환 값은 다음과 같이 정의합니다 :

func EmailDealWith (args ... interface {}) string

다음의 구현 예를 살펴 보자 :

package main

import (
"fmt"
"html / template"
"os"
"strings"
)

type Friend struct {
Fname string
}

type Person struct {
UserName string
Emails [] string
Friends [] * Friend
}

func EmailDealWith (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
// find the @ symbol
substrs : = strings.Split (s, "@")
if len (substrs)! = 2 {
return s
}
// replace the @ by "at"
return (substrs [0] + "at"+ substrs [1])
}

func main () {
f1 : = Friend {Fname : "minux.ma"}
f2 : = Friend {Fname : "xushiwei"}
t : = template.New ( "fieldname example")
t = t.Funcs (template.FuncMap { "emailDeal": EmailDealWith})
t _ = t.Parse (`hello {{.UserName}}!
{{range .Emails}}
an emails {{. | emailDeal}}
{{end}}
{{with .Friends}}
{{range}}
my friend name is {{.Fname}}
{{end}}
{{end}}
`)
p : = Person {UserName : "Astaxie"
Emails : [] string { "astaxie@beego.me", "astaxie@gmail.com"},
Friends : [] * Friend {& f1 & f2}}
t.Execute (os.Stdout, p)
}


위에서 어떻게 스스로 함수를 정의하거나 보여했습니다. 사실 템플릿 패키지 내부에서는 이미 내장 함수가 구현되어 있습니다. 아래의 코드를 잘라 자신의 템플릿 패키지 안에 붙여주세요.

var builtins = FuncMap {
"and": and,
"call": call,
"html": HTMLEscaper,
"index": index,
"js": JSEscaper,
"len": length,
"not": not,
"or": or,
"print": fmt.Sprint,
"printf": fmt.Sprintf,
"println": fmt.Sprintln,
"urlquery": URLQueryEscaper,
}


## Must 조작
템플릿 패키지에는`Must`라는 함수가 있습니다. 이 작용은 템플릿이 올바른지 검사하는 것입니다. 예를 들어 대괄호가 갖추어져 있는지 코멘트가 제대로 닫혀 있는지 변수가 제대로 적혀 있는지 등입니다. 여기에서는 예를 하나 보여 드리겠습니다. Must를 사용하여 템플릿이 올바른지 확인합니다 :

package main

import (
"fmt"
"text / template"
)

func main () {
tOk : = template.New ( "first")
template.Must (tOk.Parse ( "some static text / * and a comment * /"))
fmt.Println ( "The first one parsed OK")

template.Must (template.New ( "second") Parse ( "some static text {{.Name}}"))
fmt.Println ( "The second one parsed OK")

fmt.Println ( "The next one ought to fail")
tErr : = template.New ( "check parse error with Must")
template.Must (tErr.Parse ( "some static text {{.Name}"))
}

출력은 다음과 같은 내용입니다 :

The first one parsed OK.
The second one parsed OK.
The next one ought to fail.
panic : template : check parse error with Must : 1 : unexpected "}"in command

## 중첩 템플릿
Web 응용 프로그램을 만들 때는 템플릿의 일부가 고정 불변 인 경우가 많으며 뽑아 독립적 인 부분으로 할 수 있습니다. 예를 들어 블로그의 헤더와 내려했지만 고정으로 변경이있는 것은 중간의 콘텐츠 부분 뿐이라고합니다. 따라서`header``content``footer`의 세 부분으로 정의 할 수 있습니다. Go 언어에서는 다음과 같은 문법에 따라이를 선언합니다

{{define "서브 템플릿의 이름"}} 콘텐츠 {{end}}

다음의 방법에 의해 호출합니다 :

{{template "서브 템플릿의 이름"}}

여기에서는 어떻게하고 중첩 된 템플릿을 사용하거나 보여드립니다. 3 개의 파일을 정의합니다. `header.tmpl``content.tmpl``footer.tmpl` 파일입니다. 내용은 다음과 같습니다

//header.tmpl
{{define "header"}}
<html>
<head>
<title> 논증 정보 </ title>
</ head>
<body>
{{end}}

//content.tmpl
{{define "content"}}
{{template "header"}}
<h1> 중첩 데모 </ h1>
<ul>
<li> 중첩은 define을 사용하여 서브 템플릿을 정의합니다. </ li>
<li> template의 사용을 호출 </ li>
</ ul>
{{template "footer"}}
{{end}}

//footer.tmpl
{{define "footer"}}
</ body>
</ html>
{{end}}

데모 코드는 다음과 같습니다 :

package main

import (
"fmt"
"os"
"text / template"
)

func main () {
s1 _ : = template.ParseFiles ( "header.tmpl", "content.tmpl", "footer.tmpl")
s1.ExecuteTemplate (os.Stdout "header", nil)
fmt.Println ()
s1.ExecuteTemplate (os.Stdout "content", nil)
fmt.Println ()
s1.ExecuteTemplate (os.Stdout "footer", nil)
fmt.Println ()
s1.Execute (os.Stdout, nil)
}

위의 예에서`template.ParseFiles`를 사용하여 모든 중첩 된 템플릿을 템플릿에서 퍼스있는 것을 알 주실 수 있었다고 생각합니다. 각 정의 {{define}}은 모든 독립 한 개의 템플릿으로 서로 독립적입니다. 병렬로 존재하는 관계입니다. 내부에서는 map과 같은 관계 (key가 템플릿의 이름이고 value가 템플릿의 내용입니다.)이 저장되어 있습니다. 그 후`ExecuteTemplate`을 사용하여 해당 하위 템플릿의 내용을 실행합니다. header와 footer를 모두 서로 독립적으로있는 것을 알 수 있습니다. 아무도 컨텐츠를 출력 할 수 있습니다. content에서 header와 footer의 내용이 내포하고 있기 때문에 동시에 3 가지 내용을 출력 할 수 있습니다. 그러나`s1.Execute`을 실행했을 때, 아무것도 출력되지 않습니다. 기본적으로 기본 서브 템플릿이 없기 때문입니다. 따라서 아무것도 출력되지 않습니다.

> 단일 집합 같은 템플릿은 서로를 알고 있습니다. 만약 템플릿이 여러 집합에 의해 사용 된 경우 다중 집합에서 별도로 퍼스 될 필요가 있습니다.

## 정리
템플릿에 대한 위의 상세한 소개로 어떻게 동적 데이터와 템플릿을 융합시키는 지 이해하실 수 있을까 생각합니다 : 루프 데이터의 출력 함수를 정의 템플릿의 중첩 등등. 템플릿의 기술을 응용하여 MVC 패턴의 V의 처리를 완성 할 수 있습니다. 다음 장에서는 어떻게 M과 C를 처리하거나 소개합니다.

