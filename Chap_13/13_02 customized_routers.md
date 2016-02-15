# Customized Routers

## HTTP 라우팅
HTTP 라우팅 설정은 HTTP 요청에 해당하는  함수처리를 담당 합니다. 예를들어 앞 절에서 소개 한 라우팅 프레임워크에서  
이벤트 프로세서에 해당 합니다. 이 이벤트는 다음과 같은 것을 포함 합니다.

- 사용자의  요청 경로(path) (예: /user/123/article/123), 당연히 문자열 정보를 검색 합니다 (예를들면 ?id=11)  
- HTTP 요청 처리 메소드방식(method) (GET, POST, PUT, DELETE PATCH 등)   

라우터는 사용자가 요청한 이벤트 정보에 따라 대응하는 처리함수(컨트롤 레이어)로 리디렉션 합니다.

## 기본 라우팅 구현
3.4 절에서 Go의 `http`패키지를 통해서 Go언어에서 라우팅이 어떻게 설계되고 라우팅을 구현하는지를 설명했습니다.  
여기에 덧붙여 또 하나 예를 들어 설명하도록 하겠습니다. 
```
func fooHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
}

http.Handle("/foo", fooHandler)

http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

log.Fatal(http.ListenAndServe(":8080", nil))
```

상기의 코드는 http 패키지의 `DefaultServeMux`를 호출하여 라우팅을 처리하고 있습니다.   
2개의 인수를 전달하여 사용자가 액세스하는 리소스를 제공 합니다. 첫 번째 인수는 사용자가 리소스에 접근할  URL경로  
(`r.URL.Path`에 저장)이며, 두 번째 인수는 이 경로에 접근할 때 실행되는 함수에 대한 설정입니다.   

라우팅의 개념은 다음의 두 가지로 요약 됩니다.

- 라우팅 정보 추가
- 사용자의 요청을 처리하는  함수로 리디렉션

Go의 기본 라우팅은 `http.Handle`과 `http.HandleFunc` 함수에 의해서 처리되며, 이 것은 실제적으로         
`DefaultServeMux.Handle(pattern string, handler Handler)`를 호출 합니다. 이 함수는 라우팅 정보를 map 정보  
`map[string] muxEntity`에 저장하여 라우팅 처리를 하게 됩니다. 이것으로 첫 번째 개념을 구현 합니다. 

Go는 해당 포트를 모니터링하고 tcp 접속을 받아들이고 해당 Handler에게 처리를 위임합니다.      
위의 예제에서는 기본 처리는 `http.DefaultServeMux` 입니다. 이것은 `DefaultServeMux.ServeHTTP` 함수에 의해   
전송 됩니다. 이미 저장한  map라우팅 정보를 검색하여 사용자가 접근하려는 URL에 등록 된 처리함수를 찾습니다.    
따라서 이것은 두번째 개념을 구현 합니다.

```
for k, v := range mux.m {
    if !pathMatch(k, path) {
        continue
    }
    if h == nil || len(k) > n {
        n = len(k)
        h = v.h
    }
}
```

## beego 프레임워크의 라우팅 구현
현재 거의 모든 Web 어플리케이션의 모든 라우팅처리는 `http`의 기본 라우터 처리방식으로 구현되어 있습니다.   
그러나 Go에서 제공하는 라우터에는 몇 가지 제한이 있습니다.

- 파라미터 설정을 지원하지 않습니다. 예를 들어 /user/:uid 등의  매칭을 지원하지 않습니다.    
- REST 모드에 대한 지원이 미약합니다. 접근을 제한하는 방법이 없습니다.    
  위 예제로  표현하면, 사용자가 /foo URL에 GET, POST, DELETE HEAD 같은 방법으로 액세스 할 수 있습니다.
- 일반적으로 웹 사이트의 라우팅 규칙은 너무 많아서 사용하기가 힘듭니다.   
  일례로, API 응용 프로그램을 개발 할 경우 사용했던 라우팅 규칙은 30개 정도 였습니다.     
  이런  많은 라우팅 규칙들은 사실 단순화시킬 수 있습니다. struct 처리 방법을 통해 간소화 할 수 있습니다.

beego 프레임워크의 라우터는 위의 몇 가지 제한을 고려하여 설계된 REST메소드의 라우팅을 구현하고 있습니다.    
라우팅 디자인도 Go언어의 기본 설계에 기반하여 작성되어 있습니다. 즉, 라우팅 저장 및 라우팅 리디렉션 입니다.

### 라우팅 저장
상기에서 설명한 제한 중에서 첫번째 제한인 매개변수 지원에는  정규표현식을 사용해서 처리해야 합니다.  
2 번째와 3 번째 제한은 유연한 방법으로 해결 합니다. REST 방식을 struct을 이용한 처리방법에  포함시켜 처리 합니다.  
해당 처리함수가 아닌 struct로 라우팅하여 라우팅을 리디렉션 할 때,  method에 따라 다른 방법을 처리  할 수 있습니다.

위의 설명대로 2개의 데이터형 `controllerInfo`(경로와 해당 처리 struct를 저장한다. 여기에서는 reflect.Type 형)과   
`ControllerRegistor`(routers는 slice를 사용하여 사용자가 추가 한 라우팅 정보를 저장)를 설계 했습니다.

```
type controllerInfo struct {
    regex          *regexp.Regexp
    params         map[int]string
    controllerType reflect.Type
}

type ControllerRegistor struct {
    routers     []*controllerInfo
    Application *App
}

```

ControllerRegistor의 외부 인터페이스 함수는 다음과 같습니다.
```
func (p * ControllerRegistor) Add (pattern string, c ControllerInterface)
```

세부 사항은 다음과 같습니다
```
func (p *ControllerRegistor) Add(pattern string, c ControllerInterface) {
    parts := strings.Split(pattern, "/")

    j := 0
    params := make(map[int]string)
    for i, part := range parts {
        if strings.HasPrefix(part, ":") {
            expr := "([^/]+)"

            //a user may choose to override the defult expression
            // similar to expressjs: ‘/user/:id([0-9]+)’

            if index := strings.Index(part, "("); index != -1 {
                expr = part[index:]
                part = part[:index]
            }
            params[j] = part
            parts[i] = expr
            j++
        }
    }

    //recreate the url pattern, with parameters replaced
    //by regular expressions. then compile the regex

    pattern = strings.Join(parts, "/")
    regex, regexErr := regexp.Compile(pattern)
    if regexErr != nil {

        //TODO add error handling here to avoid panic
        panic(regexErr)
        return
    }

    //now create the Route
    t := reflect.Indirect(reflect.ValueOf(c)).Type()
    route := &controllerInfo{}
    route.regex = regex
    route.params = params
    route.controllerType = t

    p.routers = append(p.routers, route)

}
```

### 정적 라우팅 구현
위에서는 동적 라우팅 구현을 했습니다. Go의 `http` 패키지는 기본적으로 정적 파일을 처리하는 `FileServer`를   
제공하고 있습니다. 여기서는 직접 정의한 라우터를 구현했으므로, 정적 파일도 직접 설정해야 합니다.   
beego 정적 디렉토리 경로는 전역 변수 `StaticDir`에 저장되어 있습니다. 
StaticDir는 map형태로 다음과 같이 구현되어 있습니다.
```
func (app *App) SetStaticPath(url string, path string) *App {
    StaticDir[url] = path
    return app
}
```
응용 프로그램에서 정적 라우팅을 설정하려면 다음과 같이합니다.
```
beego.SetStaticPath("/img", "/static/img")
```

### 리디렉션 라우팅
라우팅은 `ControllerRegistor` 라우팅 정보에 따라 리디렉션이 발생하게 됩니다.    
이것을 구현한 코드는 다음과 같습니다.
```
// AutoRoute
func (p *ControllerRegistor) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    defer func() {
        if err := recover(); err != nil {
            if !RecoverPanic {
                // go back to panic
                panic(err)
            } else {
                Critical("Handler crashed with error", err)
                for i := 1; ; i += 1 {
                    _, file, line, ok := runtime.Caller(i)
                    if !ok {
                        break
                    }
                    Critical(file, line)
                }
            }
        }
    }()
    var started bool
    for prefix, staticDir := range StaticDir {
        if strings.HasPrefix(r.URL.Path, prefix) {
            file := staticDir + r.URL.Path[len(prefix):]
            http.ServeFile(w, r, file)
            started = true
            return
        }
    }
    requestPath := r.URL.Path

    //find a matching Route
    for _, route := range p.routers {

        //check if Route pattern matches url
        if !route.regex.MatchString(requestPath) {
            continue
        }

        //get submatches (params)
        matches := route.regex.FindStringSubmatch(requestPath)

        //double check that the Route matches the URL pattern.
        if len(matches[0]) != len(requestPath) {
            continue
        }

        params := make(map[string]string)
        if len(route.params) > 0 {
            //add url parameters to the query param map
            values := r.URL.Query()
            for i, match := range matches[1:] {
                values.Add(route.params[i], match)
                params[route.params[i]] = match
            }

            //reassemble query params and add to RawQuery
            r.URL.RawQuery = url.Values(values).Encode() + "&" + r.URL.RawQuery
            //r.URL.RawQuery = url.Values(values).Encode()
        }
        //Invoke the request handler
        vc := reflect.New(route.controllerType)
        init := vc.MethodByName("Init")
        in := make([]reflect.Value, 2)
        ct := &Context{ResponseWriter: w, Request: r, Params: params}
        in[0] = reflect.ValueOf(ct)
        in[1] = reflect.ValueOf(route.controllerType.Name())
        init.Call(in)
        in = make([]reflect.Value, 0)
        method := vc.MethodByName("Prepare")
        method.Call(in)
        if r.Method == "GET" {
            method = vc.MethodByName("Get")
            method.Call(in)
        } else if r.Method == "POST" {
            method = vc.MethodByName("Post")
            method.Call(in)
        } else if r.Method == "HEAD" {
            method = vc.MethodByName("Head")
            method.Call(in)
        } else if r.Method == "DELETE" {
            method = vc.MethodByName("Delete")
            method.Call(in)
        } else if r.Method == "PUT" {
            method = vc.MethodByName("Put")
            method.Call(in)
        } else if r.Method == "PATCH" {
            method = vc.MethodByName("Patch")
            method.Call(in)
        } else if r.Method == "OPTIONS" {
            method = vc.MethodByName("Options")
            method.Call(in)
        }
        if AutoRender {
            method = vc.MethodByName("Render")
            method.Call(in)
        }
        method = vc.MethodByName("Finish")
        method.Call(in)
        started = true
        break
    }

    //if no matches to url, throw a not found exception
    if started == false {
        http.NotFound(w, r)
    }
}

```
### Getting Start!
이러한 라우팅 설계를 기반으로 구현했다면 앞에서 설명한 세 가지 제한 사항을 해결 할 수 있습니다.   
사용법은 다음과 같습니다.

기본적인 라우팅 등록의 사용
```
beego.BeeApp.RegisterController("/", &controllers.MainController{})
```

옵션 등록
```
beego.BeeApp.RegisterController("/:param", &controllers.UserController{})
```

정규 표현 매칭
```
beego.BeeApp.RegisterController("/users/:uid([0-9]+)", &controllers.UserController{})
```
