# Prevent hijack of Session

session 납치 광범위하게 존재하는 비교적 심각한 취약점입니다. session 기술에서 클라이언트 측과 서버 사이드는 session의 ID로 세션을 유지합니다. 그러나이 ID는 쉽게 스니핑되어 제삼자에게 이용되어 버립니다. 이것은 중간자 공격의 일종입니다.

이 장에서는 세션 하이재킹의 실례를 보여드립니다. 이 사례를 통해 독자가 더 session의 본질에 대한 이해를하실 수 있도록 노력하고 있습니다.
## session 납치 과정
아래와 같은 count 카운터를 씁니다 :

func count (w http.ResponseWriter, r * http.Request) {
sess : = globalSessions.SessionStart (w, r)
ct : = sess.Get ( "countnum")
if ct == nil {
sess.Set ( "countnum"1)
} else {
sess.Set ( "countnum"(ct (int) + 1))
}
t _ : = template.ParseFiles ( "count.gtpl")
w.Header () Set ( "Content-Type", "text / html")
t.Execute (w, sess.Get ( "countnum"))
}


count.gtpl 코드는 다음과 같습니다 :

Hi. Now count : {{}}

브라우저에서 새로 고침을 수행하고 다음과 같은 내용을 확인할 수 있습니다 :

! [] (images / 6.4.hijack.png? raw = true)

그림 6.4 브라우저로 count 수를 표시

장전에 의해 숫자는 끝없이 증가합니다. 숫자가 6 일 때만 브라우저 (여기에서는 chrome을 예로합니다)의 cookie 관리자를 열면 다음과 같은 정보를 볼 수 있습니다 :


! [] (images / 6.4.cookie.png? raw = true)

그림 6.5 브라우저가 저장된 cookie를 취득

다음 단계가 중요합니다 : 다른 브라우저 (여기에서는 firefox 브라우저를 열었습니다)를 열고 chrome 주소창의 주소를 새롭게 열린 브라우저의 주소창에 복사합니다. 그 firefox의 cookie 에뮬레이션 플러그인을 열고 새 cookie를 만듭니다. 위 그림의 cookie의 내용을 그대로 firefox 안에 다시 설정합니다 :

! [] (images / 6.4.setcookie.png? raw = true)

그림 6.6 cookie를 에뮬레이트

엔터 키를 누르면 아래와 같은 내용이 나타납니다 :

! [] (images / 6.4.hijacksuccess.png? raw = true)

그림 6.7 session 하이재킹에 성공

브라우저를 바꿔도 sessionID를 얻을 수있었습니다. 이 후 cookie의 저장 과정을 에뮬레이트합니다. 이 예는 하나의 컴퓨터에서 한 것입니다. 비록 두 대의 통해 갔다해도 결과는 동일합니다. 이때 만약 교대로 두 브라우저의 링크를 클릭하면 작업중인 카운터가 실은 같은 것이라는 것을 알게 될 것이다. 놀라운 일은 아닙니다. 여기에서는 firefox이 chrome과 goserver 사이의 세션 유지의 열쇠를 훔쳐했습니다. 즉 gosessionid입니다. 이것은 "세션 하이재킹"의 일종입니다. goserver에서하면 http 요청에서 gosessionid을 얻었습니다. HTTP 프로토콜의 상태 정보에 의해 gosessionid이 chrome에서 "납치"된 것 따위 알 방법이 없습니다. 여전히 해당 session을 찾고 관련 계산을 수행합니다. 동시에 chrome도 자신이 보유하고있는 세션이 이미 "납치"된 것을 알 수있는 방법도 없습니다.
## session 납치 예방 조치
### cookieonly와 token
위의 세션 하이재킹의 간단한 예에서 session이 다른 사람에게 납치되면 매우 위험하다고 알았습니다. 납치 측은 납치 된 측면을 가장 많은 불법적 인 작업을 수행 할 수 있습니다. 그럼 어떻게 효과적으로 session 납치를 방지하는 것입니까?

하나의 방법은 sessionID 값에 cookie에 의해서만 설정되도록하는 것입니다. URL 재 작성 방법은 허용하지 않도록 동시에 cookie의 httponly을 true로 설정합니다. 이 속성은 클라이언트 측 스크립트가 설정된 cookie에 액세스 할 수 있는지 여부를 설정합니다. 먼저 이에 따라이 cookie가 XSS에 의해 읽혀져 session 납치를 일으키는 것을 방지 할 수 있습니다. 다음 cookie의 설정에서 URL 재 작성 방법에 의해 쉽게 sessionID를 얻을 수 없습니다.

2 단계는 각 요청에 token을 추가하는 것입니다. 앞 장에서 언급 한 form의 중복 전송을 방지하기 위해 비슷한 기능을 구현합니다. 각 요청에서 숨겨진 token을 추가하고 매번이 token을 검증함으로써 사용자의 요청이 독특한임을 보증합니다.

h : = md5.New ()
salt : = "astaxie % ^ 7 & 8888"
io.WriteString (h, salt + time.Now () String ())
token : = fmt.Sprintf ( "% x", h.Sum (nil))
if r.Form [ "token"]! = token {
// 로그인 화면을 표시
}
sess.Set ( "token"token)


### 간격을두고 새로운 SID를 생성하는
또 다른 방법은 session 외에 작성 시간을 마련하는 것입니다. 일정한 시간이 지나면이 sessionID은 삭제되고 다시 새로운 session이 생성됩니다. 이렇게하면 어느 정도 session 납치 문제를 방지 할 수 있습니다.

createtime : = sess.Get ( "createtime")
if createtime == nil {
sess.Set ( "createtime"time.Now () Unix ())
} else if (createtime (int64) + 60) <(time.Now () Unix ()) {
globalSessions.SessionDestroy (w, r)
sess = globalSessions.SessionStart (w, r)
}

session이 시작되면 생성 된 sessionID 시간을 기록하는 하나의 값이 설정됩니다. 매번 요청이 만료 (여기에서는 60 초로 설정하고 있습니다)을 초과하지 않거나 판단하고 정기적으로 새로운 ID를 생성합니다. 공격자는 유효한 s​​essionID를 얻을 수있는 기회를 크게 잃게됩니다.

위의 두 가지 방법을 결합과 실천에서 session 납치의 위험을 제거 할 수 있습니다. sessionID를 자주 바꾼다 공격자에 유효한 s​​essionID를 얻을 수있는 기회를 잃게됩니다. sessionID는 cookie에서 교환되고 httponly 설정되므로 URL을 기반 공격의 가능성은 제로입니다. 동시에 XSS에 따르면 sessionID의 취득도 불가능합니다. 마지막으로 MaxAge = 0을 설정합니다. 이를 통해 session cookie가 브라우저 로그에 기록되지 않습니다.


