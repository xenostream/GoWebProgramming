# Localized Resources

이전 절에서는 어떻게하고 Locale를 설정하거나 소개했습니다. Locale를 설정 한 후에는 어떻게하고 Locale에 대응하는 정보를 저장 하는가하는 문제를 해결해야합니다. 여기에서 정보는 다음의 내용을 포함합니다 : 텍스트 정보, 시간과 날짜, 통화 가치, 이미지 파일이나 동영상 등 자원 등입니다. 여기에서는 이러한 정보에 대해 소개하고 싶다고 생각합니다. Go 언어에서는 이러한 형식의 정보를 JSON으로 저장합니다. 그 각각 적합한 방법으로 표시합니다. (이하에서는 일본어와 영어 두 언어를 비교하여 예를 듭니다. 저장 형식은 각각 en.json과 ja-JP.json입니다.)
## 현지화 된 텍스트 정보
이 정보는 Web 응용 프로그램을 작성 중 가장 많이 사용되는 것으로, 지역화 리소스에서도 가장 많은 정보도 있습니다. 로케일의 언어에 맞는 방법으로 텍스트 정보를 표시 할 경우 하나의 방법으로 필요한 언어에 대응 한 map을 작성하여 key-value의 관계를 유지하는 것입니다. 출력되기 전에 최적 map에서 해당 텍스트를 꺼냅니다. 다음은 간단한 예입니다 :

package main

import "fmt"

var locales map [string] map [string] string

func main () {
locales = make (map [string] map [string] string 2)
en : = make (map [string] string 10)
en [ "pea"] = "pea"
en [ "bean"] = "bean"
locales [ "en"] = en
cn : = make (map [string] string 10)
cn [ "pea"] = "땅콩"
cn [ "bean"] = "완두콩"
locales [ "ja-JP"] = cn
lang : = "ja-JP"
fmt.Println (msg (lang "pea"))
fmt.Println (msg (lang "bean"))
}

func msg (locale, key string) string {
if v, ok : = locales [locale]; ok {
if v2, ok : = v [key]; ok {
return v2
}
}
return ""
}


위의 예에서 다른 locale 텍스트의 번역을 시도했습니다. 일본어와 영어에 대해 동일한 key로 다른 언어의 구현을 실현하고 있습니다. 위의 문장의 텍스트 정보를 구현하고 있습니다. 만약 영어 버전으로 전환하려면 lang 설정을 en으로하면됩니다.

경우에 따라서는 key-value를 바꾸는 것만으로는 요구를 만족하지 못할 수 있습니다. 예를 들어 "I am 30 years old"라는 같은 일본어로는 "올해로 30입니다"가되는 경우 여기서 30은 변수입니다. 어떻게하면 좋을까요? 이때`fmt.Printf` 함수를 결합하여 구현할 수 있습니다. 아래의 코드를 참조하십시오 :

en [ "how old"] = "I am % d years old"
cn [ "how old"] = "올해 % d입니다"

fmt.Printf (msg (lang "how old") 30)

위의 코드 예제에서는 내부의 구현 방법을 사용했을뿐, 실제 데이터는 JSON에 저장되어 있습니다. 따라서`json.Unmarshal`를 사용하여 해당 map에 데이터를 추가 할 수 있습니다.

## 날짜와 시간의 현지화
시간대의 관계에서 동일한 시간에 있어서도 다른 지역에서 그 표시는 달라집니다. 또한 Locale의 관계에서 시간의 형식도 전혀 달라집니다. 예를 들어 일본어 환경 하에서는 :`2013 년 10 월 24 일 수요일 23시 11 분 13 초 JST`되고, 영어 환경에서는`Wed Oct 24 23:11:13 CST 2012`과 같이 표시됩니다. 여기에서는 두 항목을 해결해야합니다.

1. 시간대 문제
2. 형식의 문제

$ GOROOT / lib / time 패키지 timeinfo.zip에는 locale에 대응하는 시간대 정의가 포함되어 있습니다. 대응하는 현재의 locale의 시간을 얻기 위해 먼저`time.LoadLocation (name string)`를 사용하여 해당 시간대의 locale을 가져옵니다. 예를 들어`Asia / Shanghai` 또는`America / Chicago`에 대응하는 시간대 데이터입니다. 그 후,이 정보를 재사용하고`time.Now`를 호출함으로써 얻을 수있는 Time 객체와 함​​께 마지막 시간을 가져옵니다. 자세한 내용은 아래의 예를 참조하십시오 (이 예에서는 위의 몇 가지 변수를 채택하고 있습니다) :

en [ "time_zon​​e"] = "America / Chicago"
cn [ "time_zon​​e"] = "Asia / Tokyo"

loc _ : = time.LoadLocation (msg (lang "time_zon​​e"))
t : = time.Now ()
t = t.In (loc)
fmt.Println (t.Format (time.RFC3339))

본문 형식을 처리하는 비슷한 방법으로 시간 형식의 문제를 해결 할 수 있습니다. 예를 들면 :

en [ "date_format"] = "% Y- % m- % d % H : % M : % S"
cn [ "date_format"] = "% Y 년 % m 월 % d 일 % H시 % M 분 % S 초"

fmt.Println (date (msg (lang "date_format"), t))

func date (fomate string, t time.Time) string {
year, month, day = t.Date ()
hour, min, sec = t.Clock ()
// 해당 % Y % m % d % H % M % S를 분석해서 정보를 반환합니다
// % Y는 2012로 대체됩니다
// % m은 10로 대체됩니다
// % d는 24로 대체됩니다
}

## 통과 값의 지역화
각 지역의 통과의 표시도 다릅니다. 처리 방법도 일시와별로 다르지 않습니다. 자세한 내용은 다음의 코드를 참조하십시오 :

en [ "money"] = "USD % d"
cn [ "money"] = "¥ % d 원형"

fmt.Println (date (msg (lang "date_format"), 100))

func money_format (fomate string, money int64) string {
return fmt.Sprintf (fomate, money)
}


## 뷰 및 리소스 지역화
Locale의 차이에 따라 뷰를 표시 할 수 있을지도 모릅니다. 이러한 뷰는 이미지, css, js 등 각종 정적 리소스가 포함되어 있습니다. 은 이러한 정보를 어떻게 처리해야할까요? 우선 locale에 따라 파일 정보를 구성해야합니다. 아래의 디렉토리 구성을 참조하십시오 :

views
| --en // 영어 템플릿
| --images // 이미지 정보를 저장
| --js // JS 파일 저장
| --css // css 파일을 저장
index.tpl // 사용자의 톱 페이지
login.tpl // 로그인 톱 페이지
| --ja-JP // 일본어 템플릿
| --images
| --js
| --css
index.tpl
login.tpl

이 디렉토리 구성이 비로소 다음과 같은 코드로 구현할 수 있습니다 :


s1 _ : = template.ParseFiles ( "views"+ lang + "index.tpl")
VV.Lang = lang
s1.Execute (os.Stdout, VV)

또한 index.tpl 중 자원의 설정은 다음과 같습니다 :

// js 파일
<script type = "text / javascript"src = "views / {{. VV.Lang}} / js / jquery / jquery-1.8.0.min.js"> </ script>
// css 파일
<link href = "views / {{. VV.Lang}} / css / bootstrap-responsive.min.css"rel = "stylesheet">
// 이미지 파일
<img src = "views / {{. VV.Lang}} / images / btn.png">

이러한 방법을 채용하여보기와 자원을 집중하면 준비에 확장을 할 수 있습니다.

## 정리
이 절에서는 어떻게 지역화 리소스를 사용하여 저장하거나 소개했습니다. 어떤 때는 대체 함수로 구현해야, 또 어떤 때는 lang 의해 설정해야합니다. 그러나 결국 아무도 key-value의 방법에 의해 Locale에 대응 한 데이터를 저장할 수 있습니다. 필요할 때 대응하는 Locale 정보를 제거하고 만약 그것이 밖으로 k 시우와 정보라면 직접 출력하고 만약 시간과 날짜 또는 통과 인 경우는`fmtPrintf`를 사용하거나 다른 포맷 함수에 의해 처리 해야합니다. 다른 Locale의 뷰와 리소스에 대해 가장 간단하고 경로에 lang을 추가하는 것만으로 구현 할 수 있습니다.

