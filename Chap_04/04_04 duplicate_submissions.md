# Duplicate Submissions

지금까지 어느 BBS 나 블로그에서 보신 적이 있을지도 모르지만, 하나의 스레드와 문장 다음에 몇 개의 중복이 기록되어있을 수 있습니다. 이러한 대다수는 사용자가 여러 번 기록 양식을 제출했다 때문입니다. 다양한 원인에서 사용자는 대개 양식을 여러 번 보내 버립니다. 일반적으로 마우스의 오 조작에 의한 것으로, 전송 버튼을 더블 클릭 해 버리거나 일단 제출 한 내용을 다시 고치려고 브라우저의 뒤로 버튼을 누른 후 다음 버튼이 아니라 또한 전송 버튼을 누르는 것에 의한 것입니다. 당연히, 고의적 인 것도 있습니다. - - 예를 들어 인터넷 설문 조사와 제비 뽑기에서 중복 투표 등입니다. 는 어떻게하고 사용자가 동일한 내용의 양식을 제출하는 것을 효과적으로 막을 수 있을까요?

해결 방법은 양식에 고유 값을 가진 hidden 필드를 추가하는 것입니다. 양식을 확인할 때이 독특한 값을 가진 양식이 이미 제출되어 있는지 확인합니다. 만약 이미 전송 된 경우, 두 번째 전송을 거절합니다. 그렇지 않으면 양식에 처리 로직을 수행합니다. 또한 만약 Ajax 형식으로 제출 양식 인 경우 양식이 제출 된 후 javascript 의해 양식의 제출 단추를 금지합니다.

4.2 절의 예를 개량 해 봅시다 :

<input type = "checkbox"name = "interest"value = "football"> 축구
<input type = "checkbox"name = "interest"value = "basketball"> 농구
<input type = "checkbox"name = "interest"value = "tennis"> 테니스
사용자 이름 : <input type = "text"name = "username">
암호 : <input type = "password"name = "password">
<input type = "hidden"name = "token"value = "{ {} }">
<input type = "submit"value = "로그인">

템플릿에서`token`는 hidden 필드를 추가했습니다. 이 값은 MD5 (타임 스탬프)에 의해 고유 값을 할당합니다. 이 값을 서버에 저장함으로써 (session에 의한 컨트롤은 6 장에서 어떻게 저장하는지 설명합니다) 폼이 전송 될 때의 판정에 사용할 수 있습니다.

func login (w http.ResponseWriter, r * http.Request) {
fmt.Println ( "method :"r.Method) // 요청을받는 방법
if r.Method == "GET"{
crutime : = time.Now () Unix ()
h : = md5.New ()
io.WriteString (h, strconv.FormatInt (crutime 10))
token : = fmt.Sprintf ( "% x", h.Sum (nil))

t _ : = template.ParseFiles ( "login.gtpl")
t.Execute (w, token)
} else {
// 요청은 로그인 데이터입니다. 로그인 로직을 실행하여 판단합니다.
r.ParseForm ()
token : = r.Form.Get ( "token")
if token! = ""{
// token의 합법성을 확인합니다.
} else {
// token이 존재하지 않으면 에러가 발생한다.
}
fmt.Println ( "username length :"len (r.Form [ "username"] [0]))
fmt.Println ( "username :"template.HTMLEscapeString (r.Form.Get ( "username"))) // 서버 측에 출력합니다.
fmt.Println ( "password :"template.HTMLEscapeString (r.Form.Get ( "password")))
template.HTMLEscape (w [] byte (r.Form.Get ( "username"))) // 클라이언트에 출력합니다.
}
}

출력되는 페이지의 소스는 다음과 같습니다 :

! [] (images / 4.4.token.png? raw = true)

그림 4.4 token을 추가 한 후 클라이언트가 출력하는 소스 정보

token은 이미 출력 값을 가지고 있기 때문에 연속해서 페이지를 업데이트 할 수 있습니다. 이 값이 계속 변화하는이 아실 것입니다. 이처럼 매번 form이 표시 될 때 독특한되도록 보장합니다. 사용자가 제출 양식은 유일성이 유지됩니다.

이 해결 방법은 악의적 인 공격에 방지 할 수 있습니다. 또한 악의적 인 사용자에 대해서도 잠시 효과가 있습니다. 그런 다음 사용자에게이 악의적 인 동기를 버리게 할 수 없었던 경우는 더욱 복잡한 작업이 필요합니다.

