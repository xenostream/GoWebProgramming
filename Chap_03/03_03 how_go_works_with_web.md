# How Go Works with Web

앞 절에서 어떻게 Go 통해 Web 서비스를 세울까 소개했습니다. net / http 패키지를 쉽게 응용하여 편리하게 세울 수 있었는지 생각합니다. 에서는 Go는 낮은 계층에서 도대체 무엇을하고있는 것일까 요? 만물은 모습을 바꿔도 원래​​는 같고 있습니다. Go의 Web 서비스 작업도 제 1 장에서 소개 한 Web의 작업 방식에 관계하고 있습니다.

## web 작업 방법의 몇 가지 개념

다음은 모두 서버의 개념 중 일부입니다

Request : 사용자가 요구하는 데이터입니다. 사용자의 요청 정보를 분석합니다. post, get, cookie, url 등의 정보를 포함합니다.

Response : 서버가 클라이언트에 데이터를 피드백해야합니다.

Conn : 사용자 매번 다시 연결합니다.

Handler : 요청을 처리하고 반환 데이터를 생성하는 처리 로직.

## http 패키지가 수행하는 기능을 분석하는

아래의 그림은 Go가 제공하는 Web 서비스의 작업 모드 프로세스 그림입니다

! [] (images / 3.3.http.png? raw = true)

그림 3.9 http 패키지의 실행 흐름

1. Listen Socket을 생성하고 지정된 포트를 모니터링합니다. 클라이언트의 요청을 기다립니다.

2. Listen Socket은 클라이언트의 요청을 받아들입니다. Client Socket을 얻을 때 Client Socket 통해 클라이언트와 통신합니다.

3. 클라이언트의 요청을 처리합니다. 먼저 Client Socket에서 HTTP 요청의 프로토콜 헤더를 읽고 만약 POST 메서드이면 클라이언트가 입력 한 데이터를 더 읽을 수 없습니다. 그 후 해당 handler가 요청을 처리합니다. handler가 클라이언트 요청하는 데이터를 준비했으면 Client Socket 통해 클라이언트에 써냅니다.

이 모든 과정은 3 개의 문제 만 이해 해두면 상관 없습니다. 이것은 또한 Go가 어떻게하여 Web을 수행 하는가하는 것을 안다는 뜻입니다.

- 어떻게 포트를 감시 하는가?
- 클라이언트의 요청을 어떻게 받아 들일지?
- handler에 어떻게받을 수 있는가?

이전 절의 코드에서는 Go 함수`ListenAndServe` 통해 이러한 일을 처리하고있었습니다. 여기에서는 사실 이와 같이 처리하고 있습니다 : server 객체를 초기화합니다. 그 후`net.Listen ( "tcp"addr)`를 호출합니다. 즉, 낮은 계층에서 TCP 프로토콜을 이용하여 서비스를 시작합니다. 그 후 우리가 설정 한 포트를 모니터링합니다.

아래의 코드는 Go의 http 패키지의 소스 코드에서 인용 한 것입니다. 아래의 코드에서 전체 HTTP 처리 과정을 볼 수 있습니다.

func (srv * Server) Serve (l net.Listener) error {
defer l.Close ()
var tempDelay time.Duration // how long to sleep on accept failure
for {
rw e : = l.Accept ()
if e! = nil {
if ne, ok : = e (net.Error); ok && ne.Temporary () {
if tempDelay == 0 {
tempDelay = 5 * time.Millisecond
} else {
tempDelay * = 2
}
if max : = 1 * time.Second; tempDelay> max {
tempDelay = max
}
log.Printf ( "http : Accept error : % v; retrying in % v", e, tempDelay)
time.Sleep (tempDelay)
continue
}
return e
}
tempDelay = 0
c, err : = srv.newConn (rw)
if err! = nil {
continue
}
go c.serve ()
}
}

모니터링 한 후 어떻게하여 클라이언트의 요청을받을까요? 위의 코드에서는 포트 모니터링을 수행 한 후`srv.Serve (net.Listener)`함수를 호출하고 있습니다. 이 함수는 클라이언트의 요청 정보를 처리하고 있습니다. 이 함수는`for {}`가 놓여 있고 먼저 Listener를 통해 요청을받은 후, Conn를 만듭니다. 마지막으로 단일 goroutine을 엽니 다. 이 요청의 데이터를 인수로서이 conn에 전달합니다. :`go c.serve ()`. 이것은 멀티 스레드를하고 있습니다. 사용자가 수행 요청은 새로운 goroutine 위에서 이루어 서로 영향을주지 않습니다.

에서 어떻게 구체적으로 목적 함수에 요청을 처리하도록 배분 있을까요? conn 먼저 request를 분석합니다 :`c.readRequest ()`다음 원하는 handler를 가져옵니다 :`handler : = sh.srv.Handler`, 즉 우리가 조금 전`ListenAndServe`를 호출했을 때, 그 두 번째 인수입니다. 앞의 예에서 nil을 전달했는데, 이것은 하늘이라는 것입니다. 기본적으로`handler = DefaultServeMux`를 가져옵니다. 이 변수는 도대체 무엇에 사용되는 것일까 요? 그렇습니다. 이 변수는 라우터입니다. 이것은 매치하는 url을 지원하는 handler 함수로 리디렉션하는 데 사용됩니다. 우리는 이것을 설정 한 것입니까? 그래. 우리가 호출 한 코드 맨 먼저에`http.HandleFunc ( "/", sayhelloName)`를 호출 한 잖아요. 이것은`/`를 요청하는 경로 규칙을 등록합니다. url이 "/"를 요청하면 루트는 함수 sayhelloName로 리디렉션합니다. DefaultServeMux는 ServeHTTP 메소드를 호출합니다. 이 메소드 내에서는 사실 sayhelloName 본체를 호출하고 있습니다. 마지막으로 response 정보를 입력하여 클라이언트에 피드백을 제공합니다.


전체 흐름의 자세한 내용은 아래 그림과 같습니다 :

! [] (images / 3.3.illustrator.png? raw = true)

그림 3.10 http 접속의 처리 흐름

여기에 와서 우리는 세 가지 문제에 대해 모든 해답을 얻었습니다. Go가 얼마나 Web을 달리게하거나 이미 기본적인 것은 이해 된 것이 아닐까요?
