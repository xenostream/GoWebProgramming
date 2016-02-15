# Session and Cookies

session과 cookie의 2 개는 홈페이지의 열람에서 비교적 흔히 볼 개념입니다. 이들은 또한 구별하기 어려운 개념이기도합니다. 그러나 인증이 필요한 서비스 나 페이지의 통계에서는 상당한 중요 해지고 있습니다. 우선 session과 cookie가 도대체​​ 무엇인지 이해하고 나가기로하자이 같은 문제를 생각합니다 :

어떻게 접근에 제한이있는 페이지를된다고하면 좋을까요? 예를 들어 신랑 마이크로 블로그 친구의 메인 페이지와 개인의 마이크로 블로그의 페이지 등입니다.

당연히 브라우저에서 수동으로 사용자 이름과 암호를 입력하고 페이지에 액세스 할 수 있습니다. 소위 "된다고"란 프로그램을 사용하여 유사한 작업을 할 말합니다. 따라서 "로그인"의 과정에서 무엇이 발생하는지 이해할 필요가 있습니다.

사용자가 마이크로 블로그의 로그인 화면에 왔을 때 사용자 이름과 암호를 입력 한 후 "로그인"을 클릭하면 브라우저가 인증 정보를 원격 서버로 전송합니다. 서버는 검증 로직을 실행하여 만약 검증을 통과하면 브라우저는 로그인 한 사용자의 마이크로 블로그의 톱 페이지로 리디렉션합니다. 로그인이 성공하면 서버는 어떻게 우리가 다른 제한이있는 페이지에 대한 액세스를 확인하는 것입니까? HTTP 프로토콜은 비 상태이므로, 서버는 우리가 이전 HTTP 요청에서 검증을 통과 한 것을 알 수가 없습니다. 당연히 가장 간단한 해결 방법은 모든 요청에​​ 사용자 이름과 암호를 포함하는 것입니다. 이래도 상관 없지만 서버의 부하를 매우 높여 버립니다. (매번 다시 모든 데이터베이스의 확인을 필요로합니다.) 사용자의 경험도 저하됩니다. (모든 페이지에서 다시 사용자 이름과 암호를 입력해야합니다. 모든 페이지에 로그인 폼이 나옵니다.) 직접 요청에서 사용자 이름과 암호를 포함 할 수는 없기 때문에 서버 또는 클라이언트 신분을 나타내는 정보의 종류를 저장 할 수 밖에 없습니다. cookie와 session은이를 위해 있습니다.

cookie는 간단히 로컬 컴퓨터에 저장된 사용자의 작업 이력 정보입니다 (당연히 로그인 정보를 포함합니다). 또한 사용자가 다시이 페이지에 액세스 할 때 브라우저는 HTTP 프로토콜을 통해 로컬 cookie의 내용을 서버에 전송하고 확인합니다. 또는 계속해서 이전을 수행합니다.

! [] (images / 6.1.cookie2.png? raw = true)

그림 6.1 cookie의 원리도

session은 간단히 말하면 서버에 저장된 사용자의 작업 이력 정보입니다. 서버는 session id를 사용하여 session을 식별합니다. session id는 서버가 생성합니다. 임의성과 고유성을 보장하고 임의의 비밀 키에 해당합니다. 핸드 셰이크 및 데이터 통신 중에 사용자의 실제 비밀번호가 노출되는 것을 방지합니다. 그러나이 방법은 여전히​​ 요청을 보낸 클라이언트 또는 session을 대응시킬 필요가 있습니다. 따라서 cookie 메커니즘을 통해 클라이언트의 ID (session id)를 취득하여 GET 메서드 id를 서버로 보낼 수 있습니다.

! [] (images / 6.1.session.png? raw = true)

그림 6.2 session의 원리도

## cookie
Cookie는 브라우저에 의해 유지됩니다. 클라이언트 작은 텍스트 정보로 저장됩니다. 사용자의 요청과 화면에 따라 Web 서버와 브라우저 사이에 전송됩니다. 사용자가 페이지에 접근했을 때, Web 어플리케이션은 cookie에 포함 된 정보를 읽을 수 있습니다. 브라우저의 설정에서 cookie 개인 데이터를 선택할 수 있습니다. 이것을 열면 이미 방문한 적이있는 페이지의 cookie를 많이 볼 수 있습니다. 아래의 그림을 참조하십시오 :

! [] (images / 6.1.cookie.png? raw = true)

그림 6.3 브라우저에서 저장되는 cookie 정보

cookie에는 유효 기간이 있습니다. 유효 기간의 차이에 따라 2 가지로 나눌 수 있습니다 : 세션 cookie 및 지속 쿠키가 있습니다.

만약 유효 기간을 설정하지 않으면이 cookie의 유효 기간은 새로 만들어진에서 브라우저를 닫을 때까지이되어 cookie은 소멸됩니다. 이러한 유효 기간은 열람시의 세션의 세션 cookie라고합니다. 세션 cookie는 일반적으로 하드 디스크에 저장되지 않고 메모리에 저장됩니다.

만약 유효 기간 (setMaxAge (60 * 60 * 24))이 설정되어 있으면 브라우저는 cookie를 하드 디스크에 저장합니다. 브라우저를 닫았다가 다시 열면 이러한 cookie은 여전히​​ 설정된 만기까지 유효합니다. 하드 디스크에 저장된 cookie 다른 브라우저 프로세스간에 공유 할 수 있습니다. 예를 들어 IE를 2 개 열고 메모리에 저장된 cookie 대해 다른 브라우저는 다른 방식을 취합니다.


### Go에서 cookie를 설정하기
Go 언어에서는 net / http 패키지 SetCookie 통해 설정합니다 :

http.SetCookie (w ResponseWriter, cookie * Cookie)

w는 입력해야하는 response, cookie는 struct입니다. cookie 객체가 어떻게되어 있는지 살펴 보자.

type Cookie struct {
Name string
Value string
Path string
Domain string
Expires time.Time
RawExpires string

// MaxAge = 0 means no 'Max-Age'attribute specified.
// MaxAge <0 means delete cookie now, equivalently 'Max-Age : 0'
// MaxAge> 0 means Max-Age attribute present and given in seconds
MaxAge int
Secure bool
HttpOnly bool
Raw string
Unparsed [] string // Raw text of unparsed attribute-value pairs
}

예를 하나 살펴 보자. 어떻게 cookie를 설정하는 방법이다.

expiration : = time.Now ()
expiration = expiration.AddDate (1, 0, 0)
cookie : = http.Cookie {Name : "username", Value : "astaxie"Expires : expiration}
http.SetCookie (w & cookie)


### Go에서 cookie보기
위의 예에서는 어떻게 cookie 데이터를 설정하거나 설명했습니다. 여기에서는 어떻게 cookie를 읽을 것인지 살펴 보자.

cookie _ : = r.Cookie ( "username")
fmt.Fprint (w, cookie)

또 하나의로드 방법은

for _ cookie : = range r.Cookies () {
fmt.Fprint (w, cookie.Name)
}

request 통해 cookie가 매우 쉽게 얻을 수있는 것이 이해할 수 있다고 생각합니다.

## session

session, 중국어에서는 흔히 '대화'로 번역됩니다. 본래는 처음부터 끝까지의 일련의 액션 / 메시지를 의미합니다. 예를 들어 전화를 걸 때는 수화기를 손에 취해 전화 번호를 눌러 전화를 끊을 사이의 일련의 과정을 session이라고 부를 수 있습니다. 그러나 session이라는 말이 네트워크 프로토콜과 관계시는 종종 "연결된 통신"또는 / 혹은 "상태 유지"의 두 가지 의미가 포함되어 있습니다.

session은 Web 개발 환경에서는 또한 새로운 의미가 포함되어 있습니다. 클라이언트 측과 서버 측 사이에 상태를 유지하기위한 솔루션입니다. 종종 Session은 또한 이러한 솔루션의 저장 구조도 말합니다.

session 메커니즘은 서버 사이드 메커니즘입니다. 서버에서 해시 테이블의 구조와 비슷 (해시 테이블을 사용하는 경우도 있습니다)를 사용하여 정보를 저장합니다.

그러나 프로그램이 클라이언트의 요청에 session을 확립 할 필요가있는 경우, 서버는 우선 클라이언트의 요청에 sessionID가 있는지를 검사합니다. 서버는 session id를 참조하여이 session을 검색 (검색 할 수없는 경우 새로 생성됩니다. 이러한 상황은 서버가 이미이 사용자에 대응하는 session 객체를 삭제 해 버린 경우에 발생 얻습니다 그러나 사용자는 인위적으로 요청 URL의 끝에 JSESSION 인수를 추가합니다.) 사용합니다. 만약 사용자의 요청에 session id가 포함되어야이 사용자에 session을 생성하고 동시에이 session과 관련된 session id를 생성합니다. 이 session id는 이번 대한 응답으로 클라이언트에 반환 저장됩니다.

session 메커니즘 자체는 특히 복잡하지 않지만 구현 및 설정의 유연성은 매우 복잡합니다. 이것은 한번의 경험과 하나의 브라우저, 서버 만의 경험에서 가지고 보편적으로 통용되는 것은 아닙니다.

## 정리

언급 한 바와 같이 session과 cookie의 목적은 동일합니다. 모두 http 프로토콜의 무국적이라는 단점을 극복하기 위해 있습니다. 그러나 그 방법은 다릅니다. session는 cookie를 통해 클라이언트에 session id를 저장합니다. 또한 사용자의 다른 세션 정보는 서버의 session 객체에 저장됩니다. 이와는 대조적으로, cookie는 모든 정보를 클라이언트에 갖게 할 필요가 있습니다. 따라서 cookie에는 어느 정도 잠재적 인 위협이 존재합니다. 예를 들어 로컬 cookie에 저장된 사용자 이름과 암호가 해독되고, cookie가 다른 홈페이지에 수집됩니다 (예 : 1.appA이 주도적으로 지역 B의 cookie를 설정하고 영역 B에 cookie를 취득 시킵니다; 2.XSS, appA에서 javascript 통해 document.cookie를 취득하고 자신의 appB로 보냅니다).


위의 몇 가지 간단한 소개로 cookie와 session의 기초 지식을 소개했습니다. 이들 사이의 관계와 구별을 알고 web 개발하기 전에 필요한 지식을 미리 숙지하여 대응에 곤궁하고 bug 수정을 할 때 우연한가되기도 않아도됩니다. 이후 여러 장에서는 session에 관한 더 섬세한 기술에 대해 소개합니다.

