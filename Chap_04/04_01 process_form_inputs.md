# Process form inputs

우선 양식에 의한 전송의 예를 살펴 보겠습니다. 다음과 같은 양식의 내용이 있다고 가정 합니다.    
login.gtpl 파일을 만듭니다. (새로운 디렉토리를 만들고 그 안에 넣어주세요)
``` html
<html>
<head>
<title> </title>
</head>
<body>
<form action = "/login"method = "post">
사용자 이름 : <input type = "text" name = "username">
암호 : <input type = "password" name = "password">
<input type = "submit" value = "로그인">
</form>
</body>
</html>
```
상기 페이지에서 입력후 전송되는 양식은 서버의 `/login`에 전달 됩니다. 사용자가 정보를 입력하고 로그인을 클릭 하면,    
서버의 `login`으로 그 입력 데이터를 리디렉션 합니다. 우선이 전송방법이 `POST/GET`인지를 판단해야 합니다.   

`http` 패키지에는 매우 쉽게 알아내는 방법이 있습니다. 이전장에서 설명한 예제를 기초로 login 페이지의 form 데이터를    
어​​떻게 처리하는지 살펴 보겠습니다.    

``` Go
package main

import (
    "fmt"
    "html/template"
    "log"
    "net/http"
    "strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()             // url의 전달 옵션을 분석 합니다. 
                              // POST방식일 경우 응답 패킷의 바디를 분석 합니다 (request body)
                              // 주의: ParseForm 메소드가 호출되지 않으면 양식 데이터를 검색 할 수 없습니다.
    fmt.Println(r.Form)       //이 데이터는 서버의 인쇄 정보에 출력됩니다
    fmt.Println("path"r.URL.Path)
    fmt.Println("scheme"r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key :", k)
        fmt.Println("val :", strings.Join(v, ""))
    }
    fmt.Fprintf(w, "Hello astaxie!") // 여기서 w에 기록 된 것이 클라이언트에 출력 됩니다.
}

func login(w http.ResponseWriter, r *http.Request) {
    fmt.Println("method :", r.Method)       // 요청 처리 방식 
    if r.Method == "GET" {
        t , _ := template.ParseFiles("login.gtpl")
        t.Execute(w, nil)
    } else {
        // 로그인 데이터가 요청되고, 로그인 논리 판단이 실행됩니다.
        fmt.Println("username :", r.Form["username"])
        fmt.Println("password :", r.Form["password")
    }
}

func main() {
    http.HandleFunc("/", sayhelloName)        // 액세스 라우팅을 설정합니다
    http.HandleFunc("/ login", login)         // 액세스 라우팅을 설정합니다
    err := http.ListenAndServe(": 9090", nil) // 감시하는 포트를 설정합니다
    if err != nil {
        log.Fatal("ListenAndServe :", err)
    }
}
```

위의 코드에서 요청 메소드를 얻으려면 `r.Method`만으로 모든 처리가 끝나는 것을 알 수 있습니다. 이것은 문자열 변수입니다.   
GET, POST, PUT 등의 요청 method 정보를 반환 합니다.   

login 함수에서 `r.Method` 따라 로그인 화면 표시와 로그인 로직을 처리할지를 결정 합니다.    
GET 메소드에 의한 요청일 경우 로그인 화면을 표시하고,  다른 방법에 의한 요청은 로그인 로직을 처리 합니다.    
예를 들어 데이터베이스를 검색하거나 로그인 정보를 확인하는 등의 일 입니다.   

브라우저에서 `http:/127.0.0.1:9090/login`을 입력하면 다음과 같은 화면이 나타납니다.  

![](images/4.1.login.png)   
그림 4.1 사용자 로그인 화면

사용자 이름과 암호를 입력해도 서버는 아무것도 출력하지 않습니다. 왜 일까요? 기본적으로는 Handler에서는 form의 내용을    
자동으로 분석하지 않기 때문 입니다. 반드시 명시적으로 `r.ParseForm()`를 호출하지 않는다면, 아무것도 분석할 수 없습니다.    
코드를 약간 수정하여 `fmt.Println("username :", r.Form[ "username"])` 앞에 `r.ParseForm()`라는 줄을 추가 하십시오.  
다시 컴파일한 후 입력후 로그인하면 이번에는 서버가 사용자가 입력 한,  사용자 이름과 암호를 출력하게 됩니다.    

`r.Form`에는 모든 요청의 데이터가 포함되어 있습니다. 예를 들어 URL 중 query-string, POST 데이터, PUT 데이터 등 입니다.    
URL의 query-string 필드 및 POST가 충돌하는 경우 slice에 저장 됩니다. 여기에는 여러 값이 저장되어 있습니다.    
Go의 공식 문서에따르면, 다음 버전에서 POST, GET 등의 데이터를 분리해서 저장한다고 말합니다.

이제는 login.gtpl의 form의 action 값인 `http:/127.0.0.1:9090/login`을 `http:/127.0.0.1:9090/login?username=astaxie`로   
변경한 후 다시 시도하면, 서버의 출력은 다음과 같습니다.

![](images/4.1.slice.png)   
그림 4.1 서버가 수신 한 데이터를 표시   

`request.Form`은 url.Values​​ 형 입니다.`key = value`와  같은 해당 데이터가 저장되어 있습니다.    
form 데이터에 대한 몇 가지 예제를 들겠습니다.    
```
v := url.Values{}
v.Set("name", "Ava")
v.Add("friend", "Jess")
v.Add("friend", "Sarah")
v.Add("friend", "Zoe")
// v.Encode () == "name = Ava & friend = Jess & friend = Sarah & friend = Zoe"
fmt.Println(v.Get("name"))
fmt.Println(v.Get("friend"))
fmt.Println(v["friend"])
```
> ** Tips ** 
Request의 FormValue() 함수에서도 사용자가 전송 한 데이터를 얻을 수 있습니다. 예를 들어 r.Form["username"]은    
r.FormValue("username")로 쓸 수도  있습니다. r.FormValue를 호출할때 자동으로 r.ParseForm가 호출되므로 미리 호출 할 필요   
는 없습니다. r.FormValue는 같은 데이터 중에서 첫번째 것만 반환 합니다. 만약 데이터가 존재하지 않는 경우는 `빈 문자열`을 
반환합니다.   

