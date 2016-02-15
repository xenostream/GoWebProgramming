# How to use Session in Go

앞 절에서 session은 서버 측에서 구현되는 사용자와 서버 간의 인증 솔루션의 하나임을 소개했습니다. 현재 Go 표준 패키지는 session의 지원이 없습니다. 이 절에서는 실제로 손을 움직여 go 버전의 session 관리 및 제작을 실현 해 봅니다.

## session의 생성 과정
session의 기본 원리는 서버가 각 세션에서 데이터를 보호하는 것입니다. 클라이언트 사이드는 서버 측과 글로벌 고유 ID 하나를 의지하고이 데이터에 액세스하고 대화식 목적이 달성됩니다. 사용자가 Web 응용 프로그램에 액세스 할 때 서버 측 프로그램은 session 작성의 요구에 따릅니다. 이 과정은 3 단계로 나눌 수 있습니다 :

- 글로벌 고유 ID 생성 (sessionid)
- 데이터의 저장 공간을 만들었습니다. 보통은 메모리에 대응하는 데이터 구조를 만듭니다. 그러나 이러한 상황에서는 시스템은 일단 끄면 모든 세션 데이터가 손실됩니다. 만약 e 커머스 같은 홈페이지 인 경우, 이것은 심각한 결과를 초래합니다. 따라서 이러한 문제를 해결하기 위해 세션 데이터를 파일 안이나 데이터베이스에 쓸 수 있습니다. 당연히이 경우 I / O 오버 헤드가 증가하지만, 어느 정도의 session 지속성 실현 될 수 있으며, session 공유에도 유리합니다.
- session의 글로벌 고유 ID를 클라이언트 측에 보냅니다.

위의 3 단계에서 가장 중요한 것은 어떻게이 session의 고유 ID를 전달할 것인지에 단계입니다. HTTP 프로토콜의 정의에 따라 데이터 요청 행 헤더 부 또는 Body에 포함 밖에 없습니다. 따라서 일반적으로 두 가지 일반적으로 사용되는 방법이 있습니다 : cookie 및 URL 재 작성합니다.

1. Cookie
서버 사이드 Set-cookie 헤더를 설정하여 session의 ID를 클라이언트 측에 보낼 수 있습니다. 클라이언트 사이드는 이후의 각 요청이 들어이 ID를 포함합니다. 또한 session 정보를 포함한 cookie 유효 기간을 0 (세션 cookie), 즉 브라우저 프로세스의 유효 기간으로 설정하는 일도 자주 발생합니다. 각 브라우저는 서로 다른 구현이되어 있습니다 만, 차이는 그리 크지 않습니다 (일반적으로 브라우저 창을 새로 만들 때 반영됩니다).
2. URL 재 작성
이른바 URL 재 작성은 사용자에게 반환되는 페이지 안의 모든 URL의 뒤에 sessionID를 추가하는 것입니다. 이와 같이 사용자가 응답을받은 후 응답 페이지에서 어떤 링크를 클릭하거나 양식을 제출하여도 자동으로 sessionID가 부여됩니다. 이로 인해 세션의 유지를 실현합니다. 이러한 방법은 조금 귀찮기는하지만, 만약 클라이언트 측이 cookie를 금지하는 경우 이러한 솔루션이 먼저 선택됩니다.

## Go에서 session 관리를 실현
위 session 생성 과정의 설명에서 독자는 session의 대부분의 지식을 얻은 것으로 생각합니다. 그러나 구체적인 동적 페이지 기술에서는 또 어떻게 session을 실현하고있는 것입니까? 여기에서는 session의 라이프 사이클 (lifecycle)와 함께 go 언어 버전의 session 관리를 실현합니다.

### session 관리 설계
session 관리는 다음의 몇 가지 요소가 관여합니다

- 글로벌 session 관리자
- sessionid가 전역 적으로 고유임을 보증
- 각 사용자를 하나의 session에 연관
- session의 저장 (메모리, 파일, 데이터베이스 등에 저장할 수 있습니다)
- session의 만료 처리

이상에서 session 관리의 종합 설계 구상과 대응하는 go 코드 예제에 대해 설명합니다 :

### Session 관리자

있는 글로벌 session 관리자를 정의합니다

type Manager struct {
cookieName string // private cookiename
lock sync.Mutex // protects s​​ession
provider Provider
maxlifetime int64
}

func NewManager (provideName, cookieName string, maxlifetime int64) (* Manager, error) {
        provider, ok : = provides [provideName]
if! ok {
return nil, fmt.Errorf ( "session : unknown provide % q (forgotten import?)"provideName)
}
return & Manager {provider : provider, cookieName : cookieName, maxlifetime : maxlifetime} nil
}

Go에서 제공되는 전체의 흐름은 대체로 이런 것들입니다. main 패키지에서 글로벌 session 관리자를 만듭니다.

var globalSessions * session.Manager
//이 후 init 함수에서 초기화됩니다.
func init () {
globalSessions _ = NewManager ( "memory", "gosessionid", 3600)
}

우리는 session이 서버 측에 저장되는 데이터임을 알고 있습니다. 이것은 어떤 방식으로 저장 되어도 괜찮습니다. 예를 들어 메모리 데이터베이스 또는 파일에 저장합니다. 따라서 Provider 인터페이스를 추상화하여 토큰 session 관리자가 낮은 계층 구조를 저장합니다.

type Provider interface {
SessionInit (sid string) (Session, error)
SessionRead (sid string) (Session, error)
SessionDestroy (sid string) error
SessionGC (maxLifeTime int64)
}

- SessionInit 함수는 Session의 초기화를 구현합니다. 작업에 성공하면이 새로운 Session 변수를 반환합니다.
- SessionRead 함수는 sid가 나타내는 Session 변수를 반환합니다. 만약 존재하지 않으면 sid를 인수로 SessionInit 함수를 호출하고 새로운 Session 변수를 새로 생성하여 반환합니다.
- SessionDestroy 함수는 sid에 대응하는 Session 변수를 처리하는 데 사용됩니다.
- SessionGC는 maxLifeTime 따라 만료 된 데이터를 삭제합니다.

에서는 Session 인터페이스는 어떤 기능을 구현해야한다 있을까요? Web 개발의 경험이있는 독자라면 아시 고는 생각 합니다만, Session에 대한 처리의 기본 값을 설정하는 값을 취득하는 값을 삭제하는 현재 sessionID를 취득하는 4 가지 작업입니다 있습니다. 그래서 우리의 Session 인터페이스도이 네 가지 작업을 구현합니다.

type Session interface {
Set (key, value interface {}) error // set session value
Get (key interface {}) interface {} // get session value
Delete (key interface {}) error // delete session value
SessionID () string // back current sessionID
}

> 이상의 설계 구상은 database / sql / driver에서 유래합니다. 먼저 인터페이스를 정의하고, 그 후 실제로 session을 저장하는 구조가 해당 인터페이스를 구현 등록하면 해당 기능을 사용할 수 있습니다. 다음은 주문형으로 등록 session 구조를 저장하는 Register 함수의 구현입니다.

var provides = make (map [string] Provider)

// Register makes a session provide available by the provided name.
// If Register is called twice with the same name or if driver is nil,
// it panics.
func Register (name string, provider Provider) {
if provider == nil {
panic ( "session : Register provide is nil")
}
if _ dup : = provides [name]; dup {
panic ( "session : Register called twice for provide"+ name)
}
provides [name] = provider
}

### 글로벌 독특한 Session ID

Session ID는 Web 응용 프로그램에 액세스하는 각 사용자를 식별하는 데 사용됩니다. 그러므로 이것은 글로벌 고유임을 보장해야합니다. (GUID) 아래의 코드는 어떻게이 요구를 만족시키는 지 보여줍니다.

func (manager * Manager) sessionId () string {
b : = make ([] byte 32)
if _ err : = io.ReadFull (rand.Reader b); err! = nil {
return ""
}
return base64.URLEncoding.EncodeToString (b)
}

### session 만들기
각 사용자에 대해 그들과 결부 Session을 주거나 취득하여 Session 정보에 따라 작업을 확인해야합니다. SessionStart라는 함수의 Session이 현재 사용하고있는 사용자와 이미 관련되어 있는지 검사하기 위해 사용됩니다. 만약 없으면 새로 이것을 만듭니다.

func (manager * Manager) SessionStart (w http.ResponseWriter, r * http.Request) (session Session) {
manager.lock.Lock ()
defer manager.lock.Unlock ()
cookie, err : = r.Cookie (manager.cookieName)
if err! = nil || cookie.Value == ""{
sid : = manager.sessionId ()
session _ = manager.provider.SessionInit (sid)
cookie : = http.Cookie {Name : manager.cookieName, Value : url.QueryEscape (sid), Path : "/"HttpOnly : true, MaxAge : int (manager.maxlifetime)}
http.SetCookie (w & cookie)
} else {
sid _ : = url.QueryUnescape (cookie.Value)
session _ = manager.provider.SessionRead (sid)
}
return
}

이전의 login 조작으로 나타낸 session의 운용을 이용합니다 :

func login (w http.ResponseWriter, r * http.Request) {
sess : = globalSessions.SessionStart (w, r)
r.ParseForm ()
if r.Method == "GET"{
t _ : = template.ParseFiles ( "login.gtpl")
w.Header () Set ( "Content-Type", "text / html")
t.Execute (w, sess.Get ( "username"))
} else {
sess.Set ( "username", r.Form [ "username"])
http.Redirect (w, r, "/", 302)
}
}

### 치의 조작 : 설정로드 및 삭제
SessionStart 함수는 Session 인터페이스를 만족시킬 변수를 반환합니다. 그럼 어떻게 이것을 이용하여 session 데이터에 대해 조작을 할 것입니까?

위 예제 코드`session.Get ( "uid")`에서 기본적인 데이터로드 작업을 보여했습니다. 여기에 더 자세히 작업을 살펴​​ 보도록합시다 :

func count (w http.ResponseWriter, r * http.Request) {
sess : = globalSessions.SessionStart (w, r)
createtime : = sess.Get ( "createtime")
if createtime == nil {
sess.Set ( "createtime"time.Now () Unix ())
} else if (createtime (int64) + 360) <(time.Now () Unix ()) {
globalSessions.SessionDestroy (w, r)
sess = globalSessions.SessionStart (w, r)
}
ct : = sess.Get ( "countnum")
if ct == nil {
sess.Set ( "countnum"1)
} else {
sess.Set ( "countnum"(ct (int) + 1))
}
t _ : = template.ParseFiles ( "count.gtpl")
w.Header () Set ( "Content-Type", "text / html")
t.Execute (w, sess.Get ( "countnum"))
}

위의 예는 Session 작업과 key / value 데이터베이스를 닮은 조작이다 : Set, Get, Delete 등의 작업을 볼 수 있습니다.

Session은 만기 개념이 있기 때문에, GC 작업을 정의했습니다. 액세스가 만료되면 GC의 트리거 조건을 충족 GC를 호출합니다. 그러나 우리가 어떤 session 작업을 수행하면 Session 개체에 대해 업데이트를 실시해, 마지막 액세스 시간 수정합니다. 이렇게 GC를 할 때 실수로 아직 사용되는 Session 개체를 삭제 해 버리지 않도록합니다.

### session 재구성
Web 어플리케이션은 사용자 로그 아웃 작업이 있습니다. 사용자가 응용 프로그램을 로그 아웃 할 때이 사용자의 session 데이터를 삭제해야합니다. 위의 코드는 이미 어떻게 session의 재구성 작업을 사용하거나 보여줍니다. 여기에서는이 함수가이 기능을 구현합니다 :

// Destroy sessionid
func (manager * Manager) SessionDestroy (w http.ResponseWriter, r * http.Request) {
cookie, err : = r.Cookie (manager.cookieName)
if err! = nil || cookie.Value == ""{
return
} else {
manager.lock.Lock ()
defer manager.lock.Unlock ()
manager.provider.SessionDestroy (cookie.Value)
expiration : = time.Now ()
cookie : = http.Cookie {Name : manager.cookieName, Path : "/"HttpOnly : true, Expires : expiration, MaxAge : -1}
http.SetCookie (w & cookie)
}
}


### session의 파기
는 Session Manager가 어떻게 폐기를 관리하고 있는지 살펴보기로합시다. Main가 호출 될 때 실행하기 만하면됩니다 :

func init () {
go globalSessions.GC ()
}

func (manager * Manager) GC () {
manager.lock.Lock ()
defer manager.lock.Unlock ()
manager.provider.SessionGC (manager.maxlifetime)
time.AfterFunc (time.Duration (manager.maxlifetime) func () {manager.GC ()})
}

GC가 충분히 time 패키지의 타이머 기능을 이용하고있는 것을 알 수 있을까 생각합니다. 시간이`maxLifeTime`를 초과 한 후 GC 함수를 호출했을 때, 이는`maxLiefTime` 시간에서 session을 사용할 수 있는지를 확인할 수 있습니다. 이러한 방법은 또한 온라인 사용자 수 등 통계에 이용할 수도 있습니다.

## 정리
지금까지 Web 어플리케이션의 글로벌 Session 관리에 사용되는 SessionManager를 구현하고 왔습니다. Session을 제공하기 위해 사용되는 스토리지를 정의하고 Provider 인터페이스를 구현했습니다. 다음 절에서는 인터페이스의 정의를 통해 Provider를 구현합니다. 꼭 참고하시기 바랍니다.

