# Sockets

많은 낮은 계층의 네트워크 응용 프로그램 개발자의 눈에는 일체의 프로그램이 모두 Socket처럼 보입니다. 조금 과장일지도 모릅니다 만, 대체로 이런 것입니다. 현재 네트워크 프로그래밍은 거의 모든면에서 Socket을 사용하여 프로그램되어 있습니다. 이 같은 장면을 생각해 본 적이 없습니까? 매일 브라우저를 열고 페이지를 탐색 할 때 브라우저 프로세스는 어떻게하고 Web 서버와 통신을하고있는 것일까 요? QQ를 사용하여 채팅 할 때, QQ 프로세스는 어떻게 해 서버 또는 당신의 친구가 QQ 프로세스와 통신을하고있는 것일까? PPstream을 열고 스트리밍 영상을 볼 때, PPstream 프로세스는 어떻게하고 동영상 서버와 통신을하고있는 것일까 요? 이처럼 모든 Socket에 의존하여 통신을하고 있습니다. 하나를보고 모두를 이해하면 Socket 프로그래밍은 현대의 프로그래밍 중에서도 많은 중요한 위치를 차지하고 있음을 알 수있다. 이 장에서는 Go 언어에서 어떻게 Socket 프로그래밍을 할 것인가 소개합니다.

## Socket이란 무엇인가?
Socket은 Unix를 기원합니다. Unix의 기본 철학 중 하나는 "모든 파일 인"입니다. 모두는 "열 open -> 읽고 write / read -> 닫기 close"패턴에 의해 조작됩니다. Socket이 패턴의 구현입니다. 네트워크 Socket 데이터 통신은 특수 I / O 중 하나입니다. Socket도 파일 디스크립터의 일종입니다. Socket도 파일을 열 함수를 가지고 있습니다 : Socket (). 이 함수는 int 형의 Socket을 반환한다. 이후의 연결을 설치해서 데이터 전송 등 모든 작업이 Socket을 통과함으로써 실현됩니다.

자주 사용되는 Socket에는 두 종류가 있습니다 : 스트림 Socket (SOCK_STREAM)과 데이터 그램 Socket (SOCK_DGRAM)입니다. 스트림은 연결 지향 Socket의 일종입니다. 연결 지향 TCP 서비스 응용 프로그램에 사용됩니다. 또한 데이터 그램 Socket 무 연결 Socket의 일종입니다. 연결없는 UDP 서비스 응용 프로그램에 사용됩니다.
## Socket은 어떻게 통신을하거나
네트워크에서 프로세스 간은 어떻게하고 Socket을 사용하여 통신을 할 것입니까? 먼저 해결해야 할 문제는 어떻게 독특한 프로세스 중 하나를 인식 하는가하는 것입니다. 그렇지 않으면 통신을 시작조차 남아되지 않습니다! 로컬 프로세스 PID를 사용하여 하나의 프로세스를 식별합니다. 사실 TCP / IP 프로토콜 족에서는 이미이 문제를 해결 해주고 있습니다. 네트워크 계층 "ip 주소"는 네트워크의 호스트를 고유하게 인식하고 있으며, 전송 계층 "프로토콜 + 포트"는 호스트 어플리케이션 프로그램 (프로세스)을 고유하게 인식 할 수 있습니다. 이러한 세 가지 조합 (ip 주소, 프로토콜, 포트)를 이용하여 네트워크 프로세스를 인식 할 수 있습니다. 네트워크에서 통신을 수행해야하는 프로세스는이 태그를 이용하여 서로 통신을 할 수 있습니다. 다음 TCP / IP 프로토콜의 구조도를 참조하십시오.

! [] (images / 8.1.socket.png? raw = true)

그림 8.1 일곱 계층 네트워크 프로토콜의 그림

TCP / IP 프로토콜을 사용하는 응용 프로그램은 일반적으로 응용 프로그램의 포트를 채용합니다 : UNIX BSD 소켓 (socket)과 UNIX System V의 TLI (이미 도태되어 있습니다)에 의해 네트워크 프로세스 간 통신을 제공합니다 . 현재는 거의 모든 어플리케이션 프로그램은 socket이 채용되고 있습니다. 현재는 네트워크 시대에서 네트워크 프로세스 통신은 어디든지 존재합니다. "모든 Socket"이란 이런 것입니다.

## Socket의 기초 지식
Socket에는 두 종류 있는지 소개했습니다 : TCP Socket 및 UDP Socket입니다. TCP와 UDP 프로토콜에서 하나의 프로세스를 확정하기 위해서는 3 개의 쌍이 필요합니다. IP 주소와 포트 번호가 필요합니다.

### IPv4 주소
현재의 인터넷이 채택하고있는 프로토콜 족은 TCP / IP 프로토콜입니다. IP는 TCP / IP 프로토콜의 네트워크 계층의 프로토콜입니다. TCP / IP 프로토콜 족의 핵심 프로토콜입니다. 현재 주로 사용되고있는 IP 프로토콜 버전 4 (간단한 용어는 IPv4)입니다. 현재까지 30여 년여에 걸쳐 사용되어 왔습니다.

IPv4 주소 비트는 32 비트입니다. 즉 많아도 2의 32 제곱의 네트워크 장비가 Internet에 연결 할 수 있습니다. 지난 십 년간 인터넷의 강력한 발전에 따라 IP 주소에 대한 수요가 높아지고 있습니다. IP 주소 할당은 더 엄격 해 이전 보도에 따르면 IPV4 주소가 이미 할당이 종료되었습니다. 우리의 회사에 현재 많이 존재하는 서버의 IP는 모두 귀중한 자원의 하나가되고 있습니다.

주소 형식이 같다 : 127.0.0.1 172.122.121.111

### IPv6 주소
IPv6는 다음 버전의 인터넷 프로토콜입니다. 차세대 인터넷 프로토콜해도 괜찮습니다. 이것은 IPv4의 실시 과정에서 발생하는 각종 문제를 해결하기 위해 제안 된 것입니다. IPv6는 128 비트 주소 길이를 채용하고있어 거의 무제한으로 주소를 제공 할 수 있습니다. IPv6을 실제로 분배 할 수있는 주소를 계산하면 싸게 잡아도 지구상의 1 평방 미터의 면적에 1000 개 이상의 주소를 할당 할 수 있습니다. IPv6의 설계에 있어서는 미리 주소 고갈 문제를 해결하는 이외에, IPv4에서 잘 해결하지 못한 다른 문제도 고려하고 있습니다. 주로 종단 간 IP 연결 퀄리티 오브 서비스 (QoS), 보안, 멀티 캐스트 이동성 플러그 앤 플레이 등입니다.

주소의 형식은 다음과 같습니다 : 2002 : c0e8 : 82e7 : 0 : 0 : 0 : c0e8 : 82e7

### Go에서 지원되는 IP 형식
Go의`net` 패키지는 여러 가지 형태가 정의되어 있습니다. 함수와 메서드는 네트워크 프로그래밍을 위해 사용됩니다. 이 중 IP 정의는 다음과 같습니다 :

type IP [] byte

`net` 패키지에서는 많은 함수가 IP를 조작합니다. 그러나 상대적으로 사용되는 것은 몇 개 밖에 없습니다. 이 중`ParseIP (s string) IP` 함수는 IPv4 또는 IPv6 주소를 IP ​​형식으로 변환합니다. 아래의 예를 참조하십시오 :

package main
import (
"net"
"os"
"fmt"
)
func main () {
if len (os.Args)! = 2 {
fmt.Fprintf (os.Stderr "Usage : % s ip-addr \ n", os.Args [0])
os.Exit (1)
}
name : = os.Args [1]
addr : = net.ParseIP (name)
if addr == nil {
fmt.Println ( "Invalid address")
} else {
fmt.Println ( "The address is", addr.String ())
}
os.Exit (0)
}

실행하면 IP 주소를 입력하여 해당 IP 형식이 출력되는 아실 거라고 생각합니다.

## TCP Socket
어떻게 네트워크 포트에서 서비스에 액세스하거나 안다면 무엇을 할 수 있을까요? 클라이언트에서하면 원격 장치의 네트워크 포트에 대한 요청을 하나 전송하여 장비의 포트를 감시하고있는 서버의 피드백 정보를 얻을 수 있습니다. 서버에서하면 서버의 지정된 포트에 끈 부착이 포트를 감시해야합니다. 클라이언트 액세스가있을 때 정보를 취득하고 피드백 정보를 기록 할 수 있습니다.

Go 언어의`net` 패키지에는`TCPConn`라는 형태가 있습니다. 이 형태는 클라이언트와 서버 간의 상호 작용의 통로로 사용 할 수 있습니다. 여기에는 크게 두 가지의 함수가 존재합니다 :

func (c * TCPConn) Write (b [] byte) (n int, err os.Error)
func (c * TCPConn) Read (b [] byte) (n int, err os.Error)

`TCPConn`는 클라이언트와 서버가 데이터를 읽고 쓰는 데 사용할 수 있습니다.

또한`TCPAddr` 형도 알고 있어야합니다. 이것은 TCP의 주소 정보를 보여줍니다. 이 정의는 다음과 같다 :

type TCPAddr struct {
IP IP
Port int
}
Go 언어에서는`ResolveTCPAddr`를 사용하여 하나의`TCPAddr`를 가져옵니다.

func ResolveTCPAddr (net, addr string) (* TCPAddr, os.Error)

- net 인수는 "tcp4", "tcp6", "tcp"중 임의의 하나입니다. 각각 TCP (IPv4-only), TCP (IPv6-only) 및 TCP (IPv4, IPv6 어떤 하나)를 나타냅니다.
- addr는 도메인 또는 IP 주소를 보여줍니다. 예를 들어 "www.google.com:80"또는 "127.0.0.1:22"입니다.


### TCP client
Go 언어에서는 net 패키지의`DialTCP` 함수에 의해 TCP 연결을 하나 설립하고`TCPConn` 형의 객체를 하나 돌려줍니다. 연결했을 때 서버도 동일한 형태의 객체를 작성합니다. 이때 클라이언트와 서버는 각자가 가지고있는`TCPConn` 개체를 사용하여 데이터를 교환합니다. 일반적으로 클라이언트는`TCPCon` 객체를 사용하여 요청 정보를 서버로 전송하고 서버의 응답 정보를 읽습니다. 서버는 클라이언트의 요청을 읽고 해석하여 응답 정보를 반환합니다. 이 연결은 어느 쪽인지가 연결을 해제해야만 해지하고 그렇지 않으면이 연결은 계속 사용할 수 있습니다. 연결을 설정하는 함수의 정의는 다음과 같습니다 :

func DialTCP (net string, laddr, raddr * TCPAddr) (c * TCPConn, err os.Error)

- net 인수는 "tcp4", "tcp6", "tcp"중 임의의 하나입니다. 각각 TCP (IPv4-only), TCP (IPv6-only) 및 TCP (IPv4, IPv6 어떤 하나)를 나타냅니다.
- laddr은 로컬 주소를 나타냅니다. 일반적으로 nil을 설정합니다.
- raddr는 원격 서버 주소를 나타냅니다.

여기에서는 간단한 예를 하나 써 봅시다. HTTP 프로토콜 기반 클라이언트에 의한 Web 서버에 요청을 에뮬레이트합니다. 간단한 http 요청 헤더를 작성해야합니다. 형식은 다음과 같습니다 :

"HEAD / HTTP / 1.0 \ r \ n \ r \ n"

서버로부터받은 응답 정보의 형식은 다음과 같습니다 :

HTTP / 1.0 200 OK
ETag : "-9985996"
Last-Modified : Thu, 25 Mar 2010 17:51:10 GMT
Content-Length : 18074
Connection : close
Date : Sat, 28 Aug 2010 00:43:48 GMT
Server : lighttpd / 1.4.23

우리의 클라이언트 코드는 다음과 같습니다 :

package main

import (
"fmt"
"io / ioutil"
"net"
"os"
)

func main () {
if len (os.Args)! = 2 {
fmt.Fprintf (os.Stderr "Usage : % s host : port"os.Args [0])
os.Exit (1)
}
service : = os.Args [1]
tcpAddr, err : = net.ResolveTCPAddr ( "tcp4"service)
checkError (err)
conn, err : = net.DialTCP ( "tcp", nil, tcpAddr)
checkError (err)
_ err = conn.Write ([] byte ( "HEAD / HTTP / 1.0 \ r \ n \ r \ n"))
checkError (err)
result, err : = ioutil.ReadAll (conn)
checkError (err)
fmt.Println (string (result))
os.Exit (0)
}
func checkError (err error) {
if err! = nil {
fmt.Fprintf (os.Stderr "Fatal error : % s", err.Error ())
os.Exit (1)
}
}

위의 코드에서 아는 것은 : 우선 프로그램은 사용자의 입력을 인수`service`로`net.ResolveTCPAddr`에 나, tcpAddr를 하나 가져옵니다. 그 tcpAddr을 DialTCP에 전달 TCP 연결`conn`을 설정합니다. `conn` 통해 요청 정보를 전송하고 마지막에`ioutil.ReadAll` 통해`conn`에서 모든 텍스트, 즉 서버의 요청 피드백 정보를 가져옵니다.

### TCP server
위의 TCP 클라이언트 프로그램을 썼습니다. 또한 net 패키지를 사용하여 서버 프로그램을 만들 수 있습니다. 서버에서 서비스를 지정 활성화되지 않은 포트에 끈 부착이 포트를 감시해야합니다. 클라이언트의 요청이 도착했을 때 클라이언트에서 연결 요청을받을 수 있습니다. net 패키지에는 해당 기능의 함수가 있습니다. 함수의 정의는 다음과 같습니다 :

func ListenTCP (net string, laddr * TCPAddr) (l * TCPListener, err os.Error)
func (l * TCPListener) Accept () (c Conn, err os.Error)

인수의 설명은 DialTCP 인수와 동일합니다. 다음은 간단한 시간 동기화 서비스를 구현하고 있습니다. 7777 포트를 모니터링합니다.

package main

import (
"fmt"
"net"
"os"
"time"
)

func main () {
service : = ": 7777"
tcpAddr, err : = net.ResolveTCPAddr ( "tcp4"service)
checkError (err)
listener, err : = net.ListenTCP ( "tcp"tcpAddr)
checkError (err)
for {
conn, err : = listener.Accept ()
if err! = nil {
continue
}
daytime : = time.Now () String ()
conn.Write ([] byte (daytime)) // do not care about return value
conn.Close () // we 're finished with this client
}
}
func checkError (err error) {
if err! = nil {
fmt.Fprintf (os.Stderr "Fatal error : % s", err.Error ())
os.Exit (1)
}
}

위 서비스는 이동하면 계속 그래서 새로운 클라이언트가 요청을 보내 오기를 기다리고 있습니다. 새로운 클라이언트 요청이 도착 접수`Accept`에 동의하면이 요청시 현재 시간 정보를 피드백합니다. 주의해야 할 코드 중`for` 루프입니다. 오류가 발생했을 때 직접 continue하고 루프를 뺄 수 없습니다. 왜냐하면 서버가 코드를 실행 한 후 오류가 발생하는 상황에서는 서버에 오류를 기록하고 현재 연결된 클라이언트가 직접 오류를 발생시켜 로그 아웃합니다. 따라서 현재 서버가 실행중인 서비스 전체에는 영향을주지 않습니다.

위의 코드는 단점이 있습니다. 수행 될 때는 하나의 작업입니다. 동시에 여러 요청을받을 수 없습니다. 그럼 어떻게 병렬 처리를 지원할 수 있도록 개조하는 것입니까? Go는 goroutine 메커니즘이 있습니다. 아래의 개조 후 코드를 참조하십시오.

package main

import (
"fmt"
"net"
"os"
"time"
)

func main () {
service : = ": 1200"
tcpAddr, err : = net.ResolveTCPAddr ( "tcp4"service)
checkError (err)
listener, err : = net.ListenTCP ( "tcp"tcpAddr)
checkError (err)
for {
conn, err : = listener.Accept ()
if err! = nil {
continue
}
go handleClient (conn)
}
}

func handleClient (conn net.Conn) {
defer conn.Close ()
daytime : = time.Now () String ()
conn.Write ([] byte (daytime)) // do not care about return value
// we 're finished with this client
}
func checkError (err error) {
if err! = nil {
fmt.Fprintf (os.Stderr "Fatal error : % s", err.Error ())
os.Exit (1)
}
}

작업 처리 함수`handleClinet` 분리하여 더 나아가 병렬 실행을 제공 할 수 있습니다. 외형은 너무 멋있고 없습니다. `go` 키워드를 추가하여 서버의 병렬 처리를 실현했습니다. 이 예에서 goroutine의 강력 함을 볼 수 있습니다.

이렇게 생각하는 분도 계실지 모릅니다 :이 서버는 클라이언트가 실제로 요청 된 콘텐츠를 처리하지 않습니다. 만약 클라이언트에서 다른 요청에서 다른 시간 형식을 요구, 게다가 장시간에 걸친 연결 인 경우 어떻게하면 좋을까? 와. 이 경우 다음을 참조하십시오 :

package main

import (
"fmt"
"net"
"os"
"time"
"strconv"
)

func main () {
service : = ": 1200"
tcpAddr, err : = net.ResolveTCPAddr ( "tcp4"service)
checkError (err)
listener, err : = net.ListenTCP ( "tcp"tcpAddr)
checkError (err)
for {
conn, err : = listener.Accept ()
if err! = nil {
continue
}
go handleClient (conn)
}
}

func handleClient (conn net.Conn) {
conn.SetReadDeadline (time.Now () Add (2 * time.Minute)) // set 2 minutes timeout
request : = make ([] byte 128) // set maxium request len​​gth to 128KB to prevent flood attack
defer conn.Close () // close connection before exit
for {
read_len, err : = conn.Read (request)

if err! = nil {
fmt.Println (err)
break
}

    if read_len == 0 {
    break // connection already closed by client
    } else if string (request) == "timestamp"{
    daytime : = strconv.FormatInt (time.Now () Unix () 10)
    conn.Write ([] byte (daytime))
    } else {
    daytime : = time.Now () String ()
    conn.Write ([] byte (daytime))
    }

    request = make ([] byte 128) // clear last read content
}
}

func checkError (err error) {
if err! = nil {
fmt.Fprintf (os.Stderr "Fatal error : % s", err.Error ())
os.Exit (1)
}
}

위의 예에서는`conn.Read ()`를 사용하여 클라이언트가 보내는 요청을 끊임없이 읽고 있습니다. 고객과 장시간 연결을 유지해야하기 때문에, 한번의 요청을 읽은 후에도 연결을 끊을 수 없습니다. `conn.SetReadDeadline ()`는 타임 아웃을 설정하고 있기 때문에 일정 시간 내에 클라이언트에서 요청을 보내야`conn`는 자동으로 연결을 끊습니다. 아래의 for 루프는 연결이 끊어 됨으로써 빠져 있습니다. 주의해야 할 것은`request`는 새로 생성 될 때 flood attack을 방지하기 위해 최대 길이를 지정해야한다는 것입니다; 매번 요청이로드 처리가 완료 할 때마다 request를 정리해야합니다. 왜냐하면`conn.Read ()`는 새로 읽은 내용을 원래 내용 뒤에 append 버릴지도 모르기 때문입니다.

### TCP 연결 제어
TCP는 많은 연결 컨트롤 기능이 있습니다. 우리가 평소 자주 사용하는 것은 다음의 몇 가지 함수입니다 :

func DialTimeout (net, addr string, timeout time.Duration) (Conn, error)

연결 시간 제한을 설정하면 클라이언트와 서버 모두에 적용됩니다. 설정된 시간이 지나면 연결이 자동으로 해제됩니다.

func (c * TCPConn) SetReadDeadline (t time.Time) error
func (c * TCPConn) SetWriteDeadline (t time.Time) error
  
연결 시간 초과에 대한 읽기 / 쓰기를 설정하는 데 사용됩니다. 연결이 자동으로 해제됩니다.

func (c * TCPConn) SetKeepAlive (keepalive bool) os.Error

클라이언트가 서버와 장시간 연결을 유지 여부 설정하여 TCP 연결시 핸드 셰이크의 오버 헤드를 줄일 수 있습니다. 자주 데이터를 교환해야하는 애플리케이션에 적합합니다.

자세한 내용은`net` 패키지의 문서를 참조하십시오.
## UDP Socket
Go 언어의 UDP Socket과 TCP Socket 처리의 차이는 서버에서 처리되는 복수의 클라이언트 요청 데이터 패킷의 방법입니다. UDP는 클라이언트의 연결 요청에 대한 Accept 함수가 부족합니다. 기타 기본적으로 거의 동일합니다. TCP를 단지 UDP로 대체하면됩니다. UDP의 주요 몇 가지 기능은 다음과 같다 :

func ResolveUDPAddr (net, addr string) (* UDPAddr, os.Error)
func DialUDP (net string, laddr, raddr * UDPAddr) (c * UDPConn, err os.Error)
func ListenUDP (net string, laddr * UDPAddr) (c * UDPConn, err os.Error)
func (c * UDPConn) ReadFromUDP (b [] byte) (n int, addr * UDPAddr, err os.Error
func (c * UDPConn) WriteToUDP (b [] byte, addr * UDPAddr) (n int, err os.Error)

UDP 클라이언트 코드는 다음과 같습니다있다. 차이점은 TCP를 UDP로 대체 뿐이라고 알 수 있습니다.

package main

import (
"fmt"
"net"
"os"
)

func main () {
if len (os.Args)! = 2 {
fmt.Fprintf (os.Stderr "Usage : % s host : port"os.Args [0])
os.Exit (1)
}
service : = os.Args [1]
udpAddr, err : = net.ResolveUDPAddr ( "udp4"service)
checkError (err)
conn, err : = net.DialUDP ( "udp", nil, udpAddr)
checkError (err)
_ err = conn.Write ([] byte ( "anything"))
checkError (err)
var buf [512] byte
n, err : = conn.Read (buf [0 :)
checkError (err)
fmt.Println (string (buf [0 : n]))
os.Exit (0)
}
func checkError (err error) {
if err! = nil {
fmt.Fprintf (os.Stderr "Fatal error", err.Error ())
os.Exit (1)
}
}

UDP 서버가 어떻게 처리하는지 살펴 보자;

package main

import (
"fmt"
"net"
"os"
"time"
)

func main () {
service : = ": 1200"
udpAddr, err : = net.ResolveUDPAddr ( "udp4"service)
checkError (err)
conn, err : = net.ListenUDP ( "udp"udpAddr)
checkError (err)
for {
handleClient (conn)
}
}
func handleClient (conn * net.UDPConn) {
var buf [512] byte
_ addr, err : = conn.ReadFromUDP (buf [0 :)
if err! = nil {
return
}
daytime : = time.Now () String ()
conn.WriteToUDP ([] byte (daytime) addr)
}
func checkError (err error) {
if err! = nil {
fmt.Fprintf (os.Stderr "Fatal error", err.Error ())
os.Exit (1)
}
}

## 정리
TCP와 UDP Socket 프로그래밍의 묘사와 구현을 통해 Go 이미 Socket 프로그래밍을 완벽하게 지원하는 것을 알 주실 수 있었다고 생각합니다. 사용시 매우 편리합니다. Go 많은 기능을 제공합니다. 이 함수를 사용하여 쉽게 고성능 Socket 응용 프로그램을 작성할 수 있습니다.

