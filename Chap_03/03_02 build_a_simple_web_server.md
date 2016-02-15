# Build a Simple Web Server

앞 절에서 Web은 http 프로토콜에 기반한 서비스라고 소개했습니다. Go 언어에서는 전체 net / http 패키지를 제공하고 있습니다. http 패키지를 통해 수행 할 수있는 Web 서비스를 아주 쉽게 시작할 수 있습니다. 동시에이 패키지를 사용하여 쉽게 Web 라우팅 정적 파일, 템플릿, cookie 등의 데이터에 대해 설정하고 조작 할 수 있습니다.

## http 패키지로 Web 서버를 세우는

package main

import (
"fmt"
"net / http"
"strings"
"log"
)

func sayhelloName (w http.ResponseWriter, r * http.Request) {
r.ParseForm () // 옵션을 분석합니다. 기본적으로 분석하지 않습니다.
fmt.Println (r.Form) //이 데이터는 서버의 인쇄 정보에 출력됩니다.
fmt.Println ( "path"r.URL.Path)
fmt.Println ( "scheme"r.URL.Scheme)
fmt.Println (r.Form [ "url_long"])
for k, v : = range r.Form {
fmt.Println ( "key :"k)
fmt.Println ( "val :"strings.Join (v, ""))
}
fmt.Fprintf (w "Hello astaxie!") // 여기서 w에 들어가는 것이 클라이언트에 출력됩니다.
}

func main () {
http.HandleFunc ( "/", sayhelloName) // 액세스 라우팅을 설정합니다.
err : = http.ListenAndServe ( ": 9090", nil) // 감시하는 포트를 설정합니다.
if err! = nil {
log.Fatal ( "ListenAndServe :"err)
}
}

위의 코드는 build 한 후 web.exe을 실행했을 때, 9090 포트로 http 연결 요청을 모니터링합니다.

브라우저에서`http : // localhost : 9090`를 입력하십시오.

브라우저에서`Hello astaxie!`로 출력 된 것을 보았다라고 생각합니다.

주소를 실험 해 보자 :`http : // localhost : 9090 /? url_long = 111 & url_long = 222`

브라우저에서 출력 된 무엇입니까? 서버는 무엇 출력하고 있습니까?

서버에서 출력되는 정보는 다음과 같습니다 :

! [] (images / 3.2.goweb.png? raw = true)

그림 3.8 사용자가 Web에 액세스하여 서버가 출력하는 정보

위의 코드에서 Web 서버를 작성하려면 http 패키지의 두 함수를 호출하는 것으로 좋은 것을 알 수 있습니다.

> 만약 당신이 이전 PHP 프로그래머라면. 이렇게 묻는지도 모릅니다. 우리의 nginx, apache 서버는 필요 없나요과? 왜냐하면이 녀석은 직접 tcp 포트를 감시하기 때문에 nginx가 할 일을 해줍니다. 또한 sayhelloName는 사실 우리가 쓴 논리 함수이므로 php 안의 컨트롤러 (controller) 함수에 가깝습니다.

> 만약 당신이 Python 프로그래머 였다면, tornado을 들어 본 적이 있다고 생각합니다. 이 코드는 그것과 비슷하지 않습니까? 그래, 바로 그거야. Go는 Python과 같은 동적 언어와 비슷한 특성을 가지고 있습니다. Web 응용 프로그램을 작성하는 매우 편리합니다.

> 만약 당신이 Ruby 프로그래머 였다면, ROR의 / script / server를 시작하는 것과 약간 비슷하다는 것을 깨달았다지도 모릅니다.

Go 통해 간단한 몇 줄의 코드로 web 서버를 시작할 수있었습니다. 또한이 Web 서버의 내부에서는 멀티 스레드 특성을 지원합니다. 계속 두 절에서 Go가 얼마나 Web 멀티 스레드를 실현하고 있는지 자세히 소개합니다.
