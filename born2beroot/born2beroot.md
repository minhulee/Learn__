
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


---


### 2. UFW (Uncomplicated FireWall) 설치 및 설정


요구되는 방화벽 설정을 위해 UFW를 설치한다.
이때, ufw는 일반적으로 sudo 권한을 필요로 한다.

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
 	- UFW의 현재 상태 확인 (상세한 정보 출력)


	 2. ufw allow [service name] or [port] or [port]/[method] or [ip] ...
  	- UFW에 허가하는 연결을 추가한다.


	 3. ufw rule delete [numb] - 1부터 시작
  	- status 상의 순서 중 해당 numb 의 룰을 제거한다.


	4. ufw enable / ufw disable
 	- UFW를 사용한다. / 미사용한다.


	5. systemctl status ufw`
  	- 별개로, UFW 가 데비안 서비스 상에서 동작중인지 확인한다.


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

만약, sudo 커맨드를 사용하는 계정에 대한 인가가 서술되어 있지 않다면 해당 커맨드는 해당 계정의 권한으로 실행된다.

별도로 사용자 계정에 sudoers 파일을 생성하는 경우도 있으나, 이는 root 디렉토리의 sudoers 파일을 오버라이딩하여 보다 상세한 권한을 부여하기 위함이며 root의 sudoers 파일의 내용이 우선된다.


해당 과제에서는 강력한 보안 정책을 위해 sudoers 파일에 다음 내용을 추가해야 한다.


#### 3-1. add secure rules

	visudo : readonly인 sudoers file을 편집하기 위한 command

	1. sudo를 사용하기 위한 비밀번호의 입력은 3회로 제한된다.
	- Defaults passwd_tries=3

	2. sudo 사용 중 잘못된 패스워드로 인한 에러일 시 커스텀 메세지를 출력한다.
	- Defaults badpass_message="wrong password"
	- Defaults authfail_message="authfail"

	3. sudo의 모든 동작(입출력)은 /var/log/sudo에 저장되어야 한다.
	- Defaults log_input
	- Defaults log_output
	- Defaults iolog_dir="/var/log/sudo/"

	4. 보안적인 이유로 TTY가 적용되어야 한다.
	- Defaults requiretty

	5. 같은 이유로 sudo의 사용 경로는 제한되어야 한다.
	- Defaults secure_path="${path}"
	- /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin


---


### 4. SSH 설정 및 접속


-ing...
