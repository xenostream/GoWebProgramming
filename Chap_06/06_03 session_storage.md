# Session Storage

위의 절에서 Session 관리자의 구현 원리를 소개했습니다. session을 저장하는 인터페이스를 정의 했으므로, 여기서는 메모리에 기반 session 스토리지 인터페이스의 구현 예를 설명합니다. 기타 저장 방법은 직접 예제를 구현하려고합니다. 메모리의 구현에 대해서는 아래의 예제를 참조하십시오.

package memory

import (
"container / list"
"github.com/astaxie/session"
"sync"
"time"
)

var pder = & Provider {list : list.New ()}

type SessionStore struct {
sid string // session id 고유 ID
timeAccessed time.Time // 마지막 액세스 시간
value map [interface {} interface {} // session에 저장되는 값
}

func (st * SessionStore) Set (key, value interface {}) error {
st.value [key] = value
pder.SessionUpdate (st.sid)
return nil
}

func (st * SessionStore) Get (key interface {}) interface {} {
pder.SessionUpdate (st.sid)
if v, ok : = st.value [key]; ok {
return v
} else {
return nil
}
return nil
}

func (st * SessionStore) Delete (key interface {}) error {
delete (st.value, key)
pder.SessionUpdate (st.sid)
return nil
}

func (st * SessionStore) SessionID () string {
return st.sid
}

type Provider struct {
lock sync.Mutex // 잠금을 사용합니다
sessions map [string] * list.Element // 메모리에 저장하는 데 사용합니다
list * list.List // gc를 수행하는 데 사용합니다
}

func (pder * Provider) SessionInit (sid string) (session.Session, error) {
pder.lock.Lock ()
defer pder.lock.Unlock ()
v : = make (map [interface {} interface {} 0)
newsess : = & SessionStore {sid : sid, timeAccessed : time.Now () value : v}
element : = pder.list.PushBack (newsess)
pder.sessions [sid] = element
return newsess, nil
}

func (pder * Provider) SessionRead (sid string) (session.Session, error) {
if element, ok : = pder.sessions [sid]; ok {
return element.Value (* SessionStore), nil
} else {
sess, err : = pder.SessionInit (sid)
return sess, err
}
return nil, nil
}

func (pder * Provider) SessionDestroy (sid string) error {
if element, ok : = pder.sessions [sid]; ok {
delete (pder.sessions, sid)
pder.list.Remove (element)
return nil
}
return nil
}

func (pder * Provider) SessionGC (maxlifetime int64) {
pder.lock.Lock ()
defer pder.lock.Unlock ()

for {
element : = pder.list.Back ()
if element == nil {
break
}
if (element.Value (* SessionStore) .timeAccessed.Unix () + maxlifetime) <time.Now () Unix () {
pder.list.Remove (element)
delete (pder.sessions, element.Value (* SessionStore) .sid)
} else {
break
}
}
}

func (pder * Provider) SessionUpdate (sid string) error {
pder.lock.Lock ()
defer pder.lock.Unlock ()
if element, ok : = pder.sessions [sid]; ok {
element.Value (* SessionStore) .timeAccessed = time.Now ()
pder.list.MoveToFront (element)
return nil
}
return nil
}

func init () {
pder.sessions = make (map [string] * list.Element 0)
session.Register ( "memory"pder)
}

위의 코드는 메모리에 저장하는 session 메커니즘을 실현하고 있습니다. init 함수를 통해 session 관리자에 등록됩니다. 이렇게 쉽게 호출 할 수 있습니다. 어떻게이 엔진을 호출하는 것입니까? 아래의 코드를 참조하십시오.

import (
"github.com/astaxie/session"
_ "github.com/astaxie/session/providers/memory"
)

import를 할 때 memory 함수는 init 함수가 이미 실행되고 있습니다. 이는 이미 session 관리자에 등록이 완료된 있기 때문에, 사용 할 수 있습니다. 아래의 방법으로 session 관리자를 초기화 할 수 있습니다 :

var globalSessions * session.Manager

//이 후 init 함수로 초기화합니다.
func init () {
globalSessions _ = session.NewManager ( "memory", "gosessionid", 3600)
go globalSessions.GC ()
}
