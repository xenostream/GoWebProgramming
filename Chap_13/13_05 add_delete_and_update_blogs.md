# Add Delete and Update Blogs

앞에서는 beego 프레임워크의 전체적인 구조 및 구현과 개념에 대한 실제 구현에 대해 설명 했습니다. 
이 절에서는 beego 프레임워크를 사용해서 블로그 시스템을 설계하도록 하겠습니다.
여기에서는 블로그 검색, 추가, 수정, 삭제 등의 작업이 포함 됩니다.

## 블로그 디렉토리
블로그 디렉토리는 다음과 같습니다 :

```
/main.go
/views:
    /view.tpl
    /new.tpl
    /layout.tpl
    /index.tpl
    /edit.tpl
/models/model.go
/controllers:
    /index.go
    /view.go
    /new.go
    /delete.go
    /edit.go 
```

## 블로그 라우팅
블로그의 주요 라우팅 규칙은 다음과 같습니다 :
``` Go
//Show blog Home
beego.RegisterController("/", &controllers.IndexController{})
//View blog details
beego.RegisterController("/view/: id([0-9]+)", &controllers.ViewController{})
//Create blog Bowen
beego.RegisterController("/new", &controllers.NewController{})
//Delete Bowen
beego.RegisterController("/delete/: id([0-9]+)", &controllers.DeleteController{})
//Edit Bowen
beego.RegisterController("/edit/: id([0-9]+)", &controllers.EditController{})
```

## 데이터베이스 스키마
데이터베이스의 설계는 가장 간단한  블로그 정보입니다
``` SQL
CREATE TABLE entries (
    id INT AUTO_INCREMENT,
    title TEXT,
    content TEXT,
    created DATETIME,
    primary key (id)
);
```
## 컨트롤러
IndexController
``` Go
type IndexController struct {
    beego.Controller
}

func (this *IndexController) Get() {
    this.Data["blogs"] = models.GetAll()
    this.Layout = "layout.tpl"
    this.TplNames = "index.tpl"
}
```
ViewController
``` Go
type ViewController struct {
    beego.Controller
}

func (this *ViewController) Get() {
    inputs := this.Input()
    id, _ := strconv.Atoi(this.Ctx.Params[":id"])
    this.Data["Post"] = models.GetBlog(id)
    this.Layout = "layout.tpl"
    this.TplNames = "view.tpl"
}
```
NewController
``` Go
type NewController struct {
    beego.Controller
}

func (this *NewController) Get() {
    this.Layout = "layout.tpl"
    this.TplNames = "new.tpl"
}

func (this *NewController) Post() {
    inputs := this.Input()
    var blog models.Blog
    blog.Title = inputs.Get("title")
    blog.Content = inputs.Get("content")
    blog.Created = time.Now()
    models.SaveBlog(blog)
    this.Ctx.Redirect(302, "/")
}        
```
EditController
``` Go
type EditController struct {
    beego.Controller
}

func (this *EditController) Get() {
    inputs := this.Input()
    id, _ := strconv.Atoi(this.Ctx.Params[":id"])
    this.Data["Post"] = models.GetBlog(id)
    this.Layout = "layout.tpl"
    this.TplNames = "new.tpl"
}

func (this *EditController) Post() {
    inputs := this.Input()
    var blog models.Blog
    blog.Id, _ = strconv.Atoi(inputs.Get("id"))
    blog.Title = inputs.Get("title")
    blog.Content = inputs.Get("content")
    blog.Created = time.Now()
    models.SaveBlog(blog)
    this.Ctx.Redirect(302, "/")
}
```
DeleteController
``` Go
type DeleteController struct {
    beego.Controller
}

func (this *DeleteController) Get() {
    id, _ := strconv.Atoi(this.Ctx.Params[":id"])
    this.Data["Post"] = models.DelBlog(id)
    this.Ctx.Redirect(302, "/")
}    
```
## model 레이어
``` Go
package models

import (
    "database/sql"
    "github.com/astaxie/beedb"
    _ "github.com/ziutek/mymysql/godrv"
    "time"
)

type Blog struct {
    Id      int `PK`
    Title   string
    Content string
    Created time.Time
}

func GetLink() beedb.Model {
    db, err := sql.Open("mymysql", "blog/astaxie/123456")
    if err != nil {
        panic(err)
    }
    orm := beedb.New(db)
    return orm
}

func GetAll() (blogs []Blog) {
    db := GetLink()
    db.FindAll(&blogs)
    return
}

func GetBlog(id int) (blog Blog) {
    db := GetLink()
    db.Where("id=?", id).Find(&blog)
    return
}

func SaveBlog(blog Blog) (bg Blog) {
    db := GetLink()
    db.Save(&blog)
    return bg
}

func DelBlog(blog Blog) {
    db := GetLink()
    db.Delete(&blog)
    return
}
```

## view 레이어

layout.tpl
``` HTML
<html>
<head>
    <title>My Blog</title>
    <style>
        #menu {
            width: 200px;
            float: right;
        }
    </style>
</head>
<body>

<ul id=menu>
    <li><a href=/>Home</a></li>
    <li><a href=/new>New Post</a></li>
</ul>

{{.LayoutContent}}

</body>
</html>
```

index.tpl
``` HTML
<h1>Blog posts</h1>

<ul>
{{range .blogs}}
    <li>
        <a href="/view/{{.Id}}">{{.Title}}</a> 
        from {{.Created}}
        <a href="/edit/{{.Id}}">Edit</a>
        <a href="/delete/{{.Id}}">Delete</a>
    </li>
{{end}}
</ul>
```
view.tpl
``` Go
<h1>{{.Post.Title}}</h1>
{{.Post.Created}}<br/>

{{.Post.Content}}         
```
new.tpl
``` HTML
<h1>New Blog Post</h1>
<form action="" method="post">
Title:<input type="text" name="title"><br>
Content<textarea name="content" colspan="3" rowspan="10"></textarea>
<input type="submit">
</form>
```
edit.tpl
``` HTML
<h1>Edit {{.Post.Title}}</h1>

<h1>New Blog Post</h1>
<form action="" method="post">
Title:<input type="text" name="title" value="{{.Post.Title}}"><br>
Content<textarea name="content" colspan="3" rowspan="10">{{.Post.Content}}</textarea>
<input type="hidden" name="id" value="{{.Post.Id}}">
<input type="submit">
</form>

```