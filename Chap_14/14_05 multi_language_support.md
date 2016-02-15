# Multi Language Support

제 10 장에서 국제화 및 현지화 및 go-i18n 라이브러리의 개발에 대해 소개했습니다. 이 절에서는이 라이브러리를 beego 프레임 워크에서 가져 오기, 우리의 프레임 워크에서 국제화와 현지화를 지원합니다.

## i18n의 도입
beego에서 다음과 같이 전역 변수를 설정합니다 :

Translation i18n.IL
Lang string // 언어 패키지의 설정 zh, en
LangPath string // 언어 패키지의 경로를 설정

다국어 기능을 초기화 :

func InitLang () {
beego.Translation : = i18n.NewLocale ()
beego.Translation.LoadPath (beego.LangPath)
beego.Translation.SetLocale (beego.Lang)
}

템플릿에서 직접 다국어 패키지를 호출 할 수 있도록 3 개의 함수에 의해 대응하는 다국어를 설계합니다 :

beegoTplFuncMap [ "Trans"] = i18n.I18nT
beegoTplFuncMap [ "TransDate"] = i18n.I18nTimeDate
beegoTplFuncMap [ "TransMoney"] = i18n.I18nMoney

func I18nT (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
return beego.Translation.Translate (s)
}

func I18nTimeDate (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
return beego.Translation.Time (s)
}

func I18nMoney (args ... interface {}) string {
ok : = false
var s string
if len (args) == 1 {
s, ok = args [0] (string)
}
if! ok {
s = fmt.Sprint (args ...)
}
return beego.Translation.Money (s)
}

## 다국어 개발의 사용
1. 언어 및 언어 패키지의 경로를 설정합니다. 그 i18n 개체를 초기화합니다 :

beego.Lang = "zh"
beego.LangPath = "views / lang"
beego.InitLang ()

2. 다국어 패키지 디자인

위에서는 어떻게 다국어 패키지를 초기화하는 방법에 대해 소개했습니다. 지금부터 다국어 패키지를 설계합니다. 다국어 패키지는 json 파일입니다. 제 10 장에서 소개 한 것과 같이 설계 할 필요가있는 파일을 LangPath 아래에 놓습니다. 예를 들어 zh.json 또는 en.json 같은 것입니다.

# zh.json

{
"zh": {
"submit": "전송"
"create": "새로 만들기"
}
}

# en.json

{
"en": {
"submit": "Submit"
"create" "Create"
}
}

3. 다국어 패키지를 사용하는

controller에서 컴파일러를 호출하여 해​​당 번역 언어를 얻을 수 있습니다. 다음과 같습니다 :

func (this * MainController) Get () {
this.Data [ "create"] = beego.Translation.Translate ( "create")
this.TplNames = "index.tpl"
}

템플릿에서 직접 해당 번역 함수를 호출해도 괜찮습니다 :

// 직접 텍스트 번역
{{.create | Trans}}

// 시간 번역
{{.time | TransDate}}

// 통화 번역
{{.money | TransMoney}}
