# Deployment

프로그램의 개발이 완료되면 Web 응용 프로그램을 배포해야합니다. 그러나 이러한 프로그램은 어떻게 전개 할 것인가? Go 프로그램이 컴파일 된 후 실행 파일이되기 때문에 C 프로그램을 쓴 적이있는 독자라면 아마도 daemon을 채용하여 완벽하게 프로그램을 백그라운드에서 지속적으로 수행 할 수 있て 계신다고 생각합니다. 그러나 현재 Go 완전히 daemon을 제공 할 수 없습니다. 따라서 Go 응용 프로그램을 배포함에있어서 타사 도구를 사용하여 관리 할 수​​ 있습니다. 타사 도구에 몇 가지 있습니다. 예를 들어, Supervisord, upstart, daemontools 등입니다. 이 절에서는 현재 자신의 시스템에서 채택하고있는 도구 Supervisord을 소개하고 싶습니다.
## daemon
현재 Go 프로그램은 daemon을 구현하는 것은 아직 없습니다. 이 Go 언어의 bug에 대한 자세한 내용은 <`http : //code.google.com/p/go/issues/detail? id = 227`>를 참조하십시오. 요약해서 말하면 현재 사용하고있는 프로세스에서 fork하는 것은 매우 어렵다는 것입니다. 쉽게 이미 사용되는 모든 프로세스의 상태를 일치시키는 방법이 없기 때문입니다.

그러나 많은 웹 사이트에서 daemon을 구현하는 방법을 볼 수 있습니다. 예를 들어 다음의 두 가지 방법입니다 :

- MarGo 구현 사상의 하나로, Command를 사용하여 자신의 응용 프로그램을 실행합니다. 만약 정말 구현하고 싶은 경우,이 솔루션을 추천합니다.

d : = flag.Bool ( "d", false "Whether or not to launch in the background (like a daemon)")
if * d {
cmd : = exec.Command (os.Args [0]
"-close-fds"
"-addr"* addr,
"-call"* call,
)
serr, err : = cmd.StderrPipe ()
if err! = nil {
log.Fatalln (err)
}
err = cmd.Start ()
if err! = nil {
log.Fatalln (err)
}
s, err : = ioutil.ReadAll (serr)
s = bytes.TrimSpace (s)
if bytes.HasPrefix (s [] byte ( "addr :")) {
fmt.Println (string (s))
cmd.Process.Release ()
} else {
log.Printf ( "unexpected response from MarGo :`% s` error :`% v` \ n", s err)
cmd.Process.Kill ()
}
}

- 다른 하나는 syscall을 이용한 솔루션입니다. 그러나이 솔루션은 완벽하지 않습니다 :

package main

import (
"log"
"os"
"syscall"
)

func daemon (nochdir, noclose int) int {
var ret, ret2 uintptr
var err uintptr

darwin : = syscall.OS == "darwin"

// already a daemon
if syscall.Getppid () == 1 {
return 0
}

// fork off the parent process
ret, ret2, err = syscall.RawSyscall (syscall.SYS_FORK, 0, 0, 0)
if err! = 0 {
return -1
}

// failure
if ret2 <0 {
os.Exit (-1)
}

// handle exception for darwin
if darwin && ret2 == 1 {
ret = 0
}

// if we got a good PID, then we call exit the parent process.
if ret> 0 {
os.Exit (0)
}

/ * Change the file mode mask * /
_ = syscall.Umask (0)

// create a new SID for the child process
s_ret, s_errno : = syscall.Setsid ()
if s_errno! = 0 {
log.Printf ( "Error : syscall.Setsid errno : % d", s_errno)
}
if s_ret <0 {
return -1
}

if nochdir == 0 {
os.Chdir ( "/")
}

if noclose == 0 {
f e : = os.OpenFile ( "/ dev / null"os.O_RDWR 0)
if e == nil {
fd : = f.Fd ()
syscall.Dup2 (fd, os.Stdin.Fd ())
syscall.Dup2 (fd, os.Stdout.Fd ())
syscall.Dup2 (fd, os.Stderr.Fd ())
}
}

return 0
}

위에서는 Go에서 구현하는 두 가지 daemon 솔루션을 소개했습니다. 그러나 이와 같이 여러분이 구현하는 것은 역시 추천하지 않습니다. 왜냐하면 공식은 아직 공식적으로 daemon의 지원이 선언되어 있지 않기 때문입니다. 당연히, 하나 째 솔루션은 지금까지 여전히 좋게 보이지만, 실제로 현재 오픈 소스 저장소 skynet에서는이 방법에 의해 daemon을 채용하고 있습니다.

## Supervisord
위에서는 Go 현재 두 종류의 솔루션 daemon을 구현하고있는 것을 소개했습니다. 그러나 공식은 아직 지원하지 않으므로 여러분에 있어서는 타사 성숙한 도구를 사용하여 우리의 응용 프로그램을 관리 할 것을 제안합니다. supervisord 당신이 관리하는 애플리케이션 프로그램을 daemon 프로그램하는 것을 돕고 명령을 통해 쉽게 시작, 정지, 재시작 등의 작업을 수행 할 수 있습니다. 또한 관리되는 프로세스가 일단 붕괴하면 자동으로 다시 시작하기 때문에 프로그램이 실행되는 동안 중단 한 경우의자가 치유 기능을 보장 할 수 있습니다.

> 나는 전에 응용 프로그램에서 지뢰를 밟은 수 있습니다. 모든 응용 프로그램이 Supervisord 부모 프로세스에서 생성되어 있기 때문에 운영 체제의 파일 디스크립터를 수정 한 때에는 잊지 말고 Supervisord를 다시 시작하십시오. 아래의 응용 프로그램을 다시 시작하는 것만으로는 안됩니다. 원래 나는 OS를 설치하면 우선 Supervisord을 설치하고 프로그램의 배포를 실시해, 파일 디스크립터를 수정하고 프로그램을 다시 시작했습니다. 파일 디스크립터 따위 100000 개나있을 것이라고 믿고있었습니다. 사실 Supervisord는 이때 기본 1024 개 밖에 준비되어 있지 않았습니다. 결과 관리되고 있던 프로그램을 포함하는 파일 디스크립터도 총 1024 개 밖에없고, 개방 한 순간 압력이 단번에 부풀어 올라 OS가 파일 디스크립터를 다 사용한 것으로 에러를 뱉어 시작했습니다. 오랜 시간에 걸쳐 겨우이 지뢰를 발견했습니다.

### Supervisord 설치
Supervisord는`sudo easy_install supervisor`에 설치 할 수 있습니다. 당연히 Supervisord의 공식 사이트에서 다운로드하여 압축 소스 코드가있는 디렉토리에서`setup.py install`을 실행하여 설치할 수도 있습니다.

- easy_install을 사용하는 경우 반드시 setuptools를 설치해야합니다

`http : // pypi.python.org / pypi / setuptools # files`를 엽니 다. 당신의 시스템은 python 버전에 따라 해당 파일을 다운로드하고`sh setuptoolsxxxx.egg`을 실행합니다. 이는 easy_install 명령 Supervisord를 설치할 수 있습니다.

### Supervisord 설정
Supervisord의 기본 설정 파일의 경로는 /etc/supervisord.conf입니다. 텍스트 편집기를 사용하여이 파일을 수정합니다. 다음은 설정 파일의 예입니다 :

; /etc/supervisord.conf
[unix_http_server]
file = /var/run/supervisord.sock
chmod = 0777
chown = root : root

[inet_http_server]
# Web 관리 인터페이스 설정
port = 9001
username = admin
password = yourpassword

[supervisorctl]
; 반드시 'unix_http_server'의 설정과 일치해야합니다.
serverurl = unix : ///var/run/supervisord.sock

[supervisord]
logfile = / var / log / supervisord / supervisord.log; (main log file; de​​fault $ CWD / supervisord.log)
logfile_maxbytes = 50MB; (max main logfile bytes b4 rotation; default 50MB)
logfile_backups = 10; (num of main logfile rotation backups; default 10)
loglevel = info; (log level; default info; others : debug, warn, trace)
pidfile = / var / run / supervisord.pid; (supervisord pidfile; de​​fault supervisord.pid)
nodaemon = true; (start in foreground if true; de​​fault false)
minfds = 1024; (min. avail startup file descriptors; default 1024)
minprocs = 200; (min. avail process descriptors; default 200)
user = root; (default is current user, required if root)
childlogdir = / var / log / supervisord /; ( 'AUTO'child log dir, default $ TEMP)

[rpcinterface : supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface : make_main_rpcinterface

; 관리하는 단일 프로세스 설정. 여러 program을 추가 할 수 있습니다.
[program : blogdemon]
command = / data / blog / blogdemon
autostart = true
startsecs = 5
user = root
redirect_stderr = true
stdout_logfile = /var/log/supervisord/blogdemon.log

### Supervisord 관리
Supervisord를 설치하면 supervisor와 supervisorctl 두 가지 명령을 사용할 수있게됩니다. 다음은 명령의 설명합니다 :

- supervisord, Supervisord를 초기화하고 시작합니다. 구성에서 설정된 프로세스를 시작 관리합니다.
- supervisorctl stop programxxx 프로세스 (programxxx)를 중지합니다. programxxx은 [program : blogdemon]에서 설정된 값입니다. 이 예에서는 blogdemon됩니다.
- supervisorctl start programxxx 프로세스를 시작합니다.
- supervisorctl restart programxxx 프로세스를 다시 시작합니다.
- supervisorctl stop all 모든 프로세스를 중지합니다. 참고 : start, restart, stop은 최신 설정 파일을로드하지 않습니다.
- supervisorctl reload 최신 설정 파일을 읽어 들여, 새로운 설정에 따라 모든 프로세스를 시작 관리합니다.

## 정리
이 절에서는 Go가 어떻게 daemon 화를 실현하고 있는지에 대해 소개했습니다. 단지 현재 Go의 daemon 구현은 부족하고 타사 도구를 사용하여 응용 프로그램 프로그램 daemon 관리하는 방법에 의존해야합니다. 따라서 여기에서는 python으로 작성된 프로세스 관리 도구 Supervisord을 소개했습니다. Supervisord를 사용하여 쉽게 Go 응용 프로그램을 관리 할 수​​ 있습니다.


