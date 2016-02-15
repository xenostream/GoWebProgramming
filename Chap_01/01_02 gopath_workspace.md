---
layout: post
title: 'Go Web Programming: [01/02] GOPATH & Wroking Directory'
tags: goweb
---  

# $GOPATH 및 작업 디렉토리

# $GOPATH

Go명령어를 이용할 경우 대부분의 작업은 ```$GOPATH``` 라는 환경변수에 절대적으로 의존하게 됩니다.    
설치시 사용했었던, *$GOROOT* 환경변수와는 별도로 작동하는 변수입니다.    
개발자가 작성한 모든 패키지의 루트가 되는 디렉토리를 지정하는 환경변수입니다.    
대부분의 경우 워크스페이스라고 부르는 프로젝트의 루트디렉토리를 지정하는 것입니다. 
유닉스 계열의 운영체제에서는 다음과 같이 설정합니다. (사용자는 xeno이며, 홈디렉토리의 mygo )
```
export GOPATH=/home/xeno/mygo
```
윈도우의 경우에는 별도로 **GOPATH** 환경변수를 생성한 후 설정하여야 합니다.    
```go get```명령을 사용해서 원격저장소의 소스를 받아올때 GOPATH에 설정된 디렉토리를 기준으로     
작업하게 됩니다.

**$GOPATH** 디렉토리에는 반드시 다음과 같이 3개의 디렉토리가 존재해야 합니다.
* ```src``` 소스코드가 저장될 디렉토리 ( .go, .c, .g, .s ... )
* ```pkg``` 컴파일 후 라이브러리 파일이저장될 디렉토리 ( .a ... )
* ```bin``` 빌드 후 실행파일이 저장될 디렉토리 : go install 명령 사용 ( .exe ... )

본 가이드에서는 mygo 디렉토리를 GOPATH로 사용하게 됩니다.

#### 패키지 디렉토리
예를들어, ```$GOPATH/src/mymath/sqrt.go```라는 소스파일을 생성할 경우(mymath가 패키지이름)  
패키지 이름과 디렉토리 명을 통일 시켜서 각각의 패키지들을 구분해서 사용하게 됩니다.      

대형 프로젝트나 외부에 노출할 패키지의 경우에는 다중경로명을 사용해서 패키지를 구별할 수 있습니다.     
```github.com/xenostream/mymath``` 와 같은 경우가 그예입니다. (각각의 이름은 디렉토리 이름입니다.)    

상기와 같이 패키지를 저장했다면, 다음과 같이 디렉토리를 생성합니다.
```
cd $GOPATH/src
mkdir -p github.com/xenostream/mymath
cd $GOPATH/src/github.com/xenostream/mymath
```
sqrt.go 라는 파일을 생성한 후 다음과 같이 입력합니다. 
```go
// Source code of $GOPATH/src/github.com/xenostream/mymath/sqrt.go
package mymath

func Sqrt(x float64) float64 {
    z := 0.0
    for i := 0; i < 1000; i++ {
        z -= (z*z - x) / (2 * x)
    }
    return z
}
```
강제사항은 아니지만, 최대한 패키지이름의 디렉토리를 생성해서 사용하시기 바랍니다.    
해당 패키지에 연관된 소스를 한데모아서 사용하는 것이 패키지입니다.

#### 패키지 컴파일 
상기의 명령을 통해서 패키지를 이미 생성했습니다.  이제 소스파일을 컴파일하는 것에 대해서      
설명합니다. 컴파일하는 방법은 대체로 다음과 같이 두가지 방법이 있습니다. 

1. 컴파일 할 소스 디렉토리로 이동 한 후 ```go install``` 명령으로 컴파일
2. 상기 명령에서 확장자를 뺀 패키지명으로 컴파일 ```go install mymath``` 

상기의 예제코드에는 main 패키지가 없으므로, 라이브러리 패키지 입니다. 그래서 다음과같은     
디렉토리에 해당 결과물이 생성됩니다. 
```
$GOPATH/pkg/${GOOS}_${GOARCH}
mymath.a
```
**.a** 확장자의 의미는 바이너리 패키지를 의미합니다. 이제 생성한 라이브러리 패키지를 사용하겠습니다.    
```
cd $GOPATH/src
mkdir mathapp
cd mathapp
vi main.go
```
상기 명령으로 mymath 패키지를 실제로 사용할 프로그램을 작성하는 것입니다.  다음과 같이 코드를 입력합니다.    
```go
//$GOPATH/src/mathapp/main.go 
package main

import (
    "mymath"
    "fmt"
)

func main() {
    fmt.Printf("Hello, world. Sqrt(2) = %v\n", mymath.Sqrt(2))
}
```
이미 설명했던 컴파일 방법으로(```cd $GOPATH/src/mathapp; go install```) 프로그램을 컴파일 합니다.    
main 함수를 가지고 있는 main 패키지이므로 ```$GOPATH/bin``` 디렉토리에 결과물이 생성됩니다.     
```
cd $GOPATH/bin
./mathapp
```
상기의 명령으로 작성한 프로그램을 컴파일 한 후 실행합니다.    

```Hello world. Sqrt(2) = 1.414213562373095```

#### 원격 패키지 설치
Go언어는 원격 패키지를 설치할 수 있는 기능을 제공합니다. ```go get``` 명령을 이용하면, 여러가지    
원격저장소에 저장된 패키지를 가져와서 사용할 수 있게 됩니다. GitHub, Google Code, BitBurket, Launchpad등    
```go get github.com/xenostream/mymath```

상기와 같이 명령하면 해당 패키지를 **$GOPATH/src 디렉토리 밑에 디렉토리 구조를 유지한 채 가져옵니다.    
```go get -u 패키지이름``` 과 같이 사용할 경우 최신버전으로 업데이트 하게 됩니다.     
또한, 패키지 내부에서 사용하는 또다른 패키지 또한 함께 가져오게 됩니다.     

```
$GOPATH
    src
     |-github.com
          |-xenostream
               |-mymath
    pkg
     |--${GOOS}_${GOARCH}
          |-github.com
               |-xenostream
                    |-mymath.a

```
**go get** 명령을 사용하면 상기와 같은 구조로 생성됩니다.  사실 원격패키지의 복사본을 가져오게 됩니다.    
이미 말씀드렸듯이, **$GOPATH/src** 디렉토리 밑에 생성됩니다.   

상기와 같이 원격 패키지를 가져왔다면, 사용을 하려는 소스에서 다음과 같이 선언해서 사용합니다. 

```import "github.com/xenostream/mymath```


#### 최종 디렉토리 구조
지금까지의 명령어와 스텝을 수행했다면 다음과 같은 디렉토리 구조를 가지게 됩니다.    
```
bin/
    mathapp
pkg/
    ${GOOS}_${GOARCH} 예)darwin_amd64, linux_amd64
  mymath.a
  github.com/
    xenostream/
      mymath.a
src/
    mathapp
        main.go
    github.com/
        xenostream/
            mymath/
                sqrt.go

```
**bin**디렉토리에 실행파일이 생성되고, **src**디렉토리에 소스가 저장되며, **pkg**디렉토리에    
컴파일된 라이브러리가 존재한다고 생각하시면 됩니다. 

