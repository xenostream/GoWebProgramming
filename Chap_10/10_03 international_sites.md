# International Sites

앞 절에서 어떻게 지역화 리소스를 처리하거나 소개했습니다. Locale에 대응 한 설정 파일입니다. 그럼 만약 여러 지역화 리소스를 처리하는 경우는? 일부는 우리가 일반적으로 사용하는 예는 : 간단한 텍스트 번역, 시간과 날짜, 숫자 같은 것은 어떻게 처리 할 것인가? 이 절에서는 하나 하나 이러한 문제를 해결하고 있습니다.
## 여러 로켈 패키지 관리
응용 프로그램을 하나 개발할 때 먼저 결정해야 할 것은 하나의 언어 만 지원하면 좋을지 아니면 다국어를 지원 하는가하는 것입니다. 만약 여러 언어 지원 할 필요가있는 경우 조직 구성을 작성하여 향후 더 많은 언어를 추가 할 수 있도록해야합니다. 여기에서는 다음과 같이 설계합니다 : Locale에 관련된 파일을 config / locales 아래에 배치하고 만약 일본어와 영어를 지원해야하는 경우는이 디렉토리 아래에 en.json과 ja .json을 배치해야합니다. 대략적인 내용은 다음과 같다 :

# ja.json

{
"ja": {
"submit": "전송"
"create": "작성"
}
}

# en.json

{
"en": {
"submit": "Submit"
"create" "Create"
}
}

국제화를 지원하는데있어서 여기에서는 국제화와 관련된 패키지를 사용하기로합니다 - - [go-i18n (https://github.com/astaxie/go-i18n)입니다. 우선 go-i18n 패키지에 config / locales 디렉토리를 등록하여 모든 locale 파일을로드합니다.

Tr : = i18n.NewLocale ()
Tr.LoadPath ( "config / locales")

이 패키지는 매우 사용하기 쉽고, 다음의 방법으로 시도 할 수 있습니다 :

fmt.Println (Tr.Translate ( "submit"))
// Submit을 출력
Tr.SetLocale ( "ja")
fmt.Println (Tr.Translate ( "submit"))
// "전송"을 출력

## 자동으로 로켈 패키지를로드
위에서는 어떻게 자동으로 사용자 언어 패키지를로드하거나 소개했습니다. 사실 go-i18n 라이브러리는 이미 많은 기본 포맷 정보를로드하고 있습니다. 예를 들어 시간 형식, 통화 형식입니다. 사용자 지정 설정을 할 경우 이러한 기본 설정을 수정할 수 있습니다. 다음 처리 과정을 참조하십시오 :


// 기본 설정 파일을로드합니다. 이 파일은 모든 go-i18n / locales 아래에 있습니다.

// 파일은 각각 zh.json en-json en-US.json 등과 이름을 붙입니다. 더 많은 언어를 계속 확장 할 수 있습니다.

func (il * IL) loadDefaultTranslations (dirPath string) error {
dir 또는 err : = os.Open (dirPath)
if err! = nil {
return err
}
defer dir.Close ()

names, err : = dir.Readdirnames (-1)
if err! = nil {
return err
}

for _ name : = range names {
fullPath : = path.Join (dirPath, name)

fi, err : = os.Stat (fullPath)
if err! = nil {
return err
}

if fi.IsDir () {
if err : = il.loadTranslations (fullPath); err! = nil {
return err
}
} else if locale : = il.matchingLocaleFromFileName (name); locale! = ""{
file, err : = os.Open (fullPath)
if err! = nil {
return err
}
defer file.Close ()

if err : = il.loadTranslation (file, locale); err! = nil {
return err
}
}
}

return nil
}

위의 방법으로 설정 파일을 기본 파일에로드합니다. 이렇게 사용자 정의 시간 정보가없는 경우에도 다음과 같은 코드에서 해당 정보를 얻을 수 있습니다 :

// locale = zh 상황에서 다음 코드를 실행 :

fmt.Println (Tr.Time (time.Now ()))
// 출력 : 2009 년 1 월 08 일 별 기 사 20:37:58 CST

fmt.Println (Tr.Time (time.Now () "long"))
// 출력 : 2009 년 1 월 08 일

fmt.Println (Tr.Money (11.11))
// 출력 : ¥ 11.11

## template mapfunc
위에서는 여러 언어 패키지 관리 및로드를 구현했습니다. 일부 함수의 구현 로직 계층을 기반으로합니다. 예를 들면 : "Tr.Translate", "Tr.Time", "Tr.Money"등입니다. 로직 계층에서는 이러한 함수를 이용하여 필요한 매개 변수에 대해 대체 한 후 템플릿 레이어에 적용 할 때 직접 출력 할 수 있습니다. 그러나 만약 템플릿 레이어에서이 함수를 직접 사용하려면 어떻게 실현하는 것입니까? 기억 계십니까 ​​전에 템플릿을 소개했을 때 : Go 언어의 템플릿은 사용자 템플릿 함수를 지원합니다. 이하에서는 작업에 편리한 mapfunc을 구현합니다 :

1. 텍스트 정보

텍스트 정보는`Tr.Translate`를 호출하여 해​​당 데이터를 대체합니다. mapFunc의 구현은 다음과 같습니다 :

func I18nT (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
return Tr.Translate (s)
}

함수의 등록은 다음과 같습니다 :

t.Funcs (template.FuncMap { "T": I18nT})

템플릿의 사용은 다음과 같습니다 :

{{.V.Submit | T}}


2. 시간과 날짜

시간과 날짜는`Tr.Time` 함수를 호출하여 해​​당 시간을 대체합니다. mapFunc의 구현은 다음과 같습니다 :

func I18nTimeDate (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
return Tr.Time (s)
}

함수의 등록은 다음과 같습니다 :

t.Funcs (template.FuncMap { "TD": I18nTimeDate})

템플릿의 사용은 다음과 같습니다 :

{{.V.Now | TD}}

3. 통화 정보

통화`Tr.Money` 함수를 호출하여 해​​당 시간을 대체합니다. mapFunc의 구현은 다음과 같습니다 :

func I18nMoney (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
return Tr.Money (s)
}

함수의 등록은 다음과 같습니다 :

t.Funcs (template.FuncMap { "M": I18nMoney})

템플릿의 사용은 다음과 같습니다 :

{{.V.Money | M}}

## 정리
이 절을 통해 다국어 패키지의 Web 응용 프로그램을 어떻게 실현할지 알 수있었습니다. 사용자 언어 패키지로 편리하게 다국어를 구현할 수 있습니다. 또한 설정 파일에 의해 아주 쉽게 여러 언어를 확장 할 수 있습니다. 기본적으로 go-i18n는 공용 설정 파일을로드합니다. 예를 들어 시간, 통화 등입니다. 아주 쉽게 사용할 수 있으며, 동시에 템플릿에서 이러한 기능을 지원하기 위해 해당 템플릿 함수도 구현했습니다. 이렇게하여 Web 응용 프로그램을 개발할 때 직접 템플릿에서 pipeline 방식으로 다국어 패키지를 조작 할 수 있습니다.

