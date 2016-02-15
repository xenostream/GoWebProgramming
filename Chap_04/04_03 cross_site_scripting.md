# Cross Site Scripting

현재의 홈페이지에는 대량의 동적컨텐츠를 포함해서 사용자의 반응에 민감하게 처리할 수 있게 되었습니다.    
예전에 비해서 아주 복잡하게 처리되고 있습니다. 이러한 동적컨텐츠는 사용자의 환경과 요구에 따라서  Web 어플리케이션이   
원하는 내용을 출력 할 수 있도록 해줍니다.    

동적 홈페이지는 "크로스 사이트 스크립팅"(Cross Site Scripting,  보안 전문용어로 XSS)이라는 공격을 받을 수 있습니다.   
공격자는 일반적으로 보안에 헛점이 있는 JavaScript, VBScript, ActiveX 또는 Flash를 프로그램에 삽입해서, 사용자를 속이고,      
일단 이 공격이 성​​공한다면 계정이 도용 된 사용자의 설정을 변경하거나, cookie를 도용하거나 악의적인 광고를 포함하기도 합니다.  

XSS에 대한 가장 효과적인 방어법은 다음 두 가지를 결합하는 것입니다.
첫번째는 모든 입력 데이터를 검증하고 공격 여부를 검사​​를하는 것(앞장에서 일부를 설명했습니다.   
두번째는 출력되는 데이터에 대해서 적절한 처리를해서 이미 삽입되어있는 스크립트가 브라우저에서 실행되지 않도록하는 것입니다.   

Go에서는 어떻게 효과적으로 방어를 할 수 있을까요? Go언어의 `html/template` 패키지로 방어할 수 있습니다.    

- func HTMLEscape(w io.Writer, b []byte)       // b 배열을 탈출 처리하고 , w로 출력.
- func HTMLEscapeString(s string) string       // s 문자열을 탈출 처리하고, 결과 문자열로 반환.
- func HTMLEscaper(args ...interface{}) string // 여러 인자를 동시 탈출 처리하고, 결과문자열로 반환.


4.1절의 예제를 살펴보겠습니다.    
``` Go
fmt.Println("username :", template.HTMLEscapeString(r.Form.Get("username")))  // 서버 측에 출력
fmt.Println("password :", template.HTMLEscapeString(r.Form.Get( "password")))
template.HTMLEscape(w []byte (r.Form.Get("username")))                        // 클라이언트에 출력
```

만약 입력 된 username이 `<script>alert()</script>`일 경우에는, 브라우저에 다음과 같이 표시됩니다.

![](images/4.3.escape.png)   
그림 4.3 Javascript 필터의 출력   

Go의 `html/template` 패키지는 기본적으로 html 태그를 필터링 합니다. 그러나 때로는 `<script>alert()</script>`의 경우처럼  
정상적인 정보를 출력하고 싶은 경우도 있습니다.이런 경우 text/template를 이용하면 됩니다. 아래의 예제를 참고 하십시오.  

``` Go
import "text/template"
...

t, err := template.New("foo").Parse(`{{define "T"}} Hello, {{}}! {{end}}`)
err = t.ExecuteTemplate(out "T", "<script>alert('you have been pwned')</script>")
```

다음과 같이 출력됩니다. 

`Hello <script>alert('you have been pwned')</script>!`

또는 template.HTML 형을 사용해서 처리한다면, 다음과 같습니다. 
``` Go
import "html/template"
...
t, err := template.New("foo").Parse(`{{define "T"}} Hello, {{}}! {{end}}`)
err = t.ExecuteTemplate(out "T", template.HTML("<script>alert('you have been pwned')</script>"))
```
출력은 다음과 같습니다. 

`Hello <script>alert('you have been pwned')</script>!`

`template.HTML`형으로 변환 한 후 변수의 내용은 처리하지 않습니다.(Escape)

탈출의 예 :
``` Go
impo해rt "html/template"
...
t, err := template.New("foo").Parse(`{{define "T"}} Hello, {{}}! {{end}}`)
err = t.ExecuteTemplate(out "T", "<script>alert('you have been pwned')</script>")
```
탈출 후 출력 

`Hello,&lt;script&gt;alert(&#39;you have been pwned&#39;)&lt;/script&gt;!`
