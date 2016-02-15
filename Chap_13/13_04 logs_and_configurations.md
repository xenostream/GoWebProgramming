# Logs and Configurations

## 로그와 설정 중요성
로그가 유실됬습니다! 개발에서 로그가 아주 ​​ 중요한 영향을 미치는 것에 대해서 전에 설명 했습니다.   
로그를 통해서 프로그램과 정보를 디버깅 할 수 있습니다. 이미  seelog라는 로그 시스템을 설명 했습니다.   
level에 따라서 각각  다른 로그를 출력 합니다. 이것은 프로그램 개발 및 전개에 있어서 매우 중요 합니다.   
프로그램을 개발 중에는 level을 높게 설정 하고 배포시에는 level을 낮춤으로써, 개발 중에는 중요한 정보를   
검사하고 사용할 수 있으며, 배포시에는 중요한 정보를 은폐 할 수 있습니다.  

설정 모듈은 응용 프로그램을 배포서버에 배포할 경우 정보 설정을 할 수 있으므로, 매우 유용합니다.   
예를 들어 데이터베이스의 설정 정보, 포트 모니터링, 주소 모니터링 등은 설정 파일에서 설정을 할 수 있습니다.   
이런 응용 프로그램은 매우 강력한 유연성을 가지고 있기 때문에 다른 시스템에는  설정 파일을 조정하여 
다른 데이터베이스에 쉽게 액세스하는 등의 작업이 가능하도록 합니다. 

## beego 로그 설계
beego 로그 설계 배포 사상은 seelog에 근거하고 있습니다.
level별로 기록하지만 beego에서  설계한  로그 시스템은 상대적으로 경량 구조 입니다. 
시스템의 `log.Logger` 인터페이스를 사용하고 기본적으로 `os.Stdout`에 출력 합니다. 사용자는이 인터페이스를 
구현하는 것으로, beego.SetLoggr 설정을 통해 사용자 정의 출력을 설정할 수 있습니다. 전체 코드는  다음과 같습니다.  

```
// Log levels to control the logging output.
const (
    LevelTrace = iota
    LevelDebug
    LevelInfo
    LevelWarning
    LevelError
    LevelCritical
)

// logLevel controls the global log level used by the logger.
var level = LevelTrace

// LogLevel returns the global log level and can be used in
// own implementations of the logger interface.
func Level() int {
    return level
}

// SetLogLevel sets the global log level used by the simple
// logger.
func SetLevel(l int) {
    level = l
}
```
위의 로그 시스템은  로그 레벨을 구현하고 있습니다. 기본 수준은 Trace로서 사용자는 SetLevel을 사용해서,   
가각 다른 레벨을 설정할 수 있습니다.
```
// logger references the used application logger.
var BeeLogger = log.New(os.Stdout, "", log.Ldate|log.Ltime)

// SetLogger sets a new logger.
func SetLogger(l *log.Logger) {
    BeeLogger = l
}

// Trace logs a message at trace level.
func Trace(v ...interface{}) {
    if level <= LevelTrace {
        BeeLogger.Printf("[T] %v\n", v)
    }
}

// Debug logs a message at debug level.
func Debug(v ...interface{}) {
    if level <= LevelDebug {
        BeeLogger.Printf("[D] %v\n", v)
    }
}

// Info logs a message at info level.
func Info(v ...interface{}) {
    if level <= LevelInfo {
        BeeLogger.Printf("[I] %v\n", v)
    }
}

// Warning logs a message at warning level.
func Warn(v ...interface{}) {
    if level <= LevelWarning {
        BeeLogger.Printf("[W] %v\n", v)
    }
}

// Error logs a message at error level.
func Error(v ...interface{}) {
    if level <= LevelError {
        BeeLogger.Printf("[E] %v\n", v)
    }
}

// Critical logs a message at critical level.
func Critical(v ...interface{}) {
    if level <= LevelCritical {
        BeeLogger.Printf("[C] %v\n", v)
    }
}
```
위의 코드는 기본적으로 BeeLogger 객체를 초기화하고, 기본적으로 os.Stdout에 출력 합니다.   
사용자는 beego.SetLogger로 logger 인터페이스 출력을 구현 할 수 있습니다.   
여기서는  6개의 함수를 구현하고 있습니다.

- Trace (일반적인 정보의 기록, 예 :)
    - "Entered parse function validation block"
    - "Validation : entered second 'if'"
    - "Dictionary 'Dict'is empty. Using default value"
- Debug (디버그 정보, 예 :)
    - "Web page requested : http://somesite.com Params = '...'"
    - "Response generated. Response size : 10000. Sending"
    - "New file received. Type : PNG Size : 20000"
- Info (인쇄 정보, 예 :)
    - "Web server restarted"
    - "Hourly statistics : Requested pages : 12345 Errors : 123 ..."
    - "Service paused. Waiting for 'resume'call"
- Warn (경고 정보, 예 :)
    - "Cache corrupted for file = 'test.file'. Reading from back-end"
    - "Database 192.168.0.7/DB not responding. Using backup 192.168.0.8/DB"
    - "No response from statistics server. Statistics not sent"
- Error (오류 정보, 예 :)
    - "Internal error. Can not process request # 12345 Error : ...."
    - "Can not perform login : credentials DB not responding"
- Critical (치명적 오류 예 :)
    - "Critical panic received : .... Shutting down"
    - "Fatal error : ... App is shutting down to prevent data corruption or loss"

모든 함수의 level에 대한 설명은 이미 아실 것입니다, 
그래서 만약 배포시  level=LevelWarning 을  설정하면 Trace, Debug, Info 세 함수는 아무것도 출력하지 않습니다.

## beego의 구성 설계
설정 정보의 분석에 beego은 key=value 형식의 설정 파일을 사용하고 있습니다. 이것은 ini 설정 파일 형식과 유사하며  
파일을 파싱하는 방식 입니다. 그 후 분석한 데이터를 map에 저장한 후, 호출 할 때 몇 가지 string, int 등  
해당 값을 함수에  반환 합니다. 구체적인 구현은 다음을 참조 하십시오.

먼저 ini 설정 파일의 글로벌 상수들을 정의합니다 :
```
var (
    bComment = []byte{'#'}
    bEmpty   = []byte{}
    bEqual   = []byte{'='}
    bDQuote  = []byte{'"'}
)
```

정의된 설정 파일 포맷 :
```
// A Config represents the configuration.
type Config struct {
    filename string
    comment  map[int][]string  // id: []{comment, key...}; id 1 is for main comment.
    data     map[string]string // key: value
    offset   map[string]int64  // key: offset; for editing.
    sync.RWMutex
}
```
분석 파일의 함수를 정의하면 먼저 파일을 엽니 다. 그리고 한줄 한줄 읽기, 코멘트, 빈행 및 key=value 데이터를  
기준으로 구문을 분석 합니다.
```
// ParseFile creates a new Config and parses the file configuration from the
// named file.
func LoadConfig(name string) (*Config, error) {
    file, err := os.Open(name)
    if err != nil {
        return nil, err
    }

    cfg := &Config{
        file.Name(),
        make(map[int][]string),
        make(map[string]string),
        make(map[string]int64),
        sync.RWMutex{},
    }
    cfg.Lock()
    defer cfg.Unlock()
    defer file.Close()

    var comment bytes.Buffer
    buf := bufio.NewReader(file)

    for nComment, off := 0, int64(1); ; {
        line, _, err := buf.ReadLine()
        if err == io.EOF {
            break
        }
        if bytes.Equal(line, bEmpty) {
            continue
        }

        off += int64(len(line))

        if bytes.HasPrefix(line, bComment) {
            line = bytes.TrimLeft(line, "#")
            line = bytes.TrimLeftFunc(line, unicode.IsSpace)
            comment.Write(line)
            comment.WriteByte('\n')
            continue
        }
        if comment.Len() != 0 {
            cfg.comment[nComment] = []string{comment.String()}
            comment.Reset()
            nComment++
        }

        val := bytes.SplitN(line, bEqual, 2)
        if bytes.HasPrefix(val[1], bDQuote) {
            val[1] = bytes.Trim(val[1], `"`)
        }

        key := strings.TrimSpace(string(val[0]))
        cfg.comment[nComment-1] = append(cfg.comment[nComment-1], key)
        cfg.data[key] = strings.TrimSpace(string(val[1]))
        cfg.offset[key] = off
    }
    return cfg, nil
}

```
다음의 코드는 설정파일을 읽어서, 원하는 값을 bool, int, float64 또는 string 값으로 반환합니다.  
```
// Bool returns the boolean value for a given key.
func (c *Config) Bool(key string) (bool, error) {
    return strconv.ParseBool(c.data[key])
}

// Int returns the integer value for a given key.
func (c *Config) Int(key string) (int, error) {
    return strconv.Atoi(c.data[key])
}

// Float returns the float value for a given key.
func (c *Config) Float(key string) (float64, error) {
    return strconv.ParseFloat(c.data[key], 64)
}

// String returns the string value for a given key.
func (c *Config) String(key string) string {
    return c.data[key]
}

```
## 응용
다음 함수는 응용 프로그램의 예입니다. 원격 url 주소의 json 데이터를 검색하는 데 사용 합니다.   
구현은 다음과 같습니다.
```
func GetJson() {
    resp, err := http.Get(beego.AppConfig.String("url"))
    if err != nil {
        beego.Critical("http get info error")
        return
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    err = json.Unmarshal(body, &AllInfo)
    if err != nil {
        beego.Critical("error:", err)
    }
}
```
함수에서는 프레임워크의 로그 함수 인`beego.Critical` 함수를 호출하여 오류를 발생시키고 있습니다.   
`beego.AppConfig.String("url")`를 호출하여 설정 파일의 정보를 가져 옵니다. 
설정 파일의 내용은 다음과 같습니다 (app.conf) :
```
appname = hs
url ="http://www.api.com/api.html"
```
