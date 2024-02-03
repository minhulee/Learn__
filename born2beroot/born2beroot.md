
# born2beroot


### 1.  설치 과정

VM : Virtual BOX (6.1.48)
OS : Debian (2024.01.28 기준 최신 버전)
Language : English
Time : Korean
Host, User id : (intra id) + 42
password : 과제에서 요구된 비밀번호 정책을 통과할 수 있도록 설정 (이후 재설정 과정을 생략하기 위함)

#### 1-1. Encrypted Disk

과제는 OS 설치 과정에서 디스크의 암호화 및 파티션의 커스텀 설정을 요구한다.

<img width="620" alt="partition" src="https://github.com/minhulee/Learn__/assets/110901252/3ab34f31-9d53-4470-9d06-41bf725943de">

해당 스크린샷과 동일한 형태로 파티션을 분리하는데 용량의 경우, 일반적으로 Virtual Box의 기본 설정인 8GB를 사용하는 경우가 많으므로 실제 머신의 용량 / 요구된 용량을 기준으로 설정한다.

이는 클러스터 내 아이맥의 환경적 문제로 인해 어쩔 수 없다...

	파티션 용량 설정
 	root (2659MB)
	swap (624MB)
	home (1358MB)
	var, srv, tmp (817MB)
	var-log(남은 용량) -> CLI상에서 var--log로 표기되나 실제 파티션 명은 var-log


	파티션 설정
	swap - Use as Swap area, 자동으로 마운트 포인트 설정
	나머지 - Ext4 journaling file System, 각 이름에 맞는 마운트 포인트 설정

이외 설치 과정에서 요구되는 사항들은 설정된 기본 설정으로 진행한다.

#### 1-2. Debian

과제는 선택한 OS에 대한 이해를 요구한다.

	1. Debian vs Rocky
 
	Debian 은 오픈 소스 커뮤니티에 의해 개발된 OS로 엄격한 테스트를 통해 stable version을 출시한다.
 	또한, 데비안은 dpkg, apt를 통해 os에 설치된 패키지를 관리한다.

	Rocky 는 레드햇 계열의 리눅스 OS로 dnf 라는 패키지 관리 도구를 사용한다.
 	또한, RHEL과의 호환성이 필요한 경우 해당 os가 유리할 수 있다.

	2. apt vs apptitude
 
	apt는 패키지 관리 도구 중 하나로 간편한 명령어를 통한 사용자 친화성이 특징이다.
	이는 Debian os의 패키지 관리를 위한 명령형 도구인 apt-get의 기능들을 통합하고 보완하여 탄생했으며 apt-get의 상위 버전이 아닌 보다
 	편리한 사용을 위한 별도의 패키지 관리 도구이다.

	apttitude는 apt와 동일한 패키지 관리 도구이나 후자에 비해 고급 기능 및 유연성을 제공한다.
	특히, apptitude는 사용자가 패키지를 관리하는 과정에서 의존성 문제를 보다 자동으로 해결해주며,
	gui를 제공할 수 있다.

	구체적으로, 패키지가 충돌하는 상황에서 apt는 충돌 패키지의 정보와 메세지를 표시하고 사용자의 조치를 대기하는 한편 apttitude는 해당
 	소프트웨어의 내부 알고리즘에 따라 탐색한 다양한 해결책을 제시하고 사용자는 해당 해결책 중 하나를 선택할 수 있다.

---


### 2. UFW (Uncomplicated FireWall) 설치 및 설정


요구되는 방화벽 설정을 위해 UFW를 설치한다.
이때, ufw는 일반적으로 sudo 권한을 필요로 한다.

ufw는 안복잡한 방화벽이라는 이름에 걸맞게 초기 설정 혹은 간단한 방화벽 규칙을 관리하는데 탁월하다.
하지만, 보다 복잡한 방화벽 규칙 및 구성을 사용하기에는 다른 소프트웨어를 사용하는 게 좋다고 한다.

```
// sudo 소프트웨어를 설치하기 이전이므로, root 계정으로 이동한다.

su -$username (생략 시 root 계정)

// apt 소프트웨어를 업데이트하고, ufw를 설치한다.
apt update
apt install ufw
```

~~root 계정에서 ufw 사용 시 커맨드를 찾지 못하는 문제가 있었다.
/usr/sbin/ufw 식으로 ufw가 존재하는 디렉토리까지 명시하는 경우 실행되곤 했으나 sudo 명령어 없이 ufw
만 입력하는 경우 커맨드를 찾지 못했다.
아마도... ufw의 path를 세팅해서 문제를 해결할 수 있겠지만, root 계정에서 ufw를 직접 세팅하는 일이 없을 
 것 같아서 넘어간다.~~


#### 2-1. UFW command (sudo 생략)


	1. ufw status (verbose)
 	UFW의 현재 상태 확인 (상세한 정보 출력)


	2. ufw allow [service name] or [port] or [port]/[method] or [ip] ...
  	UFW에 허가하는 연결을 추가한다.


	3. ufw rule delete [numb] - 1부터 시작
  	status 상의 순서 중 해당 numb 의 룰을 제거한다.


	4. ufw enable / ufw disable
 	UFW를 사용한다. / 미사용한다.


	5. systemctl status ufw`
 	별개로, UFW 가 데비안 서비스 상에서 동작중인지 확인한다.


---


### 3. sudo (Super User DO / **S**ubstitute **U**ser DO)


sudo 를 설치한다.


```
// sudo 설치 전이므로, root 계정으로 전환한다.

apt install sudo

// sudo command format

sudo [option..] [command] ..
```


직역해보자면, Super User 권한으로 command를 실행한다는 의미이다.
이는 /etc/sudoers 파일에 서술되어 있는 권한을 토대로 실행된다.

이는 일반사용자는 시스템의 핵심 부분에 접근하거나 변경할 수 없는 리눅스 시스템에서 보다 높은 권한이 필요한 작업을 실행하기 위해서 존재한다.

! 물론, 이런 작업들을 root 계정에서 직접 실행할 수도 있다.
  하지만, root 계정에서 일련의 작업들을 직접 하는 것은 악의적인 코드로 인한 보안 취약성 및 실수로 인한 데이터의 손실을 유발할 수 있고, sudo를 사용하여 작업하는 경우 작업 로깅 또한 간편하게 할 수 있다.

만약, sudo 커맨드를 사용하는 계정에 대한 인가가 서술되어 있지 않다면 해당 커맨드는 해당 계정의 권한으로 실행된다.

별도로 사용자 계정에 sudoers 파일을 생성하는 경우도 있으나, 이는 root 디렉토리의 sudoers 파일을 오버라이딩하여 보다 상세한 권한을 부여하기 위함이며 root의 sudoers 파일의 내용이 우선된다.


해당 과제에서는 강력한 보안 정책을 위해 sudoers 파일에 다음 내용을 추가해야 한다.


#### 3-1. add secure rules

	visudo : readonly인 sudoers file을 편집하기 위한 command

	1. sudo를 사용하기 위한 비밀번호의 입력은 3회로 제한된다.
 
 	Defaults passwd_tries=3

	2. sudo 사용 중 잘못된 패스워드로 인한 에러일 시 커스텀 메세지를 출력한다.
 
 	Defaults badpass_message="wrong password__"
  	Defaults authfail_message="authfail"

	3. sudo의 모든 동작(입출력)은 /var/log/sudo에 저장되어야 한다.
 
 	Defaults log_input
  	Defaults log_output
   	Defaults iolog_dir="/var/log/sudo/"

	4. 보안적인 이유로 TTY가 적용되어야 한다.
 
 	Defaults requiretty

	5. 같은 이유로 sudo의 사용 경로는 제한되어야 한다.
 
 	Defaults secure_path="${path}"
  	/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin


---


### 4. SSH 설정 및 접속


SSH(Secure Shell)는 네트워크를 통해 데이터를 전송하고 원격 시스템에 접속하기 위한 프로토콜이다.
이는 주로 원격서버를 통해 명령을 실행하거나 데이터를 주고 받는데 사용된다.

SSH의 동작 방식은 다음과 같다.

	1. 접속요청
 
	클라이언트는 서버에 접속을 요청한다.

	2. 서버 - 클라이언트 간의 인증
 
	먼저 서버와 클라이언트는 공개키 기반의 인증을 통해 인증을 통해 클라이언트의 유효성을 확인한다.

	3. 세션키 교환
 
	서버는 클라이언트와의 통신을 위해 세션키를 발급하고 교환한다.

	4. 데이터 교환
 
	서버와 클라이언트는 이전 과정에서 교환한 세션키를 토대로 데이터를 주고 받으며, 이는 해당 세션에서만 유효한 임시키이므로 통신 과정에서
 	공격자가 데이터를 가로채거나 변경하는 것이 거의 불가능하다.

	5. 세션 종료
 
	SSH 프로토콜을 통한 원격 접속이 종료될 때 해당 세션키는 일반적으로 삭제된다.

이런 과정을 통해 SSH는 다음과 같은 특징을 가지게 된다.

	1. 안전한 통신
 
	SSH 프로토콜은 공개키 기반의 인증을 통해 통신 채널의 보안성을 확보하고, 해당 통신 간에만 유효한 세션키를 통해서 주고 받는 데이터의
 	기밀성을 확보한다.
	통신 과정에서 두 번의 보안 과정을 통해 높은 안정성을 확보할 수 있고, 세션키에 대해서는 서버 - 클라이언트 간에 협의된 다양한 알고리즘을
 	채택할 수 있다.

	2. 인증과 권한 부여
 
	 SSH 프로토콜은 공개키 기반의 인증으로 통신 채널을 확정하는 과정에서 클라이언트의 유효성을 검사할 수 있고 이 과정에서 해당 사용자의
  	접근 권한 등에 대해서 별도의 설정이 가능하다.

	3. 포트 포워딩
 
	SSH 프로토콜은 포트 포워딩을 지원하며, 서버 - 클라이언트는 특정 포트를 통해 데이터를 주고 받을 수 있다. 이는 로컬 혹은 원격의 다양한
 	서비스(각 서비스에 사용하는 포트)에 안전하게 접근할 수 있다.

	4.  부가적인 기능들
 
	SSH 프로토콜은 단순히 해당 프로토콜을 통한 데이터의 주고 받음 뿐만이 아니라 원격 접속, 파일 전송(SCP, SFTP), 원격 포트 포워딩 등의
 	다양한 기능을 포함하고 있다.

특히, 세션키에 대해 다양한 알고리즘을 지원하는 점에서 서버는 보안에 비교적 취약한 알고리즘을 차단할 수 있다.

과제에서는 SSH 연결 간에 다음과 같은 사항들을 요구한다.

	1. root 계정을 통한 ssh 접속을 거부한다.
 
	Permitrootlogin no

	2. 연결은 4242 port를 사용한다.
 
	Port 4242


해당 설정은 /etc/sshd_config 를 통해 설정 할 수 있다.
ssh와 sshd의 차이점은 ssh client, ssh server를 의미하며 d는 백그라운드 소프트웨어인 deamon을 의미한다.

이때, 과제를 수행하는 환경은 SSH 프로토콜인 기본 포트인 22만 허가되어 있으므로 4242포트로 접속하기 위해서는 virtual box의 포트포워딩을 통해 Host os의 접속 포토를 4242로 변경해줘야 한다.

또한, 포트포워딩 설정 시 Host os에서 점유되지 않은 포트번호를 host port로 사용해야 한다.


---


### 5. Password Policy


과제에서는 강력한 보안을 위해 다음과 같은 비밀번호 정책이 적용되기를 요구한다.

	1. 패스워드는 매 30일마다 만료된다.

	2. 패스워드는 변경 이후 2일이 초과되어야 재변경이 가능하다.

	3. 유저는 패스워드 만료 7일 이전에 경고메세지를 받아야 한다.

	4. 패스워드는 대문자, 소문자, 숫자를 포함하고 10글자 이상이어야 한다.

	6. 패스워드는 반복되는 글자가 3글자를 초과할 수 없다.

	7. 패스워드는 유저의 이름을 포함할 수 없다.

	8. 현재 패스워드는 최소 7자 이상 이전 패스워드와 달라야 한다. (root 계정 제외)

	9. (7.)을 제외한 모든 규정은 root 계정에도 적용되어야 한다.

해당 정책을 설정하기 위해 먼저 /etc/login.defs 파일을 수정한다.


```
# /etc/login.defs

PASS_MAX_DAYS 30 // 패스워드의 최대 유효기간
PASS_MIN_DAYS 2  // 패스워드 재설정 최소 간격
PASS_WARN_AGE 7  // 패스워드 만료 경고 날짜
```

데비안은 사용자 인증을 관리하기 위한 기본 인터페이스로 pam 모듈을 채택하고 있다.

그 중, 패스워드 품질 향상을 위해 존재하는 모듈인 pwquality 모듈을 이용해서 설정을 진행하겠다.


```
// pwquality 모듈 설치
apt install libpam-pwquality

// pwquality.conf 설정 - 모듈 설치 시 기본 설정 파일이 제공된다.
// /etc/security/pwquality.conf

// passwd expiration
difork = 7.         // 이전 패스워드와 유사성 비교
minlen = 10         // 패스워드 최소 길이
dcredit = -1        // 숫자 포함
ucredit = -1        // 영대문자 포함
lcredit = -1        // 영소문자 포함
maxrepeat = 3       // 반복 최대 횟수
retry = 3           // 입력 시도 최대 횟수
enforce_for_root    // 해당 설정 root 계정에도 적용
```

#### 5-1. 사용자에 관련된 명령어 (과제에서 필요한 옵션에 대해서만)

	1. 사용자 추가
	adduser [id]

	2. 사용자 제거
	deluser [id] [option]
		--remove-home : 해당 계정의 홈 디렉토리 제거

	3. 기존 계정의 패스워드 설정 변경하기
	passwd [option] [config] [id]
		-S : 패스워드 설정 조회
		--mindays : 최소 변경 간격
		--maxdays : 최대 유효기간
		--warndays : 경고 날짜

		 ex : passwd --mindays=2 --maxlen=30 --warndays=2 id

	4. 사용자에 그룹 추가
	usermod [option] [id]
		 -G : 사용자의 그룹을 지정 그룹으로 덮어쓴다.
		 -a : 해당 정보를 추가한다(덮어쓰는 게 아닌).

	5. 사용자에 그룹 제거
	gpasswd [option] [id] [group]
		 -a : 사용자에 해당 그룹을 추가한다.
		 -d : 사용자를 해당 그룹에서 제거한다.
		
	6. 그룹 제거
	groupdel [group]
 

---


### 6.  Monitering.sh 와 cron


...ing





