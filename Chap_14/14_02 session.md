# Session

제 6 장에서는 Go 언어에서 어떻게 session을 사용하는지 소개했습니다. 또한 sessionManger을 구현했습니다. beego 프레임 워크는 sessionManager에 따라 편리하게 처리 기능을 구현합니다.

## session의 구현
beego에서는 주로 다음과 같은 전역 변수 session 처리를 제어합니다.

// related to session
SessionOn bool // session 모듈이 시작되어 있는가? 기본적으로 시작되지 않습니다.
SessionProvider string // session 백엔드에서 처리 모듈을 제공합니다. 기본값은 sessionManager가 지원하는 memory입니다.
SessionName string // 클라이언트에 저장되는 cookies의 이름
SessionGCMaxLifetime int64 // cookies의 유효 기간

GlobalSessions * session.Manager // 글로벌 session 컨트롤러

당연히 위의 몇 가지 변수는 값을 초기화해야 다음 코드에 의해 설정 파일과 함께 이러한 값을 설정할 수 있습니다.

if ar, err : = AppConfig.Bool ( "sessionon"); err! = nil {
SessionOn = false
} else {
SessionOn = ar
}
if ar : = AppConfig.String ( "sessionprovider"); ar == ""{
SessionProvider = "memory"
} else {
SessionProvider = ar
}
if ar : = AppConfig.String ( "sessionname"); ar == ""{
SessionName = "beegosessionID"
} else {
SessionName = ar
}
if ar, err : = AppConfig.Int ( "sessiongcmaxlifetime"); err! = nil && ar! = 0 {
int64val _ : = strconv.ParseInt (strconv.Itoa (ar), 10, 64)
SessionGCMaxLifetime = int64val
} else {
SessionGCMaxLifetime = 3600
}

beego.Run 함수는 다음과 같은 코드가 추가되어 있습니다 :

if SessionOn {
GlobalSessions _ = session.NewManager (SessionProvider, SessionName, SessionGCMaxLifetime)
go GlobalSessions.GC ()
}

SessionOn 설정을 true로하면, 기본적으로 session 기능을 시작합니다. 독립적으로 goroutine을 시작하여 session을 처리합니다.

사용자 설정의 Controller에서 빠르게 session을 사용하기 때문에 저자는`beego.Controller`에서 다음과 같은 방법을 제공합니다 :

func (c * Controller) StartSession () (sess session.Session) {
sess = GlobalSessions.SessionStart (c.Ctx.ResponseWriter, c.Ctx.Request)
return
}

## session 사용
위의 코드는 beego 프레임 워크는 쉽게 session 기능을 상속 할 수 있다고 볼 수 있습니다. 에서는 프로젝트에서 어떻게 사용하는 것일까 요?

먼저 어플리케이션의 main에서 session을 시작합니다 :

beego.SessionOn = true


그 다음에 컨트롤러의 해당 메소드에서 다음과 같이 session을 사용합니다 :

func (this * MainController) Get () {
var intcount int
sess : = this.StartSession ()
count : = sess.Get ( "count")
if count == nil {
intcount = 0
} else {
intcount = count (int)
}
intcount = intcount + 1
sess.Set ( "count"intcount)
this.Data [ "Username"] = "astaxie"
this.Data [ "Email"] = "astaxie@gmail.com"
this.Data [ "Count"] = intcount
this.TplNames = "index.tpl"
}

위의 코드는 어떻게 컨트롤 로직에서 session을 사용하거나 보여줍니다. 주로 2 단계로 나눌 수 있습니다 :

1. session 객체를 얻을

// 객체를 가져 PHP에서 session_start ()와 비슷합니다.
sess : = this.StartSession ()

2. session을 사용하여 일반적인 session 값을 조작합니다

// session 값을 가져옵니다. PHP의 $ _SESSION [ "count"}과 비슷합니다.
sess.Get ( "count")

// session 값을 설정합니다
sess.Set ( "count"intcount)

위의 코드에서 beego 프레임 워크를 개발하는 응용 프로그램에서 사용하는 session은 꽤 유용하다고 볼 수 있습니다. 기본적으로 PHP에서 호출하는`session_start ()`와 비슷합니다.


