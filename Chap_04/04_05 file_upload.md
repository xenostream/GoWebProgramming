# File Upload

사용자가 파일 업로드를 처리하고자합니다. 예를 들어, 현재 Instagram 같은 홈페이지를 만들고 있다고합니다. 사용자가 촬영 한 사진을 저장해야합니다. 이러한 요청은 어떻게 실현하는 것입니까?

양식에 파일을 업로드하기 위해서는 먼저 form의`enctype` 속성을 추가해야합니다. `enctype` 속성은 다음의 3 가지 종류가 있습니다 :

application / x-www-form-urlencoded 보내기 전에 모든 문자열을 인코딩 (기본)
multipart / form-data 문자열을 인코딩하지 않습니다. 파일 업로드 위젯을 포함하는 폼을 사용할 때이 값이 필요합니다.
text / plain 공백을 "+"기호로 바꿉니다. 그러나 특수 문자에 인코딩되지 않습니다.

따라서 양식의 html 코드는 다음과 같이됩니다 :

<html>
<head>
<title> 파일 업로드 </ title>
</ head>
<body>
<form enctype = "multipart / form-data"action = "http://127.0.0.1:9090/upload"method = "post">
<input type = "file"name = "uploadfile"/>
<input type = "hidden"name = "token"value = "{{}}"/>
<input type = "submit"value = "upload"/>
</ form>
</ body>
</ html>

서버는 handlerFunc을 하나 추가합니다 :

http.HandleFunc ( "/ upload"upload)

// / upload를 처리하는 로직
func upload (w http.ResponseWriter, r * http.Request) {
fmt.Println ( "method :"r.Method) // 요청을받는 방법
if r.Method == "GET"{
crutime : = time.Now () Unix ()
h : = md5.New ()
io.WriteString (h, strconv.FormatInt (crutime 10))
token : = fmt.Sprintf ( "% x", h.Sum (nil))

t _ : = template.ParseFiles ( "upload.gtpl")
t.Execute (w, token)
} else {
r.ParseMultipartForm (32 << 20)
file handler에, err : = r.FormFile ( "uploadfile")
if err! = nil {
fmt.Println (err)
return
}
defer file.Close ()
fmt.Fprintf (w "% v"handler.Header)
f, err : = os.OpenFile ( "./ test /"+ handler.Filename, os.O_WRONLY | os.O_CREATE, 0666)
if err! = nil {
fmt.Println (err)
return
}
defer f.Close ()
io.Copy (f, file)
}
}

위의 코드에서는 파일 업로드를 처리하기 위해서는`r.ParseMultipartForm`를 호출해야합니다. 인수는`maxMemory`가 표시되어 있습니다. `ParseMultipartForm`를 호출 한 후 업로드 할 파일은`maxMemory` 크기의 메모리에 저장됩니다. 만약 파일의 크기가`maxMemory`을 초과하는 경우, 남은 부분은 시스템의 임시 파일에 저장됩니다. `r.FormFile` 의해 위의 파일 핸들을 얻을 수 있습니다. 그 실례 중에서는`io.Copy`을 사용하여 파일을 저장하고 있습니다.

> 다른 파일이 아닌 필드 정보를 취득 할 때는`r.ParseForm`를 호출 할 필요는 없습니다. 필요할 때는 Go가 자동으로 호출합니다. 또한`ParseMultipartFrom`를 한 번 호출하면 나중에 다시 호출해도 효과는 없습니다.

위의 사례를 통해 파일 업로드는 주로 3 단계의 처리가 있음을 알 수 있습니다 :

1. 양식에 enctype = "multipart / form-data"를 추가한다.
2. 서버에서`r.ParseMultipartForm`를 호출하여 업로드 할 파일을 메모리 및 임시 파일에 저장한다.
3.`r.FormFile`을 사용하여 파일 핸들을 가져 파일에 저장 등의 처리를한다.

파일 handler는 multipart.FileHnadler입니다. 여기에는 다음과 같은 구조체가 저장되어 있습니다.

type FileHeader struct {
Filename string
Header textproto.MIMEHeader
// contains filtered or unexported fields
}

위의 예제에서는 다음과 같이 파일 업로드를 출력합니다.

! [] (images / 4.5.upload2.png? raw = true)

그림 4.5 파일을 업로드 한 후 서버가받은 정보 출력

## 클라이언트가 파일 업로드

위의 예에서 어떻게 폼에서 파일 업로드하는지 보여주었습니다. 그 서버에서 파일을 처리하지만, Go는 사실 클라이언트 폼 파일 업로드를 에뮬레이션하는 기능을 지원합니다. 자세한 사양은 아래의 예를 참조하십시오 :

package main

import (
"bytes"
"fmt"
"io"
"io / ioutil"
"mime / multipart"
"net / http"
"os"
)

func postFile (filename string, targetUrl string) error {
bodyBuf : = & bytes.Buffer {}
bodyWriter : = multipart.NewWriter (bodyBuf)

// 핵심 작업
fileWriter, err : = bodyWriter.CreateFormFile ( "uploadfile", filename)
if err! = nil {
fmt.Println ( "error writing to buffer")
return err
}

// 파일 핸들 조작을 오픈한다
fh, err : = os.Open (filename)
if err! = nil {
fmt.Println ( "error opening file")
return err
}
defer fh.Close ()

// iocopy
_ err = io.Copy (fileWriter, fh)
if err! = nil {
return err
}

contentType : = bodyWriter.FormDataContentType ()
bodyWriter.Close ()

resp, err : = http.Post (targetUrl, contentType, bodyBuf)
if err! = nil {
return err
}
defer resp.Body.Close ()
resp_body, err : = ioutil.ReadAll (resp.Body)
if err! = nil {
return err
}
fmt.Println (resp.Status)
fmt.Println (string (resp_body))
return nil
}

// sample usage
func main () {
target_url : = "http : // localhost : 9090 / upload"
filename : = "./astaxie.pdf"
postFile (filename, target_url)
}


위의 예에서는 클라이언트가 얼마나 서버에 하나의 파일을 업로드 할 것인지 설명했습니다. 클라이언트는 multipart.Write 통해 파일의 본문을 버퍼에 씁니다. 그 후, http의 Post 메소드를 호출하여 버퍼에서 서버로 전송합니다.

> 만약 당신이 다른 username 등 일반 필드를 동시에 쓸 경우 multipart의 WriteField 메소드를 호출하여 다른 유사한 필드를 여러 쓸 수 있습니다.

