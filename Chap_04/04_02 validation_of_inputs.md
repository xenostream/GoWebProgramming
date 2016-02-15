# Validation of inputs

Web 개발시에는 사용자가 입력 한 어떤 정보도 믿지말라는 원칙이 있습니다. 따라서 사용자가 입력 한 정보를 검증하고    
필터링하는 것은 매우 중요한 작업이 되었습니다. 블로그나 뉴스에서 홈페이지가 해킹되거나 보안홀이 발생한 것을   
들었던 기억이 있을것입니다. 이들의 대부분은 사용자가 입력 한 정보를 홈페이지에서 엄격하게 검증하지 않았기 때문입니다.   
따라서 안전한 Web 프로그램을 작성하는 기법 중에 입력양식의 데이터를 검증하는 것은 매우 중요합니다.   
 
Web 응용프로그램을 개발할때는 주로 두군데에서 데이터 검증을 실시 합니다.    
첫번째는 웹페이지에서 JavaScript에 의한 검증(현재 이 방법은 많은 플러그인이 존재합니다. 예를 들어 ValidationJS 플러그인   
등이 있습니다)이고, 다른 하나는 서버 측에서의 유효성 검사 루틴입니다. 이 절에서는 서버측 검증에 대해서 설명 합니다.   

## 필수 필드
양식 요소에서 값하나를  추출하고자 합니다. 예를 들어 이전 예제의 사용자 아이디는 어떻게 처리 할 것인가 하는 문제입니다.    
Go는 builtin함수인 `len`함수를 통해서 문자열의 길이를 얻을 수 있습니다. 예를 들면 다음과 같습니다.   
``` Go
if len(r.Form["username"][0]) == 0 {
    // 입력값이 없을 경우의 처리
}
```
`r.Form`은 형태마다 폼 요소의 공백에 대해서 각기 다른 작업을 수행 합니다. 빈 텍스트 필드, 텍스트 영역 및 파일 업로드에는      
요소의 값을 비웁니다. 또한 선택되지 않은 콤보 상자 나 셀렉트 박스는 `r.Form`에서는 해당 항목을 생성하지 않습니다.     
따라서 `r.Form.Get()`을 사용해서 해당 값을 취해야 합니다. 왜냐하면 만약 필드가 존재하지 않을 경우에는, 이 방법은 빈 값을   
얻어서 알 수 있습니다. 하지만,`r.Form.Get()`의 경우는 하나의 값만 얻을 수 있습니다. 만약 map값과 같이 여러개라면 이방법은      
통하지 않습니다.    

## 숫자 검증 
예를 들어 양식에서 나이가 50세와 10세 등의 구체적인 숫자값만 필요하고, "아저씨", "젊은이" 등의 글자는 필요하지 않다고,   
가정할 경우 입력양식의 필드에 숫자만 허용하도록 시키고, 숫자일 경우에는 정수인지를 판단하기 위해   
먼저 int로 변환 한 후 처리 합니다.   

양의 정수를 판단하고자 하는 경우 먼저 int로 변환한 후 처리 합니다.     
``` Go
getint, err := strconv.Atoi(r.Form.Get("age"))
if err != nil {
    // 숫자 변환 오류가 발생했습니다. 즉, 숫자가 아닙니다.
}

// 다음으로는 숫자의 범위를 결정합니다.
if getint > 100 {
    // 너무 나이가 많습니다. 
}
```
또 다른 방법으로는  정규표현식으로 처리하는 방법입니다.
``` Go
if m , _ := regexp.MatchString("^[0-9] + $", r.Form.Get("age")); !m {
    return false
}
```
어떤방식이 더욱 성능과 안정성이 좋은지는 항상 토론되는 주제 중 하나입니다. 어떤이는 가능​​한 한 정규표현식을 피해야한다고   
생각하고 있습니다. 왜냐면 정규표현식의 처리 속도는 일반적으로 느리기 때문입니다. 그러나 현재처럼 컴퓨터 성능이 발달한    
환경이라면 간단한 정규표현식의 효율성과 형식변환 함수 사이의 성능차이는 크지 않습니다. 만약 개발자가 정규표현식에 익숙하다면,     
G​​o에서  제공하는 정규표현식을 사용하는 것도 편리한 방법 중 하나 입니다.   

> Go 정규표현식 구현은 [RE2](http://code.google.com/p/re2/wiki/Syntax)입니다. 모든 문자는 UTF-8 인코딩 입니다.

## 한국어 
폼 요소에서 한글이름만 얻고 싶은 경우에는, 올바른 한국어임을 보증하기 위해 확인을 해야합니다.    
사용자로 하여금 아무 문자열이나 입력시키도록 하지 않습니다. 한국어에 대한 올바른 검증법에는 `unicode` 패키지가 제공하는  
`func Is(rangeTab *RangeTable, r rune) bool` 함수와  정규 표현식을 사용하는 방법이 있습니다.    
다음의 코드를 참고하시기 바랍니다. 
```
if m,  _ := regexp.MatchString ("^\\p{Han}+$", r.Form.Get("realname")); !m {
    return false
}
```

## 영어
사용자의 영어 이름만 사용하고 싶을 때, 폼 요소에서 영어 값을 검색하면 `astaxie`여야만 하며,`as택시` 같은 것은 없어야
할 것입니다. 다음과 같이 간단한 정규식을 사용하여 데이터를 검증 할 수 있습니다.
``` Go
if m,  _ := regexp.MatchString("^[a-zA-Z] + $", r.Form.Get("engname")); !m {
    return false
}
```


## 이메일 주소
사용자가 입력 한 Email 주소가 올바른 것인지를 확인하려면, 다음과 같은 방법으로 확인할 수 있습니다.  
``` Go
if m,  _ := regexp.MatchString(`^(\w\\_] {2,10})@(\w{1}).([az] {2,4})$` r.Form.Get("email")); !m {
    fmt.Println("no")
} else {
    fmt.Println("yes")
}
```

## 전화 번호
사용자가 입력 한 전화번호가 올바른지를 확인하려면 다음의 정규 표현식처럼 확인할 수 있습니다.      
``` Go
if m,  _ : = regexp.MatchString(`^(01[0-9]\d{4,8})$`, r.Form.Get("mobile")); !m {
    return false
}
```

## 풀다운 메뉴
양식 중에서 `<select>` 요소가 생성하는 풀다운 메뉴에서 해커는 때때로 풀다운 메뉴에없는 항목을 위조한 후,   
허위 데이터를 보낼 수 도 있습니다. 이런 경우에 원래 설정된 값임을 확인하려면 어떻게 해야할까요?   

select에 다음과 같이 설정되어 있다고 가정 합니다.   
``` html
<select name = "fruit">
<option value = "apple"> apple </option>
<option value = "pear"> pear </option>
<option value = "banane"> banane </option>
</select>
```

이 경우 다음과 같은 방법으로 확인 할 수 있습니다.   
``` Go
slice := []string{"apple", "pear", "banane"}

for _ , v := range slice {
    if v == r.Form.Get("fruit") {
        return true
    }
}
return false
```


## 라디오 버튼
남자 /  여자와 같은 성별 구별을 하는일은 라디오 버튼을 이용해서 처리합니다.   
15세의 소년이 http프로토콜을 이용해서, telnet 클라이언트를 이용해서 프로그램에 해당 데이터를 전달했다고 가정하면,  
남자 1, 여자 2로 설정했다면, 만약 3이라는 값을 전송하면 프로그램에서는 예외를 처리할까요? 풀다운 메뉴에서 처럼 원래    
정상적으로 얻고자하는 값인지를 항상 판단해야 합니다.
``` html
<input type = "radio" name = "gender" value = "1"> 남자
<input type = "radio" name = "gender" value = "2"> 여자
```
풀다운 메뉴의 처리 방법과 같이 사용할 수 있습니다.    
``` Go
slice := []int{1,2}

for _,  v := range slice {
    if v == r.Form.Get("gender") {
        return true
    }
}
return false
```


## 체크박스 
취미와 같은 정보를 선택하는 체크박스의 경우를 알아보도록 하겠습니다.    
``` html
<input type = "checkbox" name = "interest" value = "football"> 축구
<input type = "checkbox" name = "interest" value = "basketball"> 농구
<input type = "checkbox" name = "interest" value = "tennis"> 테니스
```

체크박스는  라디오 버튼 방식과는 검증방법이 조금 다릅니다.왜냐하면 수신된 데이터가  slice형태이기 때문 입니다.   
``` Go
slice := []string{"football", "basketball", "tennis"}
a := Slice_diff(r.Form["interest", slice)
if a == nil {
    return true
}

return false
```

`Slice_diff`라는 함수는 오픈소스 외부 라이브러리입니다.(slice와 map을 처리하는 라이브러리)   
 [https://github.com/astaxie/beeku](https://github.com/astaxie/beeku)


## 날짜와 시간
사용자가 입력 한 날짜가 유효한지를 확인하고 싶을 경우가 있습니다. 예를들어 사용자가 8월 15일에 파티를 예정하거나,    
입력하거나 미래의 자기 생일이 무슨요일인지를 알아보는 경우 등입니다.    

Go는 `time` 패키지를 기본적으로 제공하고 있습니다. 사용자가 입력한 날짜를 원하는 시간으로 변환한 후  판단합니다.   
``` Go
t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
fmt.Printf("Go launched at %s \n", t.Local())
```

time 객체를 취득한 후 시간 함수를 사용해서 해당 작업을 처리할 수 있습니다. 구체적인 판단법은 요구사항에 맞게 조정 하십시오.   


## 각종 서식화  번호(카드번호, 주민번호 등...)
양식입력에서 주민등록번호를 입력하는 경우에 정규식을 사용하면 쉽게 확인할 수 있습니다. 그러나 주민등록 번호는 
6자리와 7자리가 특정한 용도가 있기 때문에 2가지  모두를 검증해야 합니다.
(주 : 주민등록번호의 마지막자리는 앞의 12자리를 연산한 후 생성되는 체크디지트 숫자 입니다.)
``` Go
// 13자리 주민등록번호 확인. 13자리 모두 숫자 입니다.
if m, _ := regexp.MatchString(`^(\d{13})$`, r.Form.Get("usercard")); !m {
    return false
}

// 13번째 숫자는 정식 주민등록번호임을 확인하는 체크 디지트입니다. 앞의 12자리를 연산한 후 13번째를 계산합니다.   
예제에서는 13번째 자리를 계산하지 않고, 숫자인지만 검사하는 방식으로 대신 설명 합니다.  
``` Go
if m,  _ := regexp.MatchString(`^(\d{13})([0-8]|9)$`, r.Form.Get("usercard")); !m {
    return false
}
```

이상으로, 자주 사용되는 서버측 입력 요소값을 검증하는 일부 방법에 대해서 설명했습니다. 
이 설명을 기초로 해서 Go언어를 이용한 데이터 검증, 특히 정규식의 처리에 대한 지식이 늘어났다면 좋겠습니다.
