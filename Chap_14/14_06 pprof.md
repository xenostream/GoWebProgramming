# pprof

Go 언어의 아주 좋은 디자인은 표준 라이브러리 코드의 성능 모니터링 도구를 갖는 것입니다. 2 개의 패키지가 있습니다 :

net / http / pprof

runtime / pprof

사실 net / http / pprof는 runtime / pprof 패키지를 사용하여 랩하고있을뿐, http 포트에서 나타나는뿐입니다.

## beego은 pprof를 지원하고 있습니다
현재 beego 프레임 워크는 pprof를 추가하고 있습니다. 이 기능은 기본적으로 시작되지 않습니다. 만약 성능을 테스트하거나 해당 goroutine의 실행 등의 정보를 확인 할 필요가 있으면, Go 기본 패키지 "/ net / http / pprof"는 이미이 기능이 있습니다. 예를 들면 Go의 기본 방법으로 Web을 실행하면 기본적으로 사용할 수 있습니다. 그러나 beego는 ServHTTP 함수를 래핑하고 있기 때문에, 만약 기본 패키지를 그대로 사용하고 있다면이 기능을 활성화 할 수 없습니다. 따라서 beego 내부 개조를 실시, pprof를 지원해야합니다.

- 우선 beego.Run 함수에서 변수에 의해 자동으로 작동 패키지를로드할지 여부를 결정합니다.

if PprofOn {
BeeApp.RegisterController (`/ debug / pprof` & ProfController {})
BeeApp.RegisterController (`/ debug / pprof / : pp (\ w +)`& ProfController {})
}

- ProfConterller를 설계하는

package beego

import (
"net / http / pprof"
)

type ProfController struct {
Controller
}

func (this * ProfController) Get () {
switch this.Ctx.Params [ ": pp"] {
default :
pprof.Index (this.Ctx.ResponseWriter, this.Ctx.Request)
case "":
pprof.Index (this.Ctx.ResponseWriter, this.Ctx.Request)
case "cmdline":
pprof.Cmdline (this.Ctx.ResponseWriter, this.Ctx.Request)
case "profile":
pprof.Profile (this.Ctx.ResponseWriter, this.Ctx.Request)
case "symbol":
pprof.Symbol (this.Ctx.ResponseWriter, this.Ctx.Request)
}
this.Ctx.ResponseWriter.WriteHeader (200)
}


## 사용 방법

위의 설계를 통해 다음과 같은 코드에 의해 pprof를 시작할 수 있습니다 :

beego.PprofOn = true

그런 다음 브라우저에서 다음 URL을 열면 다음과 같은 인터페이스가 나타납니다 :
![](14.6.pprof.png)   
! [] (images / 14.6.pprof.png? raw = true)

그림 14.7 시스템의 현재의 goroutine, heap, thread 정보

goroutine을 클릭하면 자세한 정보를 얻을 수 있습니다 :

![](14.6.pprof2.png)
   
! [] (images / 14.6.pprof2.png? raw = true)

그림 14.8 현재의 goroutine 정보를 표시

명령 줄에서 더 많은 자세한 정보를 얻을 수 있습니다

go tool pprof http : // localhost : 8080 / debug / pprof / profile

이때 프로그램은 30 초 profile 수집 시간에 들어갑니다. 이 시간에 필사적으로 브라우저에서 페이지를 다시로드하고 가능한 cpu를 소유하고 데이터를 생성합니다.

(pprof) top10

Total : 3 samples

       1 33.3 % 33.3 % 1 33.3 % MHeap_AllocLocked

       1 33.3 % 66.7 % 1 33.3 % os / exec (* Cmd) .closeDescriptors

       1 33.3 % 100.0 % 1 33.3 % runtime.sigprocmask

       0 0.0 % 100.0 % 1 33.3 % MCentral_Grow

       0 0.0 % 100.0 % 2 66.7 % main.Compile

       0 0.0 % 100.0 % 2 66.7 % main.compile

       0 0.0 % 100.0 % 2 66.7 % main.run

       0 0.0 % 100.0 % 1 33.3 % makeslice1

       0 0.0 % 100.0 % 2 66.7 % net / http (* ServeMux) .ServeHTTP

       0 0.0 % 100.0 % 2 66.7 % net / http (* conn) .serve

(pprof) web

![](14.6.pprof3.png)

! [] (images / 14.6.pprof3.png? raw = true)

그림 14.9 데모 실행 프로세스 정보

