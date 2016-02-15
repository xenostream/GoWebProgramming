# Static Files

전에 이미 어떻게 정적 파일을 지원하는지에 대해 설명하고 있습니다 만,이 절에서는 beego에서 어떻게 정적 파일을 설정 및 사용하거나 상세하게 소개합시다. 여기서 또 twitter 오픈 소스 html, css 프레임 워크 bootstrap을 소개합니다. 대량의 설계를 필요로하지 않고 아름다운 홈페이지를 만들 수 있습니다.

## beego 정적 파일의 구현 및 설정
Go의 net / http 패키지는 정적 파일의 서비스를 제공하고 있습니다. `ServeFile`와`FileServer` 같은 함수입니다. beego 정적 파일의 처리는이 계층에서 처리됩니다. 구체적인 구현은 다음과 같습니다 :

// static file server
for prefix, staticDir : = range StaticDir {
if strings.HasPrefix (r.URL.Path, prefix) {
file : = staticDir + r.URL.Path [len (prefix) :]
http.ServeFile (w, r, file)
w.started = true
return
}
}

StaticDir에 저장되어있는 것은 url이 대응하는 정적 파일이있는 디렉토리입니다. 따라서 URL 요청을 처리 할 때 해당 요청 주소에 정적 처리로 시작하는 url을 포함하고 있는지 판단 할뿐입니다. 만약 포함되어 있다면, http.ServeFile에 의해 서비스가 제공됩니다.

예를 들어 보겠습니다 :

beego.StaticDir [ "/ asset"] = "/ static"

는 요청 된 url이`http : // www.beego.me / asset / bootstrap.css` 있었다면 요청`/ static / bootstrap.css` 의해 피드백이 고객에게 제공됩니다.

## bootstrap 세트
Bootstrap은 Twitter가 만들어 낸 오픈 소스 프런트 엔드 개발 도구 패키지입니다. 개발자는 Bootstrap은 빠른 Web 응용 프로그램 개발에 최선의 프런트 엔드 도구 패키지입니다. 이것은 CSS와 HTML의 세트에서 최신 HTML5 표준을 사용하고 있습니다. Web 개발의 현대적인 버전, 폼, 버튼, 테이블, 네트워크 시스템 등을 제공합니다.

- 모듈
Bootstrap은 풍부한 Web 모듈이 포함되어 있습니다. 이러한 모듈에 의해 아름답고 기능이 갖추어 진 페이지를 만들 수 있습니다. 여기에는 다음과 같은 모듈이 포함되어 있습니다 :
풀다운 메뉴, 버튼 세트, 버튼 풀다운 메뉴, 네비게이션, 네비게이션 바, 빵 부스러기 페이징, 순위, 썸네일 오류 대화 진행률 표시 줄 미디어 객체 등
- Javascript 플러그인
Bootstrap은 13 개 jQuery 플러그인을 제공합니다. 이러한 플러그인은 Bootstrap 모듈에 "생명"을 미칩니다. 여기에는 다음이 포함됩니다 :
모드 대화 레이블 페이지 스크롤 막대 팝업 창 등
- 사용자 정의 된 프레임 워크 코드
Bootstrap의 모든 CSS 변수는 수정할 수 있습니다. 자신의 취향에 맞게 코드를 절단 할 수 있습니다.

![](14.1.bootstrap.png)     
그림 14.1 bootstrap 사이트

다음 bootstrap을 beego 프레임 워크에 모으기위한 아름다운 사이트를 만들 수 있습니다.

1. 먼저 다운로드 한 bootstrap 디렉토리를 우리의 프로젝트 디렉토리에 배포합니다. 아래 스크린 샷처럼 이름을 static합니다.

![](14.1.bootstrap2.png)    
그림 14.2 프로젝트에서 정적 파일의 디렉토리 구조

2. beego는 기본적으로 StaticDir 값을 설정하기 때문에, 당신의 정적 디렉토리가 static이면 추가 할 필요가 없습니다 :

StaticDir [ "/ static"] = "static"

3. 템플릿에서 다음과 같은 주소를 사용하면 OK입니다 :

// css 파일
<link href = "/ static / css / bootstrap.css"rel = "stylesheet">

// js 파일
<script src = "/ static / js / bootstrap-transition.js"> </ script>

// 이미지 파일
<img src = "/ static / img / logo.png">

위에서는 bootstrap을 beego 속에 구현하고 있습니다. 아래의 그림은 구현 후 효과도 있습니다 :

! [] (images / 14.1.bootstrap3.png? raw = true)

그림 14.3 bootstrap에 따라 만들어진 사이트의 인터페이스

이러한 템플릿과 포맷은 bootstrap의 공식을 제공하는 것입니다. 여기에 코드를 다시 붙여 다시 수는 없습니다. 여러분은 bootstrap의 공식 사이트에서 어떻게 템플릿을 작성하거나 알아보십시오.

