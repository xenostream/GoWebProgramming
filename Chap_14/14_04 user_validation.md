# User Validation

Web 응용 프로그램을 개발하는 과정에서 사용자 인증은 개발자가 자주 부딪히는 문제입니다. 사용자의 로그인 가입 로그 아웃 등과 같은 조작으로 일반적인 인증은 세 방면의 인증으로 나눌 수 있습니다

- HTTP Basic 및 HTTP Digest 인증
- 타사 인증 : QQ, weibo, doubian, OPENID, google, github, facebook과 twitter 등
- 사용자 정의 사용자 로그인 가입 로그 아웃은 일반적으로 session, cookie 인증을 기반으로합니다.

beego는 현재이 3 가지 방식의 어떤 형식도 지원하지 않습니다. 그러나 타사의 오픈 소스 라이브러리가 위의 세 가지 방법의 사용자 인증을 구현할 수 있습니다. 그러나 추후 beego는 전직 인증을 하나 하나 구현 할지도 모릅니다.

## HTTP Basic 및 HTTP Digest 인증
이 2 가지 인증은 일부 응용 프로그램이 채용하고있는 비교적 간단한 인증입니다. 현재 이미 오픈 소스 타사 라이브러리에서이 두 가지 인증을 지원합니다;

github.com/abbot/go-http-auth

아래의 코드는이 라이브러리를 어떻게 beego에 도입하는 방법을 보여줍니다 :

package controllers

import (
"github.com/abbot/go-http-auth"
"github.com/astaxie/beego"
)

func Secret (user, realm string) string {
if user == "john"{
// password is "hello"
return "$ 1 $ dlPL2MqE $ oQmn16q49SqdmhenQuNgs1"
}
return ""
}

type MainController struct {
beego.Controller
}

func (this * MainController) Prepare () {
a : = auth.NewBasicAuthenticator ( "example.com"Secret)
if username : = a.CheckAuth (this.Ctx.Request); username == ""{
a.RequireAuth (this.Ctx.ResponseWriter, this.Ctx.Request)
}
}

func (this * MainController) Get () {
this.Data [ "Username"] = "astaxie"
this.Data [ "Email"] = "astaxie@gmail.com"
this.TplNames = "index.tpl"
}

위의 코드는 beego의 prepare 함수를 이용하고 있습니다. 정상적인 로직을 실행하기 전에 인증 함수를 호출하여 아주 쉽게 http auth를 구현하고 있습니다. digest 인증도 같은 원리입니다.

## oauth과 oauth2 인증
oauth과 oauth2은 현재 비교적 유행하고있는 두 가지 인증 방식입니다. 타사 다만이 인증을 구현하는 라이브러리가 있는데 해외에서 구현 된 것으로, QQ, weibo 등 중국 내 응용 프로그램 인증은 없습니다.

github.com/bradrydzewski/go.auth

아래의 코드는 어떻게이 라이브러리를 beego에 도입 oauth 인증을 구현하는지 보여줍니다. 여기에서는 github를 예로하고 있습니다 :

1. 라우팅을 2 개 추가

beego.RegisterController ( "/ auth / login", & controllers.GithubController {})
beego.RegisterController ( "/ mainpage", & controllers.PageController {})

2. 다음 GithubController 로그인 화면을 처리 :

package controllers

import (
"github.com/astaxie/beego"
"github.com/bradrydzewski/go.auth"
)

const (
githubClientKey = "a0864ea791ce7e7bd0df"
githubSecretKey = "a0ec09a647a688a64a28f6190b5a0d2705df56ca"
)

type GithubController struct {
beego.Controller
}

func (this * GithubController) Get () {
// set the auth parameters
auth.Config.CookieSec​​ret = [] byte ( "7H9xiimk2QdTdYI7rDddfJeV")
auth.Config.LoginSuccessRedirect = "/ mainpage"
auth.Config.CookieSec​​ure = false

githubHandler : = auth.Github (githubClientKey, githubSecretKey)

githubHandler.ServeHTTP (this.Ctx.ResponseWriter, this.Ctx.Request)
}


3. 로그인에 성공하면 페이지 화면을 처리

package controllers

import (
"github.com/astaxie/beego"
"github.com/bradrydzewski/go.auth"
"net / http"
"net / url"
)

type PageController struct {
beego.Controller
}

func (this * PageController) Get () {
// set the auth parameters
auth.Config.CookieSec​​ret = [] byte ( "7H9xiimk2QdTdYI7rDddfJeV")
auth.Config.LoginSuccessRedirect = "/ mainpage"
auth.Config.CookieSec​​ure = false

user, err : = auth.GetUserCookie (this.Ctx.Request)

// if no active user session then authorize user
if err! = nil || user.Id () == ""{
http.Redirect (this.Ctx.ResponseWriter, this.Ctx.Request, auth.Config.LoginRedirect, http.StatusSeeOther)
return
}

// else, add the user to the URL and continue
this.Ctx.Request.URL.User = url.User (user.Id ())
this.Data [ "pic"] = user.Picture ()
this.Data [ "id"] = user.Id ()
this.Data [ "name"] = user.Name ()
this.TplNames = "home.tpl"
}

전체의 흐름은 다음과 같습니다. 우선 브라우저를 열고 주소를 입력합니다 :

! [] (images / 14.4.github.png? raw = true)

그림 14.4 로그인 버튼을 가지는 톱 페이지의 표시

다음 링크를 클릭하면 다음과 같은 인터페이스가 나타납니다 :

! [] (images / 14.4.github2.png? raw = true)

그림 14.5 로그인 버튼을 클릭하여 github의 권한 취득 페이지를 표시

Authorize app을 클릭하면 다음과 같은 인터페이스가 나타납니다 :

! [] (images / 14.4.github3.png? raw = true)

그림 14.6 권한 취득에 로그인 한 후 표시되는받은 github 정보 페이지

## 사용자 정의 인증
사용자 정의 인증은 일반적으로 session과 함께 확인됩니다. 다음의 코드는있는 beego 오픈 소스 블로그를 기반으로합니다 :


// 로그인 처리
func (this * LoginController) Post () {
this.TplNames = "login.tpl"
this.Ctx.Request.ParseForm ()
username : = this.Ctx.Request.Form.Get ( "username")
password : = this.Ctx.Request.Form.Get ( "password")
md5Password : = md5.New ()
io.WriteString (md5Password, password)
buffer : = bytes.NewBuffer (nil)
fmt.Fprintf (buffer "% x", md5Password.Sum (nil))
newPass : = buffer.String ()

now : = time.Now () Format ( "2006-01-02 15:04:05")

userInfo : = models.GetUserInfo (username)
if userInfo.Password == newPass {
var users models.User
users.Last_logintime = now
models.UpdateUserInfo (users)

// 로그인 성공 session을 설정
sess : = globalSessions.SessionStart (this.Ctx.ResponseWriter, this.Ctx.Request)
sess.Set ( "uid"userInfo.Id)
sess.Set ( "uname"userInfo.Username)

this.Ctx.Redirect (302 "/")
}
}

// 가입 처리
func (this * RegController) Post () {
this.TplNames = "reg.tpl"
this.Ctx.Request.ParseForm ()
username : = this.Ctx.Request.Form.Get ( "username")
password : = this.Ctx.Request.Form.Get ( "password")
usererr : = checkUsername (username)
fmt.Println (usererr)
if usererr == false {
this.Data [ "UsernameErr"] = "Username error, Please to again"
return
}

passerr : = checkPassword (password)
if passerr == false {
this.Data [ "PasswordErr"] = "Password error, Please to again"
return
}

md5Password : = md5.New ()
io.WriteString (md5Password, password)
buffer : = bytes.NewBuffer (nil)
fmt.Fprintf (buffer "% x", md5Password.Sum (nil))
newPass : = buffer.String ()

now : = time.Now () Format ( "2006-01-02 15:04:05")

userInfo : = models.GetUserInfo (username)

if userInfo.Username == ""{
var users models.User
users.Username = username
users.Password = newPass
users.Created = now
users.Last_logintime = now
models.AddUser (users)

// 로그인 성공 session을 설정
sess : = globalSessions.SessionStart (this.Ctx.ResponseWriter, this.Ctx.Request)
sess.Set ( "uid"userInfo.Id)
sess.Set ( "uname"userInfo.Username)
this.Ctx.Redirect (302 "/")
} else {
this.Data [ "UsernameErr"] = "User already exists"
}

}

func checkPassword (password string) (b bool) {
if ok _ : = regexp.MatchString ( "^ [a-zA-Z0-9] {4,16} $", password);! ok {
return false
}
return true
}

func checkUsername (username string) (b bool) {
if ok _ : = regexp.MatchString ( "^ [a-zA-Z0-9] {4,16} $"username);! ok {
return false
}
return true
}

사용자의 로그인과 가입이 있고, 다른 모듈에서도 다음과 같이 사용자가 로그인하고 있는지 여부의 판단을 추가 할 수 있습니다 :

func (this * AddBlogController) Prepare () {
sess : = globalSessions.SessionStart (this.Ctx.ResponseWriter, this.Ctx.Request)
sess_uid : = sess.Get ( "userid")
sess_username : = sess.Get ( "username")
if sess_uid == nil {
this.Ctx.Redirect (302 "/ admin / login")
return
}
this.Data [ "Username"] = sess_username
}
