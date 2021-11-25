# SVN 서버 구축 및 백업서버 연동 (1/3)





### 먼저, SVN(SubVersioN) 은 무엇인가?

SVN은 형상/소스 관리 툴로서, 프로젝트를 진행하며 각자 개발한 코드나 산출물을 하나의 관리 툴을 이용해 **통합적**으로 **버전별**로 관리하게 도와준다. 이를 형상 관리(Configuration Management) 혹은 버전 관리(Version Management)라고 한다.

Git과 유사한 역할을 하며, SVN저장소에 파일 또는 소스를 등록하면 다른 이들이 다운받아 수정한 뒤 다시 업로드하는 형식이다.

> 현재 Apache 최상위 프로젝트로서 전세계 개발자 커뮤니티와 함께 개발되고 있다.

형상 관리 방식에는 크게, **중앙 집중식**과 **분산 관리식**이 있다.

* 중앙 집중식 : SVN
* 분산 관리식 : GIT

중앙 집중식 관리툴인 SVN은 각 개인의 local PC에서 **Commit을 하면 곧바로 원격 저장소에 반영**되기 때문에, 다른 이용자가 원격 저장소에서 자료를 받아와(Update) 가장 최신화된 상태로 관리하도록 한다.

SVN서버 저장소의 Trunk라는 곳에 프로젝트 소스가 위치하며 자신의 Local 저장소에 그 소스를 받아와(Update) 수정 및 추가 후 다시 업로드(Commit)하는 방식이다.

> Git과 SVN의 가장 큰 차이점은 SVN은 중앙 집중식으로 각 개발자가 자신만의 Version History를 가질 수 없다는 점과 Commit한 내용이 곧바로 원격 저장소에 반영되기 때문에 즉각적으로 다른 개발자에게 영향을 미칠 수 있다는 점이다.





---





### 본격적인 SVN(SubVersioN) 설치에 들어가자.



필자는 데이터 손실과 같은 사고를 대비하여 **자동 백업용 서버**를 마련하기 위해, **SVN**을 설치했다.



* 먼저, **백업용 서버 Backup Computer** (이하 **Bac_com** 이라 지칭) 에 **Linux기반 OS**를 준비했다. 

* **개발용 서버 Developing Computer** (이하 **Dev_com** 이라 지칭) 에 Linux기반의 **CentOS7**을 준비했다.

두 PC의 svn 서버 사용 용도가 다를뿐, svn 설치 과정 자체는 동일하다. 아래 설치과정대로 진행하며, 백업을 위한 추가 과정에서 부연설명하겠다.

두 서버 모두 Linux Command 환경에서 SVN구축이 진행되었음을 참고바란다. 

> Window 환경에서 Docker를 이용해 SVN구축이 가능하며 필자 역시 테스트 용도로 구축해 보았다. Docker 관련 문서 참고 바람.



#### 1. svn 설치 유무 확인 및 설치

**subversion(이하 svn)이 설치되어 있는지 식별**하고, 설치되어 있다면 `svn --version` 또는 `rpm -qa | grep subversion` 명령을 통해 버전을 확인하고, 설치되어 있지 않다면 아래 `yum` 명령을 통해 **subversion을 설치**해준다.

```bash
# 1-1. svn 설치 유무 확인
$ svn
--- svn: command not found 혹은 사용법은 'svn help'를 통해 볼 수 있습니다.
$ rpm -qa | grep subversion
--- 내용 無 혹은 subversion-1.7.14-16.el7.x86_64 subversion-libs-1.7.14-16.el7.x86_64

# 1-2. svn 설치
$ yum -y install subversion
```



#### 2. 저장소 생성

svn 설치가 완료되면, 이제 저장소를 만들어 설정값을 입력할 차례다. 원하는 **디렉토리를 선택**하고 **svn 저장소를 생성**한다. 저장소가 생성되면 **svn서버로서 기능**할 수 있게되며 생성된 저장소 내부에 **저장소의 정보와 설정을 위한 데이터**가 생성된다.

```bash
# 저장소 생성
$ svnadmin create --fs-type fsfs  저장소명
--- ex) $ svnadmin create --fs-type fsfs /home/svn/repos
--- 저장소 내부에 conf, db, hooks, locks 등의 디렉토리 및 파일이 생성된다.

참고) 저장소 삭제
$ rm -rf 저장소명
--- 저장소 또는 프로젝트를 삭제해주면 된다.
```



#### 3. svn 설정

저장소를 생성한 뒤, **사용을 위해서 설정**을 해줄 필요가 있다.

```bash
# 3-1. 사용자 접근 권한
$ cp /저장소/conf/svnserve.conf /저장소/conf/svnserve.conf.save
--- svnserve.conf 원본 copy

$ vi svnserve.conf
----------
[general]
#익명 접근의 권한 부여 : none(권한 없음), write(쓰기), read(읽기)
anon-access = none

#인증 접근의 권한은 write 읽기/쓰기
auth-access = write

#사용자 패스워드 저장 파일 위치
password-db = passwd

#프로젝트 명칭
realm = 프로젝트 명

#인증 접근의 권한 설정 파일 위치
authz-db = authz
----------
```

프로젝트 명을 저장소와 다르게 할 경우 해당 프로젝트를 기준으로 관리하게 된다.



```bash
# 3-2. 계정 설정
$ cp /저장소/conf/passwd /저장소/conf/passwd.save
--- passwd 원본 copy

$ vi passwd
----------
[users]
# 아이디 = 패스워드
user1 = password1
user2 = password2
----------

--- group 계정 설정은 다른 문서 참조.
```



```bash
# 3-3. 계정 권한 설정
$ cp /저장소/conf/authz /저장소/conf/authz.save
--- authz 원본 copy

$ vi authz
----------
[/]
user1 = rw
user2 = rw
----------
--- r:read, w:write
```



#### 4. svn 서비스 시작

svn 사용을 위한 사전작업이 완료되었다. 이제 **svn 서버를 시작**해 svn을 이용해보자.

```bash
# 4-1. svnserve 시작
$ svnserve -d -r 저장소or프로젝트 --listen-port 포트번호
--- 해당 포트번호로 지정한 저장소 또는 프로젝트 svn 실행

# 4-2. snvserve 및 포트 확인
$ netstat -anp | grep svnserve
--- svnserve가 사용하는 포트와 PID(Process ID)를 확인한다.
$ ps -ef | grep svnserve | grep -v grep
--- svnserve에 대한 정보(디렉토리 위치, 포트, PID 등)를 알 수 있다.

# 4-3. svnserve 중지
$ kill PID
--- 중지할 svnserve의 PID를 이용한다.
$ fuser -k -n tcp 포트번호
--- 특정포트를 사용하는 프로세스 죽이기.

# 4-4. 저장소 확인
$ svn info svn://localhost/저장소
--- 저장소 정보 확인
$ svn list svn://localhost/저장소
--- 저장소 내 디렉토리 목록 확인
```



#### 5. 방화벽(외부 접근) 설정

svn 서버를 시작하면 **localhost 접근이 가능**할 것이다. 그러나 기본적으로 사용 **포트의 외부 접근이 잠겨있기** 때문에, 다른 서버에서(필자의 경우, **Bac_com**)에서 대상 서버(필자의 경우, **Dev_com**)로 **접근이 불가능**하다. 이를 위해 방화벽(firewall)을 수정해 **특정 포트에 대한 외부 접근을 허용**해 주어야 한다.

```bash
# 5-1. 방화벽 확인
$ firewall-cmd --zone=public --list-all
--- 방화벽이 열려있는 포트 확인
publick (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp3s0f0
  sources:
  services: ssh dhcpv6-client
  ports: 3306/tcp 3690/tcp 8080/tcp 3307/tcp
  protocols:
  ...
--- port 목록에 포트번호가 존재하는지 확인

# 5-2. 특정 포트 방화벽 오픈(외부 접근 허용) 및 리로드
$ firewall-cmd --permanent --zone=public --add-port=3695/tcp
--- 3695 포트 tcp 방식으로 방화벽 열기(외부 접근 허용)
success

$ firewall-cmd --permanent --zone=public --remove-port=3695/tcp
--- 3695 포트 방화벽 닫기

$ firewall-cmd --reload
--- 방화벽을 리로드해주어야 설정 변경이 적용된다. 단순히 포트를 껏다 켜서는 적용되지 않는다!
success

$ firewall-cmd --zone=public --list-all
--- 방화벽이 열려있는 포트 확인
publick (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp3s0f0
  sources:
  services: ssh dhcpv6-client
  ports: 3306/tcp 3690/tcp 8080/tcp 3307/tcp 3695/tcp
  protocols:
  ...
  ...
--- port 목록에 3695/tcp가 추가된 것을 확인할 수 있다.
```



방화벽 설정시, CentOS7 이전 버전은 iptables 명령어를 사용한다.

```bash
방법1. command
# 방화벽 설정 정보 확인
$ iptables -nL

# port 설정 조회
# -L: 규칙 출력, -v: 자세히
$ iptables -L -v

# port 열기
# -i: 새로운 규칙 추가, -p: 패킷의 프로토콜을 명시, -j: 규칙에 해당되는 패킷을 어떻게 처리할지 결정
$ iptables 옵션

# TCP inbound Port 열기
# 외부 ==>> 내부 (Inbound) TCP 포트 12345를 1번 방화벽 규칙으로 추가
$ iptables -I INPUT 1 -p tcp --dport 12345 -j ACCEPT

# (포트 설정 확인 후) 변경 사항 저장 및 재시작
service iptables save
service iptables restart

방법2. 파일 수정
vi /etc/sysconfig/iptables
# 추가
-A INPUT -m state --state NEW -m tcp -p tcp --dport 12345 -j ACCEPT
```



#### 참고. /etc/sysconfig/svnserve에 저장소 등록

앞서 `4.`에서 우리는 **svnserve를 개별 조작**하는 방법을 사용했다. 좀 더 간편한 방법으로 **svnserve 전체를 시작하고 중지하는 등의 조작 명령어**가 존재한다. 아래`기타 명령어`에서 추가로 언급될 부분으로 `service svnserve start` 가 있다. 이를 위해선 **svnserve의 root위치**가 지정되어 있어야 한다. 아래 설정을 살펴보자.

```bash
$ vi /etc/sysconfig/svnserve
--- svnserve를 설정해주어야 service svnserve start/stop/restart 명령을 사용할 수 있다.
# 저장소 최상위 폴더를 /home/svn으로 하고 3695포트를 사용하도록 함
----------
OPTIONS="--threads --root /home/svn --listen-port 3695"
----------
```

svn의 root위치를 지정하는 것은 **하나의 디렉토리 안에서** 여러 프로젝트를 관리하는 경우에는 유용할 수 있다. 그러나, 각 저장소 혹은 프로젝트가 서로 다른 디렉토리에 존재할 경우에는 root위치를 지정하는 것이 무의미하다. 또한 service svnserve start/stop/restart 명령은 생성된 svn 모두를 한꺼번에 조작하는 명령으로서 다수의 svn을 사용하는 경우, **svn의 개별적 통제가 불가능**하다. 

그러므로 필자는 **svn을 개별 지정해서 실행하고 정지하는 방법을 주로 사용**한다.



#### # 기타 명령어

```bash
# 네트워크 정보 확인 (network statistics)
$ netstat [옵션]
# -a: 모든 연결에 대해 확인, -c: 실행 명령을 초단위로 재실행(default 1 sec), -l: listening 중인 포트만 확인, -t: TCP연결 포트 확인, -u: UDP 연결 포트를 확인, -n: 주소 혹은 포트 번호 확인, -p: PID/프로그램 이름 확인, -o: 연결 여부 및 시간, -r: 라우팅 테이블 확인

# 서비스 중인 포트에 연결된 사용자 확인
# 3690, 3695포트에 연결된 사용자 목록
$ netstat -anop | grep -a "3690\|3695"
# 3690, 3695포트에 연결된 사용자 수 (wc -l 은 조회된 리스트의 라인 수를 구하는 옵션) 
$ netstat -anop | grep -a "3690\|3695" | wc -l
# XXXX와 OOOO 사용자 접속 종료
$ kill -9 XXXX OOOO

# 서비스 기동 확인
$ systemctl list-unit-files | grep svn
svnserve.service				disabled
# 서비스 기동
$ systemctl enable svnserve.service

# svn 서비스 시작/중지/재시작
$ service svnserve start/stop/restart
# svn 서비스 시작/재시작/중지/상태
$ systemctl start/restart/stop/status svnserve.service

# svn 모든 서비스 종료
$ killall svnserver

# 재부팅시 svnserve 서비스 자동시작
$chkconfig --list svnserve
$chkconfig svnserve on
$chkconfig --list svnserve
```



ref) https://goddaehee.tistory.com/81

ref) https://blog.naver.com/sumni10/222177345482



