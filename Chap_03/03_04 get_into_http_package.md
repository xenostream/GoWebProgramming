# Get Into http Package

앞 절에서 Go가 얼마나 Web 작업 모드를 제공하거나 흐름을 소개했습니다. 이 절은 http 패키지를 상세히 해부 해 나가고 있습니다. 이것은 어떻게 전체 프로세스를 실현하고있는 것입니까?

Go의 http로는 2 개의 핵심 기능이 있습니다 : Conn, ServeMux

## Conn의 goroutine
우리가 평소 쓰는 http 서버와 달리 Go는 멀티 스레드와 높은 성능을 실현하기 위해 goroutines를 사용하여 Conn 이벤트 읽고 처리합니다. 이에 따라 각 요청은 독립성을 유지 할 수 있습니다. 서로 차단하지 않고 효율적으로 네트워크 이벤트에 응답 할 수 있습니다. 이것이 Go 높은 효율을 보장합니다.

Go가 클라이언트의 요청을 기다리는에는 다음과 같이 씁니다 :

c, err : = srv.newConn (rw)
if err! = nil {
continue
}
go c.serve ()

클라이언트의 각 요청은 아무도 Conn를 하나 생성하고있는 것을 알 수 있을까 생각합니다. 이 Conn는 이번 요청 정보가 저장되어 있습니다. 이것은 목적의 handler로 전달되고이 handler에서 원하는 handler 정보를 읽을 수 있습니다. 이처럼 각 요청의 독립성을 보장합니다.

## ServeMux의 정의 정의
앞 절에서 conn.server 대해 설명했을 때 주워 내부에서 http 패키지의 기본 루트를 호출하고있었습니다. 라우터를 통해 이번 요청 데이터를 백엔드 처리 함수에 전달합니다. 는이 라우터는 어떻게 실현되고있는 것입니까?

구조는 다음과 같습니다 :

type ServeMux struct {
mu sync.RWMutex // 뮤텍스 요청이 멀티 스레드 처리에 이르렀다 것으로 뮤텍스기구가 필요합니다.
m map [string] muxEntry // 라우팅 규칙 하나의 string이 하나의 mux 엔티티에 대응합니다. 여기서 string은 등록 된 라우팅을 표현하고 있습니다.
hosts bool // 어떤 규칙에 host 정보가 포함되어 있는지
}

다음으로 muxEntry을보고하자

type muxEntry struct {
explicit bool // 정확하게 일치 여부
h Handler //이 라우팅 식은 어떤 handler에 해당하는지
pattern string // 매칭 문자열
}

다음 Handler의 정의를 살펴 보자.

type Handler interface {
ServeHTTP (ResponseWriter * Request) // 라우팅 실현 기
}

Handler는 인터페이스이지만, 앞의 부분 중`sayhelloName` 함수에서는 특히 ServeHTTP라는 인터페이스를 구현하지는 않았습니다. 왜 추가 할 수 있을까요? 원래 http 패키지 중에서는`HandlerFunc`라는 형식이 정의되어 있습니다. 우리가 정의한`sayhelloName` 함수는 바로이 HandlerFunc가 호출 된 결과이며,이 형태는 기본적으로 ServeHTTP 인터페이스를 구현하고있는 것입니다. 즉, HandlerFunc (f)를 호출하여 강제로 f를 HandlerFunc 타입으로 변환하고있는 것입니다. 이렇게하고 f는 ServeHTTP 메소드를 갖게됩니다.

type HandlerFunc func (ResponseWriter * Request)

// ServeHTTP calls f (w, r).
func (f Ha​​ndlerFunc) ServeHTTP (w ResponseWriter, r * Request) {
f (w, r)
}

라우터는 해당 라우팅 규칙을 저장 한 후 구체적으로 어떻게 요청을 배분하는 것입니까? 다음 코드를 참조하십시오. 기본 라우터는`ServeHTTP`을 구현합니다 :

func (mux * ServeMux) ServeHTTP (w ResponseWriter, r * Request) {
if r.RequestURI == "*"{
w.Header () Set ( "Connection", "close")
w.WriteHeader (StatusBadRequest)
return
}
h _ : = mux.Handler (r)
h.ServeHTTP (w, r)
}

위에 있듯이 라우터는 요청을받은 후`*`이면 연결을 해제하고 그렇지 않으면`mux.handler (r) .ServeHTTP (w, r)`를 호출하여 해​​당 설정 처리 Handler를 돌려`h.ServeHTTP (w, r)`를 실행합니다.

즉, 원하는 라우팅 handler의 ServerHTTP 인터페이스에 대한 호출입니다. 는 mux.Handler (r)는 어떻게 처리 할 것인가?

func (mux * ServeMux) Handler (r * Request) (h Handler, pattern string) {
if r.Method! = "CONNECT"{
if p : = cleanPath (r.URL.Path); p! = r.URL.Path {
_ pattern = mux.handler (r.Host, p)
return RedirectHandler (p, StatusMovedPermanently) pattern
}
}
return mux.handler (r.Host, r.URL.Path)
}

func (mux * ServeMux) handler (host, path string) (h Handler, pattern string) {
mux.mu.RLock ()
defer mux.mu.RUnlock ()

// Host-specific pattern takes precedence over generic ones
if mux.hosts {
h, pattern = mux.match (host + path)
}
if h == nil {
h, pattern = mux.match (path)
}
if h == nil {
h, pattern = NotFoundHandler () ""
}
return
}

원래 이것은 사용자의 요청 된 URL과 라우터에 저장되는 map 따라 일치하고 있습니다. 매칭에 의해 저장되는 handler가 반환 될 즈음이 handler의 ServeHTTP 인터페이스가 호출되어 원하는 기능을 수행 할 수 있습니다.

위의 소개를 통해 우리는 라우팅의 전체 프로세스를 이해했습니다. Go는 사실 외부에서 구현 된 라우터를 지원합니다. `ListenAndServe`의 제 2 인자가 외부의 라우터를 설정하기 위해 사용됩니다. 이것은 Handler 인터페이스의 하나로 외부 라우터 Handler 인터페이스를 구현하고 ServeHTTP에 사용자 정의 라우팅 기능을 구현할 수 있습니다.

아래 코드를 통해 자신 간단한 라우터를 구현하려고합니다.

package main

import (
"fmt"
"net / http"
)

type MyMux struct {
}

func (p * MyMux) ServeHTTP (w http.ResponseWriter, r * http.Request) {
if r.URL.Path == "/"{
sayhelloName (w, r)
return
}
http.NotFound (w, r)
return
}

func sayhelloName (w http.ResponseWriter, r * http.Request) {
fmt.Fprintf (w "Hello myroute!")
}

func main () {
mux : = & MyMux {}
http.ListenAndServe ( ": 9090", mux)
}

## Go 코드의 실행 과정

http 패키지에 대한 분석을 통해 전체 코드의 실행 과정을 정리해 봅시다.

- 우선 Http.HandleFunc를 호출합니다.

순서에 따라 몇 가지를 수행합니다.

1 DefaultServeMux의 HandlerFunc를 호출한다.

2 DefaultServeMux의 Handle을 호출한다.

3 DefaultServeMux의 map [string] muxEntry에서 원하는 handler 및 라우팅 규칙을 추가한다.

- 다음 http.ListenAndServe ( ": 9090", nil)을 호출한다.

순서에 따라 몇 가지를 할 :

1 Server의 엔티티 화

2 Server의 ListenAndServe ()를 호출

3 net.Listen ( "tcp", addr)를 호출하고 포트를 감시하는

4 for 루프를 시작 루프에서 요청을 Accept하기

5 각 요청에 Conn를 하나 엔터티 화하고이 요청에 대해 goroutine을 하나 띄워 go c.serve () 서비스를 실시한다.

6 각 요청의 내용을로드 w, err : = c.readRequest ()

7 handler가 존재하는지 판단한다. 만약 handler가 설정되어 있지 않으면 (이 예에서는 handler는 설정하지 않습니다) handler는 DefaultServeMux로 설정됩니다.

8 handler의 ServeHttp를 호출

9이 예에서,이 후 DefaultServeMux.ServeHttp에 들어갑니다

10 request 따라 handler를 선택하고이 handler의 ServeHTTP에 들어갑니다

mux.handler (r) .ServeHTTP (w, r)

11 handler를 선택합니다 :

A 라우터가이 request를 만족했는지 판단합니다 (루프에 의해 ServerMux의 muxEntry을 주사합니다.)

B 만약 라우팅되면이 라우팅 handler의 ServeHttp를 호출합니다.

C 라우팅되지 않으면 NotFoundHandler의 ServeHttp를 호출합니다
