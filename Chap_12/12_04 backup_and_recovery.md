# Backup and Recovery

이 절에서는 응용 프로그램을 관리하는 또 다른 측면에 대해 토론하고 싶다고 생각합니다 : 서버에서 생성 된 데이터의 백업 및 복원 정보입니다. 서버의 네트워크가 끊기거나 하드 디스크가 손상되거나 OS가 붕괴하거나 데이터베이스를 사용할 수 없게되거나하는 각종 이상 상태가 자주 발생합니다. 따라서 메인테이너 서버에서 발생하는 응용 프로그램과 데이터에 원격 재해 복구 콜드 대기 및 핫 대기 등 준비를해야합니다. 다음 소개에서 어떻게 응용 프로그램의 백업을 수행하거나 Mysql 데이터베이스 및 redis 데이터베이스 백업 / 복원에 대해 설명합니다.

## 응용 프로그램의 백업
많은 클러스터 환경에서 Web 응용 프로그램은 기본적으로 백업 할 필요가 없습니다. 왜냐하면 그것은 단순한 코드의 복사본 밖에 없기 때문입니다. 로컬 개발 환경 또는 버전 컨트롤 시스템에서 이미 이러한 코드를 유지하고 있습니다. 그러나 많은 경우 일부 개발 사이트에서는 사용자가 파일을 업로드해야 이러한 사용자가 업로드 한 파일에 대해 백업을 수행해야합니다. 현재 합리적인 방법은 웹 사이트와 관련된 저장 될 파일을 클라우드 스토리지에 저장하는 것입니다. 이렇게하면 시스템이 붕괴해도 클라우드 스토리지에 있습니다 만하면 데이터가 손실 될 수는 없습니다.

만약 클라우드 스토리지를 사용하지 않으면 어떻게 웹 사이트 백업을 할 것입니까? 여기에서는 파일 동기화 도구 인 rsync를 소개합니다 : rsync는 웹 사이트의 복사를 할 수 있으며, 다른 시스템의 파일을 동기화 할 수 있습니다. 만약 windows이면 windows 버전 cwrsync가 필요합니다.

### rsync 설치
rsync의 공식 사이트 : http : //rsync.samba.org/에서 최신 버전의 소스 코드를 얻을 수 있습니다. 당연히, rsync는 매우 간편한 소프트웨어이므로 많은 Linux 배포판에서 그 안에수록되어 있습니다.

소프트웨어 패키지 설치

# sudo apt-get install rsync 참고 : debian, ubuntu 등의 라이브 설치 방법;
# yum install rsync 참고 : Fedora, Redhat, CentOS 등 라이브 설치 방법;
# rpm -ivh rsync 참고 : Fedora, Redhat, CentOS 등 rpm 패키지 설치 방법;

다른 Linux 배포판은 해당 소프트웨어 패키지 관리 방법에 따라 설치하십시오. 소스 코드 패키지의 설치

tar xvf rsync-xxx.tar.gz
cd rsync-xxx
./configure --prefix = / usr; make; make install 참고 : 소스 코드 패키지를 컴파일하고 설치하기 전에 gcc가 컴파일 도구를 설치해야합니다;

### rsync 설정
rsync는 주로 다음의 3 가지 설정 파일 rsyncd.conf (주 설정 파일) rsyncd.secrets (암호 파일) rsyncd.motd (rsync 서버의 정보)가 있습니다.

이러한 파일의 설정에 관해서는 여러분은 공식 사이트 및 기타 rsync를 소개하고있는 사이트를 참고하실 수 있습니다. 다음은 서버 사이드와 클라이언트 사이드가 어떻게 시작하는지 소개합니다.

- 서버 사이드의 시작 :

# / usr / bin / rsync --daemon --config = / etc / rsyncd.conf

--daemon 옵션 방식은 rsync를 서버 모드로 실행합니다. rsync를 시작할 때 시작하려면

echo 'rsync --daemon'>> /etc/rc.d/rc.local

rsync 암호를 설정

echo '사용자 이름 : 암호'> /etc/rsyncd.secrets
chmod 600 /etc/rsyncd.secrets


- 클라이언트 측 동기화 :

클라이언트 사이드는 다음 명령으로 서버의 파일과 동기화 할 수 있습니다 :

rsync -avzP --delete --password-file = rsyncd.secrets 사용자 이름 @ 192.168.145.5 : www / var / rsync / backup

이 명령의 몇 가지 요점을 간략하게 아래에 설명합니다 :

1. -avzP이 무엇인지, 독자는 --help를 사용하여 확인할 수 있습니다.
2. --delete는 A에서 파일을 삭제하면 동기화 할 때 B는 자동으로 해당 파일을 삭제합니다.
3. --password-file 클라이언트 측 /etc/rsyncd.secrets에서 설정된 암호로 서버 측 /etc/rsyncd.secrets 중의 암호와 일치시킬 필요가 있습니다. 따라서 cron을 실행하면 암호를 입력 할 필요가 없습니다.
4.이 명령에서 "사용자 이름"은 서버 사이드의 /etc/rsyncd.secrets 중 사용자 이름입니다.
5.이 명령에서 192.168.145.5은 서버의 IP 주소입니다.
6. :: www 두 콜론 마크에주의하십시오. www는 서버의 설정 파일 /etc/rsyncd.conf에있는 www입니다. 의미는 서버의 /etc/rsyncd.conf 따라 그 속에서 www 필드의 내용을 동기화합니다. 하나의 콜론 마크 때는 설정 파일을 따르지 않고 직접 지정한 디렉토리를 동기화합니다.

동기화에 실시간 성을 갖게하기 ​​위해 crontab을 설정 rsync 분마다 동기화해도 괜찮습니다. 당연히 사용자는 파일의 중요도에 따라 다른 동기화 빈도를 설정할 수도 있습니다.


## MySQL의 백업
응용 프로그램 데이터베이스는 현재 역시 MySQL이 주류입니다. 현재 MySQL의 백업에는 두 가지 방법이 있습니다 : 핫 대기와 콜드 대기입니다. 핫 스탠드 바이는 현재 주로 master / slave 방식을 취하고 있습니다 (master / slave 방식의 동기화는 현재 데이터베이스의 읽기 및 쓰기를 분리합니다. 핫 대기에서도 사용할 수 있습니다). 어떻게이 방면의 자료를 설정하는지에 대해서는 얼마든지 검색 할 수 있습니다. 콜드 대기의 경우 데이터베이스에는 일정한 치 원형이 존재합니다. 하지만 이번에는 이전 데이터를 완벽하게 보장 할 수 있습니다. 예를 들어, 실수가 데이터 손실을 일으키는 원인이되었다 같은 경우 master / slave 모드에서 손실 된 데이터를 되돌릴 수 없습니다. 그러나 콜드 대기에서는 데이터의 일부를 복원 할 수 있습니다.

콜드 대기는 일반적으로 shell 스크립트를 사용하여 시간마다 데이터베이스를 백업 할 수 있습니다. 위에서 소개 한 rsync에 의해 로컬이 아닌 데이터 센터의 서버 중 하나에 동기화합니다.

다음은 mysql을 백업을 정기적으로 수행하는 스크립트입니다. mysqldump 프로그램을 사용하고 있으며,이 명령은 데이터베이스를 하나의 파일로 내 보냅니다.

#! / bin / bash

    # 다음 설정 정보는 직접 수정하십시오.
    mysql_user = "USER"#MySQL 백업 사용자
    mysql_password = "PASSWORD"#MySQL 백업 사용자 암호
    mysql_host = "localhost"
    mysql_port = "3306"
    mysql_charset = "utf8"#MySQL 문자 인코딩
    backup_db_arr = ( "db1" "db2") # 백업 할 데이터베이스의 이름 여러 경우 공백에 의해 구분됩니다. 예를 들어 ( "db1" "db2" "db3")
    backup_location = / var / www / mysql # 백업 된 데이터의 위치 끝에 "/"를 포함하지 않도록하십시오. 이 항목은 기본 상태에서도 괜찮습니다. 프로그램은 자동으로 디렉토리를 만듭니다.
    expire_backup_delete = "ON"# 만료 된 백업을 삭제할지 여부. ON에서 시작, OFF로 정지
    expire_days = 3 # 기한 일자. 기본값은 3 일이 항목은 expire_backup_delete을 시작했을 때만 유효합니다.

    #이 행 이후에는 수정할 필요가 없습니다.
    backup_time =`date + % Y % m % d % H % M` # 백업에 대한 자세한 시간을 정의
    backup_Ymd =`date + % Y- % m- % d` # 백업 디렉토리의 날짜를 정의
    backup_3ago =`date -d '3 days ago'+ % Y- % m- % d` # 3 일전의 날짜와 시간
    backup_dir = $ backup_location / $ backup_Ymd # 백업 디렉토리의 절대 경로
    welcome_msg = "Welcome to use MySQL backup tools!"# 환영 메시지

    # MYSQL이 시작하는지 판단합니다. mysql이 시작되지 않으면 백업에서 빠져 있습니다.
    mysql_ps =`ps -ef | grep mysql | wc -l`
    mysql_listen =`netstat -an | grep LISTEN | grep $ mysql_port | wc -l`
    if [$ mysql_ps == 0] -o [$ mysql_listen == 0]; then
            echo "ERROR : MySQL is not running! backup stop!"
            exit
    else
            echo $ welcome_msg
    fi

# mysql 데이터베이스에 연결합니다. 연결할 수 있어야 백업에서 빠져 있습니다.
    mysql -h $ mysql_host -P $ mysql_port -u $ mysql_user -p $ mysql_password << end
    use mysql;
    select host, user from user where user = 'root'and host = 'localhost';
    exit
    end

    flag =`echo $?`
    if [$ flag! = "0"]; then
            echo "ERROR : Can not connect mysql server! backup stop!"
            exit
    else
            echo "MySQL connect ok! Please wait ..."
            # 백업 데이터베이스가 정의되어 있는지 확인합니다. 정의되어 있으면 백업을 시작하고 그렇지 않으면 백업에서 빠져 있습니다.
            if [ "$ backup_db_arr"! = ""]; then
                    #dbnames = $ (cut -d ','-f1-5 $ backup_database)
                    #echo "arr is ($ {backup_db_arr [@]})"
                    for dbname in $ {backup_db_arr [@]}
                    do
                            echo "database $ dbname backup start ..."
                            `mkdir -p $ backup_dir`
                            `mysqldump -h $ mysql_host -P $ mysql_port -u $ mysql_user -p $ mysql_password $ dbname --default-character-set = $ mysql_charset | gzip> $ backup_dir / $ dbname- $ backup_time.sql.gz`
                            flag =`echo $?`
                            if [$ flag == "0"]; then
                                    echo "database $ dbname success backup to $ backup_dir / $ dbname- $ backup_time.sql.gz"
                            else
                                    echo "database $ dbname backup fail!"
                            fi
                            
                    done
            else
                    echo "ERROR : No database to backup! backup stop"
                    exit
            fi
            # 만료 된 백업을 삭제하도록 설정되어 있다면 삭제 작업을 수행합니다.
            if [ "$ expire_backup_delete"== "ON"-a "$ backup_location"! = ""]; then
                     #`find $ backup_location / -type d -o -type f -ctime + $ expire_days -exec rm -rf {} \;`
                     `find $ backup_location / -type d -mtime + $ expire_days | xargs rm -rf`
                     echo "Expired backup data delete complete!"
            fi
            echo "All database backup success! Thank you!"
            exit
    fi
    
shell 스크립트의 속성을 수정합니다 :
    
chmod 600 /root/mysql_backup.sh
chmod + x /root/mysql_backup.sh

속성을 설정하면 명령을 crontab에 추가합니다. 우리는 매일 00:00에 정시로 자동 백업하도록 설정 했으므로 백업 스크립트의 디렉토리 / var / www / mysql을 rsync 동기화 디렉토리로 설정합니다.

00 00 * * * /root/mysql_backup.sh

## MySQL의 복원
MySQL의 백업은 뜨거운 대기와 콜드 대기가 있다고 설명했습니다. 뜨거운 대기는 주로 실시간 복원을 실현하는 데 사용됩니다. 예를 들어, 응용 프로그램 서버에서 하드 디스크의 고장이 발생했을 경우, 설정 파일을 수정하여 데이터베이스의 읽기와 쓰기를 slave로 옮길에서 서비스 중단을 가급적 적은 시간으로 줄일 수 있습니다.

그러나 때로는 차가운 대기에 의한 백업 SQL에서 데이터를 복원해야합니다. 데이터베이스의 백업이 있으므로 명령 가져올 수 있습니다.

mysql -u username -p databse <backup.sql

데이터베이스의 데이터를 내보내거나 가져올 것은 매우 간단 하죠. 그러나 권한과 문자 인코딩 설정도 관리 할 필요가있는 경우 조금 복잡하게 될지도 모릅니다. 그러나 이들은 아무도 명령에 의해 완료 할 수 있습니다.

## redis 백업
redis는 현재 우리가 가장 자주 사용하는 NoSQL입니다. 이 백업에도 두 종류가 있습니다 : 핫 대기와 콜드 대기입니다. redis도 master / slave 모드를 지원합니다. 그래서 우리의 핫 백업은이 방법에 의해 실현 될 수 있습니다. 해당 설정은 여러분 공식 문서에있는 설정을 참고하시기 바랍니다. 매우 간단합니다. 여기에서는 콜드 대기에 대해 소개합니다. redis는 사실 메모리에 캐시 데이터를 데이터베이스 파일에 정기적으로 쓰고 있습니다. 우리의 백업은 단지 해당 파일을 복사하는 것만으로 충분합니다. 즉, 이전에 소개 한 rsync에 의해 로컬이 아닌 데이터 센터에 복사하는 것만으로 실현합니다.

## redis 복원
redis의 복원은 핫 백업과 콜드 백업으로 나눌 수 있습니다. 핫 백업의 목적 및 방법은 MySQL의 복원과 동일합니다. 응용 프로그램에서 해당 데이터베이스에 접속하는 것만으로 괜찮습니다.

그러나 때로는 콜드 백업으로 데이터를 복원해야합니다. redis 콜드 백업은 사실 저장된 데이터베이스 파일을 redis 작업 디렉토리에 복사하면됩니다. 그 redis를 시작하면 OK입니다. redis는 시작하는 동안 자동으로 데이터베이스 파일을 메모리에로드합니다. 시작 속도는 데이터베이스 파일의 대소에 의해 결정합니다.

## 정리
이 절에서는 응용 프로그램의 백업과 복원에 대해 소개했습니다. 파일 백업에서 데이터베이스 백업까지 어떻게 재해에 대응하는 하나입니다. 또한 rsync를 사용하는 다른 시스템에서 파일의 동기화에 대해서도 소개했다. MySQL 데이터베이스와 redis 데이터베이스 백업 및 복원입니다. 이 절 소개를 통해 개발 된 생산 제품의 장애에 대한 하나의 참고가되면 다행입니다.
 
