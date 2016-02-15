# Error Handling

Go 언어의 주요 설계 방침은 : 간결하고 명료합니다. 간결은 문법은 C와 비슷하여 매우 간단하다는 것입니다. 명료은 어떤 키워드도 알기 쉽다는 것을 의미합니다. 어떤 숨겨진 의미도 포함하고, 오류 처리 설계에서도이 사상은 일관하고 있습니다. C에서 -1 또는 NULL을 같은 정보를 반환하여 오류를 나타내고있는 것을 알고있을 것입니다. 그러나 사용자로부터하면 해당 API의 설명 문서를 보지 않으면,이 반환 값이 도대체 무슨 뜻을 의미하는지 원래 잘 모릅니다. 예를 들면 : 0을 반환 성공할지 실패할지 등입니다. Go는 error라는 형식을 정의하는 것으로, 오류를 나타냅니다. 사용할 때 반환되는 error 변수와 nil을 비교하여 조작이 성공했는지 확인합니다. 예를 들어`os.Open` 함수는 파일을 여는 데 실패했을 때 nil이 아닌 error 변수를 반환합니다.

func Open (name string) (file * File, err error)

아래의 예는`os.Open`을 사용하여 파일을 하나 오픈합니다. 만약 오류가 발생하면`log.Fatal`에 호출하여 오류 정보를 출력 할 수 있습니다 :

f, err : = os.Open ( "filename.ext")
if err! = nil {
log.Fatal (err)
}

`os.Open` 함수와 유사 표준 패키지의 모든 오류를 발생시킬 수있는 API는 아무도 error 변수를 반환하기 때문에 쉽게 오류 처리를 할 수 있습니다. 이 절에서는 error 형 설계를 상세하게 소개하고, Web 어플리케이션의 개발에있어서 더 나은 error 처리에 대해 논의합니다.
## Error 유형
error 형은 인터페이스 타입 중 하나입니다. 정의 :

type error interface {
Error () string
}

error는 내장 인터페이스 타입 중 하나입니다. / builtin / 패키지 아래에 해당하는 정의를 찾을 수 있습니다. 많은 내부 패키지에서 사용되는 error는 errors 패키지 이하로 구현 된 개인 구조체 errorString입니다.

// errorString is a trivial implementation of error.
type errorString struct {
s string
}

func (e * errorString) Error () string {
return e.s
}

`errors.New` 통해 문자열을 errorString로 변환하여 인터페이스 error를 충족 객체를 얻을 수 있습니다. 내부 구현은 다음과 같다 :

// New returns an error that formats as the given text.
func New (text string) error {
return & errorString {text}
}

다음 예제에서는 어떻게`errors.New`를 사용하거나 데모를 수행합니다.

func Sqrt (f float64) (float64, error) {
if f <0 {
return 0, errors.New ( "math : square root of negative number")
}
// implementation
}

다음의 예에서는 Sqrt를 호출했을 때 음수를 전달 non-nil 인 error 객체를 취득하고 있습니다. 이 개체를 nil과 비교하여 결과로 true를 얻기에 fmt.Println (fmt 패키지는 error를 처리 할 때 Error 메소드를 호출합니다)가 호출되어 오류를 출력합니다. 아래의 전화 코드 예제를 참조하십시오 :

f, err : = Sqrt (-1)
    if err! = nil {
        fmt.Println (err)
    }

## 사용자 정의 Error
위의 소개로 우리는 error가 단순한 interface라고 알았습니다. 따라서 자신의 패키지를 구현할 때이 인터페이스를 구현하는 구조를 정의하는 것으로, 자신의 오류 정의를 구현할 수 있습니다. Json 패키지의 예를 참조하십시오 :

type SyntaxError struct {
msg string // 오류 설명
Offset int64 // 에러가 발생한 장소
}

func (e * SyntaxError) Error () string {return e.msg}

Offset 필드는 Error를 호출 할 때 출력되지 않습니다. 그러나 형식 주장을 통해 오류 유형을 얻을 수 있기 때문에 해당 오류 정보를 출력 할 수 있습니다. 아래의 예를 참조하십시오 :

if err : = dec.Decode (& val); err! = nil {
if serr, ok : = err (* json.SyntaxError); ok {
line, col : = findLine (f, serr.Offset)
return fmt.Errorf ( "% s : % d : % d : % v"f.Name (), line, col, err)
}
return err
}

함수가 정의 정의 오류를 반환 할 때 반환 값에 error 형을 설정하도록 권장합니다. 특히 미리 선언 해 둘 필요가없는 오류 변수에는주의가 필요합니다. 예를 들면 :

func Decode () * SyntaxError {// 오류 위의 레이어로 호출 한 이용자에 의한 err! = nil의 판단이 영원히 true가됩니다.
        var err * SyntaxError // 미리 오류 변수를 선언합니다
        if 오류 조건 {
            err = & SyntaxError {}
        }
        return err // 오류 err는 영원히 nil이 아닌 값과 같습니다 위의 계층에서 호출 한 이용자에 의한 err! = nil의 판단이 항상 true가됩니다
    }

원인은 여기 http://golang.org/doc/faq#nil_error

위의 예에 의한 간단한 데모 Error 형식을 어떻게 스스로 정의하거나 보여했습니다. 그러나 만약 더 복잡한 오류 처리를 필요로하는 경우는 어떻게해야할까요 있습니까? net 패키지가 채용하고있는 방법을 추천합니다 :

package net

type Error interface {
error
Timeout () bool // Is the error a timeout?
Temporary () bool // Is the error temporary?
}

호출되는 장소에서는 err가 net.Error 여부 형 주장에 따라 판단하여 오류 처리를 세분화하고 있습니다. 예를 들어 아래의 예에서 만약 네트워크에 일시적으로 오류가 발생하는 경우 sleep 1 초를 행하고 다시 시도합니다 :

if nerr, ok : = err (net.Error); ok && nerr.Temporary () {
time.Sleep (1e9)
continue
}
if err! = nil {
log.Fatal (err)
}

## 오류 처리
Go는 오류 처리에서 C를 닮은 반환 값을 검사하는 방법을 채택하고 있으며, 기타 대다수의 주류 언어를 사용하는 예외 방식으로하지 않습니다. 이것은 코드를 작성하는데있어서 매우 큰 단점 중 하나입니다. 오류 처리 코드의 중복은 이러한 상황에서 우리가 검사 함수를 재사용함으로써 유사한 코드를 줄일 수 있습니다.

이 코드 예제를 참조하십시오 :

func init () {
http.HandleFunc ( "/ view"viewRecord)
}

func viewRecord (w http.ResponseWriter, r * http.Request) {
c : = appengine.NewContext (r)
key : = datastore.NewKey (c "Record"r.FormValue ( "id"), 0, nil)
record : = new (Record)
if err : = datastore.Get (c, key, record); err! = nil {
http.Error (w, err.Error (), 500)
return
}
if err : = viewTemplate.Execute (w, record); err! = nil {
http.Error (w, err.Error (), 500)
}
}

위의 예에서 데이터 검색 및 템플릿 배포를 호출 할 때 오류 검사를 실시하고 있습니다. 오류가 발생한 경우 일반적인 처리 함수 인`http.Error`를 호출하여 클라이언트에 500 오류를 반환하고 해당 오류 데이터를 표시합니다. 그러나 HnadleFunc이 추가됨에 따라 이러한 오류 처리 로직 코드가 많아집니다. 사실 우리는 스스로 정의한 라우터를 사용하여 코드를 단축시킬 수 있습니다 (구현 방식은 제 3 장의 HTTP의 자세한 설명을 참고하세요).

type appHandler func (http.ResponseWriter * http.Request) error

func (fn appHandler) ServeHTTP (w http.ResponseWriter, r * http.Request) {
if err : = fn (w, r); err! = nil {
http.Error (w, err.Error (), 500)
}
}

위에서는 스스로 라우터를 정의하고 있습니다. 다음의 방법으로 함수를 등록 할 수 있습니다 :

func init () {
http.Handle ( "/ view"appHandler (viewRecord))
}

/ view를 요청했을 때, 우리의 논리 처리는 다음과 같은 코드로 바뀝니다. 처음 구현 방법과 비교하면 상당히 쉽게되어 있습니다.

func viewRecord (w http.ResponseWriter, r * http.Request) error {
c : = appengine.NewContext (r)
key : = datastore.NewKey (c "Record"r.FormValue ( "id"), 0, nil)
record : = new (Record)
if err : = datastore.Get (c, key, record); err! = nil {
return err
}
return viewTemplate.Execute (w, record)
}

위의 예에서 오류 처리를 한 경우 모든 오류는 사용자에게 500 오류로 돌아갑니다. 그 후 해당 오류 코드를 출력합니다. 이 오류 정보의 정의는 더 사용성을 높일 수 있습니다. 디버깅 할 때 문제의 지점을 결정할 때 유용합니다. 자신 개구리 오류 유형을 정의 할 수 있습니다 :

type appError struct {
Error error
Message string
Code int
}

스스로 정의한 라우터를 다음과 같은 방법으로 변경합니다 :

type appHandler func (http.ResponseWriter * http.Request) * appError

func (fn appHandler) ServeHTTP (w http.ResponseWriter, r * http.Request) {
if e : = fn (w, r); e! = nil {// e is * appError, not os.Error.
c : = appengine.NewContext (r)
c.Errorf ( "% v"e.Error)
http.Error (w, e.Message, e.Code)
}
}

이렇게 스스로 정의 오류를 수정 한 후 논리는 다음과 같은 방법으로 수정할 수 있습니다 :

func viewRecord (w http.ResponseWriter, r * http.Request) * appError {
c : = appengine.NewContext (r)
key : = datastore.NewKey (c "Record"r.FormValue ( "id"), 0, nil)
record : = new (Record)
if err : = datastore.Get (c, key, record); err! = nil {
return & appError {err "Record n​​ot found"404}
}
if err : = viewTemplate.Execute (w, record); err! = nil {
return & appError {err "Can not display record"500}
}
return nil
}

위에서 볼 수 있듯이 view에 액세스 할 때 다른 상황에 따라 다른 오류 코드와 오류 정보를 얻을 수 있습니다. 이것은 처음 버전에 비해 코드의 양에 그다지 변화가 없지만, 이것이 표시하는 오류는 더 자세히 파악할 수 있습니다. 제공된 오류 정보의 유용성이 증가되고 확장 성을 처음에 비해 잘되어 있습니다.

## 정리
프로그램 설계에 오류 허용 중요한 일의 일부입니다. Go는 오류 처리를 통해이를 실현합니다. error는 하나의 인터페이스에 불과하지만 많은 변화시킬 수 있습니다. 자신의 수요에 따라 다른 처리를 구현할 수 있습니다. 마지막으로 소개 한 오류 처리 방법으로 여러분에게보다 나은 Web 오류의 처리 방법을 설계 할 때 조력이되면 다행입니다.

