# Files

어떤 컴퓨터 시설에서도 파일이 필요합니다. 또한 Web 프로그래밍 파일 작업은 Web 프로그래머가 부딪 치는 문제입니다. 파일 작업은 Web 애플리케이션에서 필요하며 매우 유용합니다. 우리가 잘 디렉토리 (폴더) 생성, 파일 편집 같은 작업을 할 수 있습니다. 여기에서는 Go 의한 이러한 작업을 일부 요약 해 자세히 설명하고 어떻게 사용하는지 실례를 보여드립니다.
## 디렉토리 작업
파일 조작의 대부분의 함수는 아무도 os 패키지에 있습니다. 아래에 몇 가지 디렉터리 조작을 할 것을 들어 있습니다 :

- func Mkdir (name string, perm FileMode) error

이름이 name 디렉토리를 작성합니다. 권한 설정은 perm, 예를 들면 0777입니다.

- func MkdirAll (path string, perm FileMode) error

path 따라 계층적인 하위 디렉토리를 만듭니다. 예를 들어 astaxie / test1 / test2입니다.

- func Remove (name string) error

이름이 name 디렉토리를 제거합니다. 디렉토리에 파일 또는 다른 디렉토리가있는 경우 오류를 발생시킵니다.

- func RemoveAll (path string) error

path 따라 계층적인 서브 디렉토리를 제거합니다. 예를 들어 path가 하나의 이름 인 경우,이 디렉토리의 하위 디렉토리가 모두 삭제됩니다.


다음은 예제입니다 :

package main

import (
"fmt"
"os"
)

func main () {
os.Mkdir ( "astaxie", 0777)
os.MkdirAll ( "astaxie / test1 / test2", 0777)
err : = os.Remove ( "astaxie")
if err! = nil {
fmt.Println (err)
}
os.RemoveAll ( "astaxie")
}


## 파일 작업

### 새와 파일 열기
새 파일을 만들려면 다음 두 가지 방법이 있습니다

- func Create (name string) (file * File, err Error)

주어진 파일 이름에 따라 새 파일을 만들고 파일 오브젝트를 돌려줍니다. 기본적으로 권한은 0666의 파일입니다. 반환 된 파일 객체는 읽고 쓸 수 있습니다.

- func NewFile (fd uintptr, name string) * File

파일 디스크립터에 따라 해당 파일을 만들고 파일 오브젝트를 돌려줍니다.


다음 두 가지 방법에 의해 파일을 엽니 다 :

- func Open (name string) (file * File, err Error)

이 메소드는 이름이 name 파일을 엽니 다. 그러나 읽기 밖에 없습니다. 내부에서는 사실 OpenFile가 호출되어 있습니다.

- func OpenFile (name string, flag int, perm uint32) (file * File, err Error)

이름이 name 파일을 엽니 다. flag는 오픈 모드입니다. 읽기만하거나 읽고 쓸 수 있는지 등입니다. perm은 권한입니다.

### 파일에 쓰기
파일에 쓰는 함수 :

- func (file * File) Write (b [] byte) (n int, err Error)

byte 형의 정보를 파일에 기록합니다.

- func (file * File) WriteAt (b [] byte, off int64) (n int, err Error)

지정된 위치에서 시작 byte 형의 정보를 기록합니다.

- func (file * File) WriteString (s string) (ret int, err Error)

string 정보를 파일에 기록합니다.

파일에 쓰는 예제

package main

import (
"fmt"
"os"
)

func main () {
userFile : = "astaxie.txt"
fout, err : = os.Create (userFile)
if err! = nil {
fmt.Println (userFile, err)
return
}
defer fout.Close ()
for i : = 0; i <10; i ++ {
fout.WriteString ( "Just a test! \ r \ n")
fout.Write ([] byte ( "Just a test! \ r \ n"))
}
}

### 파일로드
파일에로드하는 함수 :

- func (file * File) Read (b [] byte) (n int, err Error)

데이터를 읽어 b에 전달합니다

- func (file * File) ReadAt (b [] byte, off int64) (n int, err Error)

off로부터 시작하여 데이터를 읽기 b에 전달합니다

파일을 읽는 코드 예 :

package main

import (
"fmt"
"os"
)

func main () {
userFile : = "asatxie.txt"
fl, err : = os.Open (userFile)
if err! = nil {
fmt.Println (userFile, err)
return
}
defer fl.Close ()
buf : = make ([] byte, 1024)
for {
n _ : = fl.Read (buf)
if 0 == n {
break
}
os.Stdout.Write (buf [: n])
}
}

### 파일 삭제
Go 언어에서 파일 삭제 및 디렉토리의 삭제는 같은 함수에서 이루어집니다

- func Remove (name string) Error

이 함수를 호출하여 파일 이름이 name 파일을 제거 할 수 있습니다
