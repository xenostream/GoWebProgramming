# Design Controllers

전통적인 MVC 프레임 워크에서 종종 선행 액션(PostFix Action) 설계에 기초하고 있습니다.   
그러나, 현재는 REST 스타일의 프레임워크가 유행되고 있습니다. 이것은 가급적 `Filter` 또는 `rewrite`를     
사용하여 URL을  재작성하고, REST 스타일의 URL을 구현하고 있습니다.       
그렇다면 왜 직접 새로운 REST 스타일의 MVC 프레임워크를 설계하지 않는 것일까요? 이 장에서는 이런 개념을   
기반으로 REST 스타일의 MVC 프레임워크에 풀 스크래치 컨트롤러(Full Scratch Controller), 단순화 된 Web   
어플리케이션의 개발, 나아가서는 한 줄로 출력 가능한 "Hello, world"의 구현 방법에 대해 설명 합니다.

## controller의 작용
MVC 디자인은 현재 Web 어플리케이션의 개발에서 가장 흔히 볼 수 있는 프레임워크 설계 방식 중 하나 입니다.     
Model(모델), View(뷰) 및 Controller(컨트롤러)를 분리하여 확장 가능한 사용자 인터페이스(UI)를 쉽게 구현할     
수 있습니다. `Model`은 백엔드(주로 데이터베이스)가 반환하는 데이터를 의미합니다. `View`는 화면에 표시되는   
페이지로써, 일반적으로 템플릿 페이지가 있습니다. 템플릿을 적용한 컨텐츠는 일반적으로 HTML 입니다.   
`Controller`는 Web 개발자가 주로 코딩하는 것으로, URL 처리를 주로 담담하고 있습니다.  
이전 절에서는 우리는 URL 요청을 컨트롤러에서  리디렉션하는 과정을 처리하는 라우터를 소개했습니다. controller는    
MVC 프레임워크 중 가장 핵심적인 작용을 하는 모듈 입니다. 서비스 로직 처리를 담당하는 컨트롤러는 프레임 워크에   
필수적인 존재 입니다. Model과 View는 서비스에 따라 쓸 필요가 없을 수 도 있습니다.    
예를들어 데이터를  처리할 필요가 없는 로직(302 리디렉션) 같은 경우는 Model과 View를 필요로하지 않습니다.     
이경우에도 역시 Controller는 반드시 필요 합니다.

## beego의 REST 설계
앞 절에서는 라우터에 struct를 등록해서 처리하는 기능을 구현 했습니다. 또한 struct에서 REST 메소드를 구현하고   
있습니다. 따라서 로직 처리에 이용되는 controller의 기본 클래스를 설계해야 합니다.   

하나는 struct이고 다른 하나는 interface 입니다.
```
type Controller struct {
    Ct        *Context
    Tpl       *template.Template
    Data      map[interface{}]interface{}
    ChildName string
    TplNames  string
    Layout    []string
    TplExt    string
}

type ControllerInterface interface {
    Init(ct *Context, cn string) //Initialize the context and subclass name
    Prepare()                    //some processing before execution begins
    Get()                        //method = GET processing
    Post()                       //method = POST processing
    Delete()                     //method = DELETE processing
    Put()                        //method = PUT handling
    Head()                       //method = HEAD processing
    Patch()                      //method = PATCH treatment
    Options()                    //method = OPTIONS processing
    Finish()                     //executed after completion of treatment
    Render() error               //method executed after the corresponding method to render the page
}

```
전에 add 함수에서 라우터를 설명할 때 `ControllerInterface` 클래스를 정의 했습니다. 그래서 여기서는 이 인터페이스
를 구현하기만 해도 충분 합니다. 기본 클래스의 Contoroller의 구현은 다음과 같은 방법이 있습니다.
```
func (c *Controller) Init(ct *Context, cn string) {
    c.Data = make(map[interface{}]interface{})
    c.Layout = make([]string, 0)
    c.TplNames = ""
    c.ChildName = cn
    c.Ct = ct
    c.TplExt = "tpl"
}

func (c *Controller) Prepare() {

}

func (c *Controller) Finish() {

}

func (c *Controller) Get() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Post() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Delete() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Put() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Head() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Patch() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Options() {
    http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
}

func (c *Controller) Render() error {
    if len(c.Layout) > 0 {
        var filenames []string
        for _, file := range c.Layout {
            filenames = append(filenames, path.Join(ViewsPath, file))
        }
        t, err := template.ParseFiles(filenames...)
        if err != nil {
            Trace("template ParseFiles err:", err)
        }
        err = t.ExecuteTemplate(c.Ct.ResponseWriter, c.TplNames, c.Data)
        if err != nil {
            Trace("template Execute err:", err)
        }
    } else {
        if c.TplNames == "" {
            c.TplNames = c.ChildName + "/" + c.Ct.Request.Method + "." + c.TplExt
        }
        t, err := template.ParseFiles(path.Join(ViewsPath, c.TplNames))
        if err != nil {
            Trace("template ParseFiles err:", err)
        }
        err = t.Execute(c.Ct.ResponseWriter, c.Data)
        if err != nil {
            Trace("template Execute err:", err)
        }
    }
    return nil
}

func (c *Controller) Redirect(url string, code int) {
    c.Ct.Redirect(code, url)
}    

```
위 controller 기본 클래스는 인터페이스가  정의하는 함수를 실제로 구현하고 있습니다.   
url에 따라 라우터가 해당 controller를 실행하는 원칙에 따라서 다음과 같이 실행됩니다

- Init() 초기화
  Prepare() 함수가 초기화를 실행하여 상속 된 서브클래스는 이 함수를 구현할 수 있습니다.  
- method() 
  method에 따라 각가의 기능을 수행 합니다 : GET, POST, PUT, HEAD 등 하위 클래스에서이 함수를 구현 합니다.   
  만약 구현되어 있지 않으면 아무도 기본적으로 403 입니다.  
- Render() 옵션. 
  글로벌 변수 AutoRender 의해 실행 여부를 판단 합니다.  
- Finish() 
  실행 후에 실행되는 작업입니다. 각 서브 클래스는이 함수를 구현할 수 있습니다.  

## 응용
위에서는 beego 프레임워크에서 controller 기본 클래스 설계를 완성 했습니다.   
이 설계대로 응용 프로그램에서 다음과 같이 설계 할 수 있습니다.
```
package controllers

import (
    "github.com/astaxie/beego"
)

type MainController struct {
    beego.Controller
}

func (this *MainController) Get() {
    this.Data["Username"] = "astaxie"
    this.Data["Email"] = "astaxie@gmail.com"
    this.TplNames = "index.tpl"
}

```
위 메소드는 서브 클래스 MainController를 구현하고 Get 메소드를 구현하고 있습니다.    
만약 사용자가 다른 메소드(POST / HEAD 등)를 이용해서 이 리소스에 액세스 할 때 403을 반환 합니다.   
만약 Get 방식이면 AutoRender = true를 설정하고 있기 때문에 Get 메소드를 실행 한 후 자동으로 Render함수가   
실행되며 다음과 같은 인터페이스가 나타납니다 :

! [] (images / 13.4.beego.png? raw = true)

index.tpl의 코드는 다음과 같습니다.   
이러한 설계로 인해서 데이터 설정과 표시가 매우 단순해짐을 알수 있습니다 :
```
<!DOCTYPE html>
<html>
  <head>
    <title>beego welcome template</title>
  </head>
  <body>
    <h1>Hello, world!{{.Username}},{{.Email}}</h1>
  </body>
</html>

```

