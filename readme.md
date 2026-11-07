# 🐳 E1-1  내 컴퓨터에 개발자용 '작업실' 꾸미기 



---
        
## 1. 프로젝트 개요



**Codyssey 올인원 1기 입학연수 E1-1 미션입니다.**

Flask 기반 방명록 웹 애플리케이션을 Docker로 컨테이너화하고,  
포트 매핑 / 바인드 마운트 / 볼륨 영속성 등을 실습한 프로젝트입니다.

### 미션 목표
- `Dockerfile`을 작성하여 커스텀 이미지 빌드
- `포트 매핑`으로 컨테이너 2개 동시 실행
- `바인드 마운트`로 호스트 코드 변경 즉시 반영
- Docker `볼륨`으로 컨테이너 삭제 후에도 데이터 유지
- `Git` 설정 및 GitHub/VSCode 연동
    



---

## 2. 실행 환경

- OS: Windows 11 pro / Ubuntu 26.04.1 LTS
- Shell: PowerShell 5.1 / GNU bash, version 5.3.9
- Docker: 29.9.2
- Git: 2.56.0




---
## 3. 사전 준비 : Ubuntu 설치 
5-1, 5-2 실습을 수행하기 위해 Ubuntu를 설치합니다.  

### 📌 설치 이유
- Git Bash는 NTFS 파일시스템 기반 → `chmod` 권한 변경 불가
- Ubuntu (WSL3)는 ext4 파일시스템 → `chmod` 실습 가능

### 🔧 설치 방법


**PowerShell(관리자 권한)에서 다음 작업 진행**

1️⃣ Ubuntu 설치   
```PowerShell
# PowerShell에서 작업
$ wsl --install -d Ubuntu
다운로드 중: Ubuntu
설치 중: Ubuntu
배포가 설치되었습니다.
Ubuntu 시작하는 중...
Provisioning the new WSL instance Ubuntu
This might take a while...
Create a default Unix user account: redna
New password:
Retype new password:
passwd: password updated successfully

# 설치완료 후 자동으로 Ubuntu Bash Shell로 진입
# exit 입력 후 PowerShell로 돌아와 다음 단계 진행
redna@js:/mnt/c/WINDOWS/System32$ : exit
```


2️⃣ 설치 확인

```PowerShell
# PowerShell에서 작업
$ wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

3️⃣ Ubuntu 실행
```PowerShell
# PowerShell에서 작업
$ wsl -d Ubuntu
redna@js:/mnt/c/WINDOWS/System32$
```

⚠️ 문제 발생 시: Ubuntu 재설치  
Ubuntu를 완전히 제거하고 다시 설치하려면:

```PowerShell
# PowerShell에서 작업

# 1. Ubuntu 완전 제거 (데이터 포함)
$ wsl --unregister Ubuntu

# 2. 다시 설치
$ wsl --install -d Ubuntu

# 3. 설치 확인
$ wsl -l -v
```








___

## 4. 수행 체크리스트

- [x] 터미널 기본 조작 및 폴더 구성
- [x] 권한 변경 실습
- [x] Docker 설치/점검
- [x] hello-world 실행
- [x] Dockerfile 빌드/실행
- [x] 포트 매핑 접속(2회)
- [x] 바인드 마운트 반영
- [x] 볼륨 영속성
- [x] Git 설정 + VSCode GitHub 연동

___

## 5. 수행로그 


### 5-1 터미널 기본 조작 및 폴더 구성


```bash
# [PowerShell] Ubuntu 진입
$ wsl -d Ubuntu

# Ubuntu Bash Shell 진입 확인
redna@js:/mnt/c/WINDOWS/System32$

# Home 폴더로 이동
redna@js:/mnt/c/WINDOWS/System32$ cd ~
redna@js:~$

# 이 밑으로는 "redna@js:~$" 대신 "$" 로만 표기
```


```bash
# ── 터미널 조작 로그 기록 ────────────────────────────

# 1. 현재 위치 확인
$ pwd
/home/redna

# 2. 목록 확인 (숨김 파일 포함)
$ ls -al
total 32
drwxr-x--- 4 redna redna 4096 Nov  5 15:27 .
drwxr-xr-x 3 root  root  4096 Nov  5 15:25 ..
-rw------- 1 redna redna   50 Nov  5 16:24 .bash_history
-rw-r--r-- 1 redna redna  220 Nov  5 15:25 .bash_logout
-rw-r--r-- 1 redna redna 3771 Nov  5 15:25 .bashrc
drwxr-x--- 4 redna redna 4096 Nov  5 15:25 .cache
drwxr-x--- 3 redna redna 4096 Nov  5 15:25 .config
-rw-rw-r-- 1 redna redna    0 Nov  5 15:25 .motd_shown
-rw-r--r-- 1 redna redna  807 Nov  5 15:25 .profile

# 3. 폴더 생성
$ mkdir practice

# 4. 폴더 이동
$ cd practice
$ pwd
/home/redna/practice

# 5. 빈 파일 생성
$ touch file_A.txt
$ ls
file_A.txt

# 6. 파일에 내용 쓰기 (touch로 만든 파일에 덮어쓰기)
$ echo "파일 A입니다." > file_A.txt

# 7. 파일 내용 확인
$ cat file_A.txt
파일 A입니다.

# 8. 파일 복사
$ cp file_A.txt file_A_copy.txt
$ ls
file_A.txt  file_A_copy.txt
$ cat file_A_copy.txt
파일 A입니다.

# 9. 파일 이름 변경
$ mv file_A_copy.txt file_B.txt
$ ls
file_A.txt  file_B.txt

# 10. 파일 이동 (폴더 B를 만들고 file_B를 이동)
$ mkdir B
$ mv file_B.txt B/file_B.txt
$ ls
B  file_A.txt
$ ls B
file_B.txt

# 11. 파일 삭제
$ rm file_A.txt
$ ls
B

# 12. 폴더 삭제
$ rm -r B
$ ls
# 아무것도 출력되지 않음 = 모든 파일·폴더 삭제 완료 ✅
```




### 5-2 권한 변경 실습

실무에서는 데이터베이스 비밀번호나 서버 설정처럼 민감한 정보를 다룰 때 권한 관리가 필수적입니다. 보안 사고를 예방하기 위해 `최소 권한 원칙`에 따라 실습을 진행했습니다.


참고: 리눅스 권한(Permission) 계산 규칙

리눅스 권한은 읽기(r), 쓰기(w), 실행(x) 비트의 값을 합산하여 숫자로 표기합니다.

| 권한 비트 | 숫자값 | 의미 |
|:---:|:---:|:---|
| **r** (read) | **4** | 파일 읽기 권한 |
| **w** (write) | **2** | 파일 수정/쓰기 권한 |
| **x** (execute) | **1** | 파일 실행/디렉토리 접근 권한 |

**[합산 예시]**
- **7** (4+2+1) : **rwx** (모든 권한 허용)
- **6** (4+2+0) : **rw-** (읽기/쓰기 가능)
- **5** (4+0+1) : **r-x** (읽기/실행 가능)
- **4** (4+0+0) : **r--** (읽기만 가능)
- **0** (0+0+0) : **---** (권한 없음)


```bash
# ── 권한 실습 및 증거 기록 ────────────────────────────────────

# 실습 준비
$ mkdir secure_configs
$ echo "DB_PASSWORD=super-secret-1234" > db_password.txt
$ ls
db_password.txt  secure_configs

# 1️⃣ 변경 전 권한 확인
# 기본 권한은 보통 소유자/그룹/기타 사용자 모두에게 읽기 권한이 열려 있습니다.
$ ls -l
total 8
-rw-r--r-- 1 redna redna   30 Nov  7 19:48 db_password.txt
drwxr-xr-x 2 redna redna 4096 Nov  7 19:48 secure_configs

# 2️⃣ 권한 변경 - 숫자 표기법 (chmod 숫자)

# [의도] secure_configs: 설정 폴더는 관리자(소유자) 외에 누구도 들어오거나 목록을 볼 수 없어야 합니다.
$ chmod 700 secure_configs
$ ls -ld secure_configs
drwx------ 2 redna redna 4096 Nov  7 19:48 secure_configs

# [의도] db_password.txt: 비밀번호 파일은 소유자만 읽고 수정할 수 있도록 제한하여 유출을 방지합니다.
$ chmod 600 db_password.txt
$ ls -l db_password.txt
-rw------- 1 redna redna 30 Nov  7 19:48 db_password.txt


# 3️⃣ 권한 변경 - 심볼릭 표기법 (chmod u/g/o)

# [의도] secure_configs: 특정 서비스 계정이 폴더 내 파일에 접근해야 할 경우를 대비해 '실행(x)' 권한만 개방합니다.
$ chmod o+x secure_configs
$ ls -ld secure_configs
drwx-----x 2 redna redna 4096 Nov  7 19:48 secure_configs

# [의도] db_password.txt: 협업하는 팀원(그룹)들이 설정을 읽고 배포할 수 있도록 '읽기(r)' 권한을 추가합니다.
$ chmod g+r db_password.txt
$ ls -l db_password.txt
-rw-r----- 1 redna redna 30 Nov  7 19:48 db_password.txt
```


### 5-3 Docker 설치 및 기본 점검

```bash
# ── Docker 설치 ───────────────────────────────────────────

# 1️⃣ Docker 설치
# 패키지 목록 업데이트
$ sudo apt update

# Docker 설치
$ sudo apt install -y docker.io
Installing:
  docker.io
...
Processing triggers for man-db (2.13.1-1build1) ...
Processing triggers for dbus (1.16.2-2ubuntu4) ...
Processing triggers for libc-bin (2.43-2ubuntu2.3) ...

# 2️⃣ Docker 버전 확인
$ docker --version
Docker version 29.1.3, build 29.1.3-0ubuntu4.1
```

```bash
# ── Docker 실행 준비 ───────────────────────────────────

# 3️⃣ Docker 데몬 시작 및 자동 시작 설정 (Standard Linux)
# (데몬: 백그라운드에서 계속 돌아가는 프로그램)
# [의도] start는 현재 실행, enable은 부팅 시 자동 실행을 의미합니다.
$ sudo systemctl start docker
$ sudo systemctl enable docker

# 서비스 상태 확인 (정상 동작 및 자동 시작 설정 여부)
# [의도] Active: active (running) 상태와 'enabled' 설정을 확인합니다.
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-11-07 20:02:09 KST; 9min ago
 Invocation: 1d674770e3a246ed9a29d2ba040d70ec
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 327 (dockerd)
      Tasks: 15
     Memory: 86M (peak: 115.1M)
        CPU: 363ms
     CGroup: /system.slice/docker.service
             └─327 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
...

# ⚠️ [WSL 환경 주의사항]
# 만약 WSL에서 Docker Desktop을 사용 중이라면, 위 명령어 대신 
# Docker Desktop 설정(Settings > General)에서 
# 'Start Docker Desktop when you log in'이 체크되어 있어야 합니다.


# [문제 해결] 서비스 이상 발생 시 조치 명령어
# 서비스 재시작
$ sudo systemctl restart docker

# 에러 로그 확인 (문제가 생겨 실행되지 않을 때 원인 파악용)
$ sudo journalctl -u docker
Nov 05 17:12:03 js systemd[1]: Starting docker.service - Docker Application Container Engine...
...
Nov 05 17:29:24 js dockerd[3588]: time="2026-11-05T17:29:24.753108121+09:00" level=info msg="Starting up"
Nov 05 17:29:24 js dockerd[3588]: time="2026-11-05T17:29:24.754123447+09:00" level=info msg="OTEL tracing is not config>
Nov 05 17:29:24 js dockerd[3588]: time="2026-11-05T17:29:24.754205683+09:00" level=info msg="CDI directory does not exi>
lines 1-29



# 4️⃣ 최종 설치 확인
$ sudo docker info
Client:
 Version:    29.1.3
 Context:    default
 Debug Mode: false
 Plugins:
  trust: Manage trust on Docker images (Docker Inc.)
    Version:  29.1.3
    Path:     /usr/libexec/docker/cli-plugins/docker-trust

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
...
...
...


# ------------------------------------------
# [선택사항] 매번 sudo 안붙이려면
# 사용자를 "docker" group에 넣어주면 된다.
$ sudo usermod -aG docker $USER

# 바뀐 그룹 권한 적용 위해 터미널 종료 후 다시 열기
# 또는 다음 명령어 실행
$ exit

# 이제 sudo 없이 docker 명령어 사용 가능
$ docker info
Client:
 Version:    29.1.3
 Context:    default
 Debug Mode: false
 Plugins:
  trust: Manage trust on Docker images (Docker Inc.)
    Version:  29.1.3
    Path:     /usr/libexec/docker/cli-plugins/docker-trust

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
...
```




### 5-4  Docker 기본 운영 명령 수행

`hello-world` 실습을 통해 Docker의 정상 동작을 확인하며, 실행 시점의 타임스탬프를 기록하여 수행의 추적성을 확보합니다.

```bash
# hello-world로 Docker 기본 동작 확인

# [추적성 확보] 실행 시점의 날짜와 시간 기록
$ date
Sat Nov  7 20:37:19 KST 2026


# 1️⃣ 이미지 다운로드 & 컨테이너 실행
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
...

# 2️⃣ 다운로드된 이미지 확인
$ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE
hello-world:latest   c567b231f656       25.9kB         9.49kB

# 3️⃣ 실행 중인 컨테이너 확인 (hello-world는 실행 후 즉시 종료됨)
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# 4️⃣ 종료된 컨테이너까지 전체 확인 (실행/종료 시간 정보 포함)
$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     NAMES
16e10cf5fd07   hello-world   "/hello"   4 minutes ago   Exited (0) 4 minutes ago   inspiring_wright
```

![hello-world 실행화면](img/hello-world%20실행화면.png)


### 5-5 컨테이너 실행 실습

```bash
# 1️⃣ hello-world 실행 
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
...
...
For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

```bash
# 2️⃣ Ubuntu 컨테이너 실행 + 진입
$ docker run -it my-ubuntu ubuntu bash  




# -------컨테이너 내부 진입----------

# 컨테이너 내부에서 간단 명령(ls, echo) 수행결과
$ ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

$ echo "Hello from container"
Hello from container

# 컨테이너 종료
$ exit

# -------컨테이너 내부 탈출----------




# 3️⃣ 로그 확인
$ docker logs my-ubuntu
root@505a8db31a50:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@505a8db31a50:/# echo "Hello from containder"
Hello from containder
root@505a8db31a50:/# echo "Hello from container"
Hello from container
root@505a8db31a50:/# exit
exit

# 4️⃣ 리소스 확인
$ docker stats my-ubuntu
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT   MEM %     NET I/O   BLOCK I/O   PIDS
505a8db31a50   my-ubuntu   0.00%     0B / 0B             0.00%     0B / 0B   0B / 0B     0
```



### 5-6 컨테이너 관리 (attach, exec, 종료)

```bash
# 1️⃣ 백그라운드 실행
$ docker run -it -d --name my-ubuntu ubuntu bash
1c8bea5cd7a5ef81bd7a7dd5793934f0e5887001d538efda87c2daeb1f11da9f

$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
04bc555411c4   ubuntu    "bash"    30 seconds ago   Up 29 seconds             my-ubuntu
```

```bash 
# ── attach vs exec ────────────────────────────────────

# 2️⃣ docker attach (재접속)
# - 실행 중인 컨테이너의 메인 프로세스에 접속
# - exit 하면 컨테이너도 종료됨 ⚠️
# - 이미 종료된 컨테이너는 불가

$ docker attach my-ubuntu
# -------컨테이너 내부 진입----------
root@xxxxx:/# ls
root@xxxxx:/# pwd
root@xxxxx:/# exit
# -------컨테이너 내부 탈출----------

# exit 하면 컨테이너도 종료됨
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# ==================================

# 3️⃣ docker exec (명령 실행)
# - 새로운 세션에서 명령 실행
# - exit 해도 컨테이너는 유지됨 ✅
# - 실행 중인 컨테이너에 추가 작업할 때 사용

$ docker start my-ubuntu
$ docker exec -it my-ubuntu bash
# -------컨테이너 내부 진입----------
root@xxxxx:/# ls
root@xxxxx:/# pwd
root@xxxxx:/# exit
# -------컨테이너 내부 탈출----------

# exit 해도 컨테이너는 유지됨
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
04bc555411c4   ubuntu    "bash"    23 minutes ago   Up 10 minutes     
```


### 5-7 기존 Dockerfile 기반 커스텀 이미지 제작

#### 1️⃣ 베이스 이미지 선택

선택 : python:3.11-slim

선택이유 
```
✅ Python 3.11 사전 설치 (pip 포함)
✅ slim 버전으로 이미지 크기 최소화 (~150MB)
✅ 패키지 관리자(apt) 포함으로 추가 도구 설치 가능
✅ Flask 웹 서버 실행에 최적화
```

대안 비교

|이미지|	크기	|특징 |
|---|---|---|
|python:3.11|	~900MB|	개발용 (무거움)|
|python:3.11-slim|	~150MB|	프로덕션 ⭐|
|python:3.11-alpine|	~50MB	|극도로 경량|
|ubuntu:24.04|	~80MB	범용| 커스터마이징 많을 때|



#### 2️⃣ Dockerfile 작성
```docker
# 베이스 이미지 선택
FROM python:3.11-slim

# 환경변수 설정
ENV FLASK_APP=app.py    
ENV APP_HOME=/app

# 시스템 패키지 설치 (vi 편집기)
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 작업 디렉토리 설정
WORKDIR ${APP_HOME}

# 앱 코드 복사
COPY requirements.txt .
COPY . .

# Python 패키지 설치
RUN pip install --no-cache-dir -r requirements.txt

# 포트 노출
EXPOSE 5000

# 헬스체크
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# 서버 실행
CMD ["python", "app.py"]
```

#### 3️⃣ 커스텀 포인트 및 목적
|포인트	| 목적 |
|---|---|
|환경변수 설정	| 변수 사용으로 Dockerfile 유지보수성 향상 |
|시스템 패키지 설치 |	curl (헬스체크)|
|작업 디렉토리 설정 |	모든 명령을 /app 에서 실행|
|앱 코드 복사	| 호스트 애플리케이션 파일을 이미지에 포함|
|Python 패키지 설치 |	Flask 라이브러리 설치|
|포트 노출	|Flask 포트(5000) 문서화|
|헬스체크	|컨테이너 상태 자동 모니터링|

#### 4️⃣ 기존 Dockerfile 기반 커스텀 이미지 제작 작업 로그

```bash
# ── Git Clone 및 프로젝트 준비 ────────────────────────

# github에서 프로젝트 코드 가져오기 
$ git clone https://github.com/Genie1486/guestbook.git
Cloning into 'guestbook'...
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 10 (delta 0), reused 10 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (10/10), done.

# 이미지 파일 빌드 하려면 dockerfile 있는 폴더로 진입해야함
$ cd guestbook  
$ ls
Dockerfile  app.py  data  requirements.txt  templates

# 프로젝트 디렉토리 구조 확인
$ tree
guestbook/
├── Dockerfile          # Docker 이미지 빌드 설정 파일
├── app.py              # Flask 웹 애플리케이션 메인 코드
├── requirements.txt    # Python 패키지 의존성 목록
├── data/               # 방명록 데이터 저장 디렉토리 (볼륨 마운트 대상)
│   └── guestbook.json  # 방명록 데이터 파일 (JSON 형식)
└── templates/          # HTML 템플릿 디렉토리 (Flask 렌더링 대상)
    └── index.html      # 방명록 메인 페이지 템플릿

# [설계 기준]
# 1. app.py     : 애플리케이션 로직만 담당, 데이터·화면과 분리
# 2. data/      : 데이터만 격리 보관 → 볼륨(-v) 마운트로 영속성 확보
# 3. templates/ : 화면(HTML)만 격리 → 로직 변경 없이 UI 수정 가능
# 4. Dockerfile : 빌드 설정을 코드로 관리 → 누구나 동일 환경 재현 가
```

```bash
# ── 이미지 빌드 ──────────────────────────────────────────
$ docker build -t guestbook:1.0 .
[+] Building 4.7s (12/12) FINISHED                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 686B                                                                               0.0s
 => [internal] load metadata for docker.io/library/python:3.11-slim                                                1.4s
 => [auth] library/python:pull token for registry-1.docker.io                                                      0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 70B                                                                                   0.0s
 => [1/6] FROM docker.io/library/python:3.11-slim@sha256:65bd3d9f1d7fa268dd1909973ba7eb4bdf41f6dcf2cfbe76ba9528b1  0.0s
 => => resolve docker.io/library/python:3.11-slim@sha256:65bd3d9f1d7fa268dd1909973ba7eb4bdf41f6dcf2cfbe76ba9528b1  0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 3.20kB                                                                                0.0s
 => CACHED [2/6] RUN apt-get update && apt-get install -y     curl     && rm -rf /var/lib/apt/lists/*              0.0s
 => CACHED [3/6] WORKDIR /app                                                                                      0.0s
 => CACHED [4/6] COPY requirements.txt .                                                                           0.0s
 => [5/6] COPY . .                                                                                                 0.0s
 => [6/6] RUN pip install --no-cache-dir -r requirements.txt                                                       2.3s
 => exporting to image                                                                                             0.8s
 => => exporting layers                                                                                            0.5s
 => => exporting manifest sha256:a00d74a60422049651da1863f6018a685ba6bb32c038013d4ebd593ae8eab9a9                  0.0s
 => => exporting config sha256:79f69b2ace8fd4f57df84dfcbeb45f6cc259441487d1cc02f81039a709c74c21                    0.0s
 => => exporting attestation manifest sha256:f5467342d0e71cd3166e50cec5045bb862c9a90c4ed09c81ce01f77ebe0b1466      0.0s
 => => exporting manifest list sha256:f57fab6eb2a249632a3d94602aed07763e7ccdbf1be3be73452dda5c72e87ca3             0.0s
 => => naming to docker.io/library/guestbook:1.0                                                                   0.0s
 => => unpacking to docker.io/library/guestbook:1.0      

# 빌드 잘 되었는지 확인
$ docker images
IMAGE              ID             DISK USAGE   CONTENT SIZE   EXTRA
guestbook:1.0      beca38b24f4c        234MB         56.9MB
```

🛠️ 빌드 실패 시 주요 점검 항목 (Troubleshooting Guide)  
이미지 빌드 중 오류가 발생할 경우, 다음 항목을 순차적으로 점검합니다.

| 점검 항목 | 확인 내용 | 해결 방법 |
|:---|:---|:---|
| **Dockerfile 경로** | 현재 디렉토리에 `Dockerfile` 파일이 있는지 확인 | `ls` 명령어로 파일명(대소문자) 확인 |
| **빌드 컨텍스트** | `COPY`하려는 파일(예: requirements.txt)이 실제 경로에 있는지 확인 | 파일 경로 수정 및 `.dockerignore` 확인 |
| **베이스 이미지** | `FROM`에 작성된 이미지 이름과 태그가 정확한지 확인 | Docker Hub에서 정확한 이미지명 검색 |
| **네트워크 오류** | `RUN apt-get` 또는 `pip install` 중 연결이 끊기지 않는지 확인 | 인터넷 연결 확인 및 DNS 설정 점검 |
| **문법 오류** | Dockerfile 내 명령어(RUN, COPY, EXPOSE 등) 오타 확인 | Dockerfile 린트(Lint) 체크 |

> 💡 **Tip:** 빌드 과정에서 캐시 때문에 오류가 반복된다면 `--no-cache` 옵션을 사용하여 처음부터 다시 빌드할 수 있습니다.
> ```bash
> $ docker build --no-cache -t guestbook:1.0 .
> ```


```bash
# ── 컨테이너 실행 및 동작 점검 ───────────────────────────

# 컨테이너 실행
$ docker run -d --name  guestbook-app guestbook:1.0
2d467bfb2a3a9c6a1162589b2b3b83388f757a3e2719e0ab24e8db899bdf2d3b

$ docker exec -it guestbook-app bash



# -------컨테이너 내부 진입------------------------------
root@xxxxx:/# curl http://localhost:5000

# app 잘 돌고 있는지 확인
root@xxxxx:/# curl http://localhost:5000/health
{
  "status": "ok"
}

root@xxxxx:/# exit
# -------컨테이너 내부 탈출--------------------

# 포트포워딩이 되어있지 않아 컨테이너 밖에서는 접근 불가능
$ curl http://localhost:5000/health
curl: (7) Failed to connect to localhost port 5000 after 0 ms: Could not connect to server                                         
```

브라우저 접속화면
![포트포워딩 전](img//포트포워딩%20전%20브라우저%20접속화면.png)


> #### 🔒 [개념] 컨테이너 네트워크 격리와 포트 포워딩 원리
>
> ##### 1. 네트워크 네임스페이스란?
>
> Docker는 리눅스 커널의 **네트워크 네임스페이스(Network Namespace)** 기술로
> 컨테이너마다 독립된 네트워크 환경을 만듭니다.
> 아파트에 비유하면, 같은 건물(호스트)에 살지만 각 호수(컨테이너)는
> 서로의 공간에 마음대로 들어갈 수 없는 것과 같습니다.
>
> ```
> 호스트 PC
> ├── IP: 192.168.0.10
> ├── 포트: 80, 443, 8080 ...
> │
> ├── 컨테이너 A (네트워크 네임스페이스 A)
> │     ├── IP: 172.17.0.2   ← 호스트랑 다른 IP!
> │     └── 포트: 5000       ← 호스트 포트랑 별개!
> │
> └── 컨테이너 B (네트워크 네임스페이스 B)
>       ├── IP: 172.17.0.3
>       └── 포트: 5000       ← A랑 같은 포트 써도 충돌 없음!
> ```
>
> ##### 2. 그래서 왜 포트 포워딩이 필요한가?
>
> 컨테이너 내부 포트는 **격리된 공간 안에만 존재**하므로
> 호스트에서 직접 접근할 수 없습니다.
>
> ```
> ❌ 포트포워딩 없을 때
> 브라우저 → localhost:5000 → 🚫 호스트 5000번엔 아무것도 없음
>                                  (컨테이너 5000번은 격리된 공간 안에 있어서 못 찾음)
>
> ✅ 포트포워딩 있을 때 (-p 8080:5000)
> 브라우저 → localhost:8080 → Docker가 연결해줌 → 컨테이너:5000 → 🎉
> ```
>
> **포트 포워딩 = 격리된 공간에 창문을 내는 것**
>
> ✅ 5-7에서 포트포워딩 없이 실행했을 때 접근이 안 됐던 이유가 바로 이것입니다!

---


### 5-8 포트 매핑 접속(2회)

```bash
# ── 포트 매핑 접속 (8080 / 8081) ─────────────────────────

# 8080 포트로 접근할 수 있는 컨테이너 만들기
$ docker run -d -p 8080:5000 --name guestbook-app-8080 guestbook:1.0
5e199748517ef78227884d4d8ba441ff36e2b340f7946f2222cf4716f2be2bb2

# 8080 포트 실행 확인
$ curl http://localhost:8080/health
{
  "status": "ok"
}

# 8081 포트로 접근할 수 있는 컨테이너 다시 만들기
$ docker run -d -p 8081:5000 --name guestbook-app-8081 guestbook:1.0
d7e6741d011dee4987ad8df3eb612c1895e3e405fa275d4344e3c709dd547dc7

# 8081 포트 실행 확인
$ curl http://localhost:8081/health
{
  "status": "ok"
}

# 컨테이너 상태 조회
$ docker ps
CONTAINER ID   IMAGE           COMMAND           CREATED              STATUS                        PORTS                                         NAMES
d7e6741d011d   guestbook:1.0   "python app.py"   About a minute ago   Up About a minute (healthy)   0.0.0.0:8081->5000/tcp, [::]:8081->5000/tcp   guestbook-app-8081
5e199748517e   guestbook:1.0   "python app.py"   5 minutes ago        Up 5 minutes (healthy)        0.0.0.0:8080->5000/tcp, [::]:8080->5000/tcp   guestbook-app-8080
```

포트 매핑 후 브라우저 접속 기록 캡처 (8080, 8081)
![img](img/8080.png)

![img](img/8081.png)

> #### ✅ 포트 매핑 실패 시 빠른 점검 체크리스트
>
> 포트 매핑이 안 될 때 아래 순서대로 확인하세요.
>
> | 순서 | 점검 항목 | 확인 명령어 | 정상 상태 |
> |:---:|:---|:---|:---|
> | 1 | 컨테이너가 실행 중인가? | `docker ps` | STATUS가 `Up` |
> | 2 | 포트 매핑이 설정됐는가? | `docker ps` | PORTS에 `0.0.0.0:8080->5000/tcp` 표시 |
> | 3 | 호스트 포트가 이미 점유됐는가? | `ss -tulnp \| grep 8080` | 아무것도 출력 안 되면 정상(빈 포트) |
> | 4 | 컨테이너 내부 앱이 살아있는가? | `docker exec -it 컨테이너명 curl localhost:5000/health` | `{"status": "ok"}` |
> | 5 | 컨테이너 로그에 에러가 있는가? | `docker logs 컨테이너명` | 에러 메시지 없음 |
>
> 💡 포트 충돌 등 상세 해결 방법은 [6. 트러블슈팅 - Issue 0](#-issue-0-포트-충돌-진단-및-해제)을 참고하세요.


### 5-9 바인드 마운트 반영

```bash
# ── 바인드 마운트 반영 ─────────────────────────────────────

# 기존 컨테이너 정리
$ docker rm -f guestbook-app
$ docker rm -f guestbook-app-8080
$ docker rm -f guestbook-app-8081

# guestbook-no-mount는 바인드 마운트 없이 실행한 컨테이너
# 8080포트 사용
$ docker run -d --name guestbook-app-no-mount \
  -p 8080:5000 \
  guestbook:1.0
f60aacde6bce198c19857130d524656911f2890c1796bba0c819f7b39cc63d86

# guestbook-mount는 바인드 마운트 하여 실행한 컨테이너
# 8081포트 사용
$ docker run -d --name guestbook-app-with-mount \
  -p 8081:5000 \
  -v $(pwd):/app \
  guestbook:1.0
cb6b959e9562cb5ed0c67ae71b59ec6acfb54f210550dabc985bbd73b18969f7

# ── 절대경로 vs 상대경로 사용 기준 ───────────────────────────
#
# [절대경로] 루트(/)부터 시작하는 전체 경로
#   → 현재 위치와 무관하게 항상 동일한 대상을 가리킴
#   → 예: /home/redna/guestbook:/app
#
# [상대경로] 현재 위치(.)를 기준으로 하는 경로
#   → 현재 디렉토리가 바뀌면 가리키는 대상도 달라짐
#   → 예: ./data  (터미널 직접 작업 시에만 사용)
#
# ✅ 절대경로를 써야 할 때
#   - docker run -v 볼륨 마운트 (호스트 경로 지정)
#   - cron, 쉘 스크립트 등 자동 실행 환경
#   - 실행 위치가 고정되지 않은 경우
#
# ✅ 상대경로를 써야 할 때
#   - 터미널에서 직접 파일 탐색·복사·이동
#   - Dockerfile 내부 COPY, ADD 명령 (빌드 컨텍스트 기준)
#   - 프로젝트 폴더 안에서만 동작하는 명령
#
# ✅ $(pwd) 사용 → 터미널에서 직접 실행할 때 절대경로 대체
#   - 현재 디렉토리를 자동으로 절대경로로 변환해줌
#   - 예: $(pwd) → /home/redna/guestbook 으로 자동 변환
#   - 단, 실행 위치가 바뀌면 마운트 대상도 바뀌므로 주의
#
# ❌ 상대경로 직접 사용 금지 → Docker가 인식 못해 오류 발생
#   잘못된 예: -v ./guestbook:/app   ← 오류!
#   올바른 예: -v $(pwd):/app
#           또는 -v /home/redna/guestbook:/app
# ──────────────────────────────────────────────────────────────

# 컨테이너 생성 확인
$ docker ps
CONTAINER ID   IMAGE           COMMAND           CREATED         STATUS                            PORTS                                         NAMES
cb6b959e9562   guestbook:1.0   "python app.py"   4 seconds ago   Up 3 seconds (health: starting)   0.0.0.0:8081->5000/tcp, [::]:8081->5000/tcp   guestbook-app-with-mount
f60aacde6bce   guestbook:1.0   "python app.py"   4 minutes ago   Up 4 minutes (healthy)            0.0.0.0:8080->5000/tcp, [::]:8080->5000/tcp   guestbook-app-no-mount
```

``` bash
# ── 바인드 마운트 검증 ──────────────────────────────────

# 로컬에서 index.html 파일 수정
$ pwd 
/home/redna/guestbook
$ cd app/templates/
$ ls 
index.html
$ vi index.html
# vi 편집기에서 i버튼 눌려서 수정 진행함
# 수정 전: <h1>📖 방명록</h1>
# 수정 후: <h1>📖 로컬에서 수정한 방명록</h1>

# 결과 1
# guestbook-no-mount 컨테이너에서는 바인드 마운트를 하지 않았기 때문에
# index.html 내용이 수정되지 않음
# 수정 내용을 반영하려면 
# 컨테이너 중지 -> 컨테이너 삭제 -> 이미지 다시 빌드 -> 새 이미지로 컨테이너 다시 생성 해야함
$ curl http://localhost:8080
<!DOCTYPE html>
<html lang="ko">
...
    <h1>📖 방명록</h1>
...

# 결과 2
# guestbook-with-mount에서는 컨테이너에서는 바인드 마운트를 했기 때문에
# 이미지 재빌드 안해도 index.html 내용이 바뀜
$ curl http://localhost:8081  
<!DOCTYPE html>
...
    <h1>📖 로컬에서 수정한 방명록</h1>
...


# 다음 실습을 위해 컨테이너 정리
$ docker rm-f guestbook-app-no-mount
$ docker rm-f guestbook-app-with-mount
```




### 5-10 볼륨 영속성

```bash
# ── 준비: 볼륨 생성 및 컨테이너 실행 ──────────────────────────

# 1. Docker 전용 볼륨 생성
#   - guestbook_data 볼륨에 방명록을 저장할 예정
$ docker volume create guestbook-data
guestbook-data

# 2. 생성된 볼륨을 확인
$ docker volume ls 
DRIVER    VOLUME NAME
local     guestbook-data

# 3. 볼륨을 연결하여 컨테이너 생성 
$ docker run -d --name guestbook-app \
  -p 8080:5000 \
  -v guestbook-data:/app/data \
  guestbook:1.0
02815b7730b0688def6d7e81cac1ef31f1b85a6ac88e8c943ee7268cbdfac4e3



# ── 검증 1: 컨테이너 삭제 전 데이터 입력 ──────────────────────

# 방명록 데이터 입력 (응답 없음이 정상)
$ curl -X POST http://localhost:8080/write -d "name=홍길동1&message=안녕하세요1"
$ curl -X POST http://localhost:8080/write -d "name=홍길동2&message=안녕하세요2"

# 입력된 데이터 확인
$ curl http://localhost:8080
<!DOCTYPE html>
...
...
...
    <!-- 방명록 목록 -->

        <p><b>홍길동1</b>: 안녕하세요1</p>

        <p><b>홍길동2</b>: 안녕하세요2</p>

</body>



# ── 검증 2: 컨테이너 삭제 후 데이터 복구 확인 ─────────────────

# 컨테이너 강제 삭제 (볼륨은 유지됨)
$ docker rm -f guestbook-app
guestbook-app


# 동일한 볼륨으로 새 컨테이너 생성
$ docker run -d --name guestbook-app \
  -p 8080:5000 \
  -v guestbook-data:/app/data \
  guestbook:1.0
559035d246108e96862a78cbe1045e1a7b4e58376f192d4e450894876272b946

# JSON 파일 열어서 데이터 생존 확인
docker exec guestbook-app cat /app/data/guestbook.json
[{"name": "홍길동1", "message": "안녕하세요1"}, {"name": "홍길동2", "message": "안녕하세요2"}]

# 웹 응답으로 데이터 생존 확인
$ curl http://localhost:8080
<!DOCTYPE html>
...
...
...
    <!-- 방명록 목록 -->

        <p><b>홍길동1</b>: 안녕하세요1</p>

        <p><b>홍길동2</b>: 안녕하세요2</p>

</body>



# ── 검증 3: 볼륨 내부 파일 소유자·권한 확인 (안전성 점검) ──────

# 파일 소유자 및 권한 확인
$ docker exec guestbook-app ls -l /app/data/guestbook.json
-rw-r--r-- 1 root root 110 Nov  7 12:33 /app/data/guestbook.json

# [결과 분석]
# 권한(-rw-r--r-- = 644)
#   - 소유자(root) : 읽기·쓰기 가능 (rw-)
#   - 그룹         : 읽기만 가능   (r--)
#   - 기타 사용자  : 읽기만 가능   (r--)
#   → 외부에서 임의로 수정 불가 → 안전한 상태
#
# 소유자(root root)
#   - 컨테이너가 root 권한으로 실행되어 파일 소유권이 root로 설정됨
#   - 볼륨을 통해 데이터가 공유되더라도
#     리눅스 표준 권한 체계를 그대로 따르고 있음을 확인

```

> #### 📊 바인드 마운트 vs 볼륨 비교
>
> | 항목 | 바인드 마운트 (`-v $(pwd):/app`) | Docker 볼륨 (`-v 볼륨명:/app/data`) |
> |:---:|:---|:---|
> | **저장 위치** | 호스트의 특정 경로 (내가 지정) | Docker가 관리하는 내부 경로 |
> | **주요 용도** | 개발 중 코드 실시간 반영 | 운영 중 데이터 영속성 보장 |
> | **장점** | 호스트에서 파일 직접 수정 가능 | 경로 의존성 없음, 이식성 높음 |
> | **단점** | 호스트 경로가 바뀌면 마운트 깨짐 | 파일 직접 수정 불편 (`docker exec` 필요) |
> | **적합한 상황** | 개발 환경 (코드 편집) | 운영 환경 (DB, 로그, 방명록 데이터) |
>
> 💡 **한 줄 요약**
> - 바인드 마운트: **"개발할 때"** → 코드 수정이 컨테이너에 즉시 반영
> - Docker 볼륨: **"운영할 때"** → 컨테이너가 삭제돼도 데이터는 살아남음



### 5-11 Git 설정 + VS code 연동

```bash
# ── 준비: GitHub repository 생성 ───────────────────────────────

# 1. github.com 접속 & 로그인
# 2. 오른쪽 상단 [+] → [New repository] 클릭
# 3. 아래처럼 설정
#
#   Repository name : test (원하는 이름)
#   Description     : (선택사항)
#   Public ✅  or  Private
#
#   ⚠️ 아래 3개는 체크하지 마세요!
#   □ Add a README file
#   □ Add .gitignore
#   □ Add license
#
# 4. [Create repository] 클릭!
```

```bash
# ── 준비: 실습 폴더 생성 ───────────────────────────────────────

$ mkdir test && cd test
$ pwd
/home/redna/test
```

```bash
# ── 1단계: Git 사용자 정보 등록 ────────────────────────────────

# 이름 등록
$ git config --global user.name "너의이름"

# 이메일 등록 (GitHub 가입 이메일)
$ git config --global user.email "너의이메일@gmail.com"

# 기본 브랜치 main으로 설정
$ git config --global init.defaultBranch main

# 설정 확인
$ git config --list
init.defaultbranch=main
user.name=너의이름
user.email=너의이메일@gmail.com
```

```bash
# ── 2단계: 로컬 Git 초기화 및 첫 커밋 ─────────────────────────

# git 초기화 (이 폴더를 Git이 관리하도록 설정)
$ git init
Initialized empty Git repository in /home/redna/test/.git/

# README.md 파일 생성
$ echo "VS code와 연동되었습니다." > README.md

# 전체 파일 스테이징
$ git add .

# 첫 커밋
$ git commit -m "first commit"
[main (root-commit) 8d14806] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

```bash
# ── 3단계: GitHub 원격 저장소 연결 및 푸시 ────────────────────

# 브랜치 이름 main으로 설정
$ git branch -M main

# GitHub 원격 저장소 연결
$ git remote add origin https://github.com/깃허브아이디/test.git

# main 브랜치를 원격 저장소에 푸시
$ git push -u origin main

# ⚠️ 인증 요구 시 아래처럼 입력
#   Username : 깃허브 아이디
#   Password : 깃허브 토큰 (비밀번호 아님!)
Username for 'https://github.com': 
Password for 'https://깃허브아이디@github.com': 
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 246 bytes | 246.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.





# ── 토큰 사용 예시 및 비밀정보 관리 주의사항 ───────────────────

# [토큰 발급 방법]
# GitHub → Settings → Developer settings
#          → Personal access tokens → Tokens (classic)
#          → Generate new token → repo 권한 체크 후 생성

# [토큰 사용 예시]
# URL에 토큰을 직접 포함하면 인증 없이 푸시 가능
# (단, 아래 방식은 .git/config에 토큰이 평문으로 저장되므로
#  개인 실습 환경에서만 사용할 것)
$ git remote set-url origin https://ghp_************@github.com/깃허브아이디/test.git




# ──────────────────────────────────────────────────────────────
# 🔴 비밀정보 관리 주의사항 (반드시 준수)
# ──────────────────────────────────────────────────────────────
# 1. 토큰을 코드나 README에 절대 직접 입력하지 말 것
#      ❌ 잘못된 예: token = "ghp_abcd1234..."
#      ✅ 올바른 예: token = os.environ.get("GITHUB_TOKEN")
#
# 2. .gitignore에 민감 파일 등록하여 실수로 커밋되지 않도록 방지
#      ❌ 잘못된 예: .env 파일을 그냥 커밋
#      ✅ 올바른 예: .gitignore에 .env 추가 후 커밋 제외
#
# 3. 토큰이 외부에 노출되었다면 즉시 GitHub에서 폐기(Revoke)
#      GitHub → Settings → Developer settings → 해당 토큰 Delete
#
# 4. 토큰에는 필요한 최소 권한만 부여할 것
#      예: 코드 푸시만 필요하면 repo 권한만 체크
# ──────────────────────────────────────────────────────────────

```



VS code 연동후 스샷
![연동후 스샷](img/연동후.png)



---

## 6. 트러블슈팅 (Troubleshooting)

### 🚨 Issue 0: 포트 충돌 진단 및 해제

```bash
🔴 문제
docker run -p 8080:5000 실행 시 아래 오류 발생
Bind for 0.0.0.0:8080 failed: port is already allocated

🤔 원인 가설
8080 포트를 다른 프로세스가 이미 점유하고 있어서
Docker가 해당 포트를 사용하지 못하는 것 아닐까?
```

```bash
✅ 확인: 포트 점유 프로세스 진단
# ── 진단 도구 1: ss ───────────────────────────────────────────
# ss : 현재 열려있는 포트 상태 조회 (Ubuntu 기본 내장)
# 옵션 설명
#   -t  TCP 소켓만 표시
#   -u  UDP 소켓만 표시
#   -l  LISTEN 상태인 소켓만 표시 (대기 중인 포트)
#   -n  서비스 이름 대신 숫자(포트 번호)로 표시
#   -p  소켓을 사용 중인 프로세스 정보 표시

$ ss -tulnp | grep 8080
tcp   LISTEN  0  128  0.0.0.0:8080  0.0.0.0:*  users:(("docker-proxy",pid=1234,fd=4))
#                                                          ↑ 프로세스명       ↑ PID

# [결과 해석]
# → PID 1234의 docker-proxy 프로세스가 8080 포트를 점유 중임을 확인




# ── 진단 도구 2: lsof (ss 결과가 불명확할 때 교차 확인) ──────────
# lsof : 포트를 점유한 프로세스의 상세 정보 확인

$ sudo lsof -i :8080
COMMAND     PID  USER   FD   TYPE  DEVICE  SIZE/OFF  NODE  NAME
docker-pr  1234  root   4u   IPv4  12345      0t0    TCP   *:8080 (LISTEN)
#            ↑ PID 확인

# [결과 해석]
# → PID 1234 (docker-proxy) 가 8080 포트를 점유하고 있음을 두 도구로 교차 확인 완료


# ⚠️ [WSL 환경 주의]
# WSL에서는 프로세스 격리 구조 차이로 인해
# ss -tulnp 에서 PID/프로세스명이 표시되지 않는 것이 정상입니다.
# 아래 docker ps 명령으로 점유 컨테이너를 확인하세요.

# ── WSL 대체 진단: docker ps ─────────────────────────────────
$ docker ps --format "table {{.Names}}\t{{.Ports}}" | grep 8080
guestbook-app   0.0.0.0:8080->5000/tcp
# ↑ 8080을 점유 중인 컨테이너 이름 확인 완료
```


```bash
🛠️ 해결
방법 A: 점유 프로세스 종료 후 재실행
방법 B: 다른 포트로 변경 (우회)

# ══════════════════════════════════════════════════════════════
# 방법 A: 점유 프로세스 종료 후 재실행
# → 8080 포트를 반드시 써야 할 때 선택
# ══════════════════════════════════════════════════════════════

# 1. 점유 프로세스 강제 종료
$ kill -9 1234

# 2. 포트 해제 확인 (아무것도 출력 안 되면 해제 완료 ✅)
$ ss -tulnp | grep 8080
# (출력 없음 → 8080 포트 해제 확인 ✅)

# 3. 컨테이너 재실행
$ docker run -d -p 8080:5000 --name guestbook-app guestbook:1.0
a3f9c1d2e4b5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1

# 4. 실행 결과 검증
$ docker ps
CONTAINER ID   IMAGE           COMMAND           CREATED         STATUS          PORTS                                         NAMES
a3f9c1d2e4b5   guestbook:1.0   "python app.py"   5 seconds ago   Up 4 seconds    0.0.0.0:8080->5000/tcp, [::]:8080->5000/tcp   guestbook-app
# ↑ PORTS 컬럼에 0.0.0.0:8080->5000/tcp 표시 → 포트 매핑 정상 ✅

$ curl http://localhost:8080/health
{
  "status": "ok"
}
# ✅ 방법 A 해결 완료 — 8080 포트 정상 응답 확인


# ══════════════════════════════════════════════════════════════
# 방법 B: 다른 포트로 변경 (우회)
# → 8080 포트 점유 프로세스를 종료하기 어려울 때 선택
# ══════════════════════════════════════════════════════════════

# 1. 비어있는 포트(8082)로 컨테이너 실행
$ docker run -d -p 8082:5000 --name guestbook-app guestbook:1.0
b4e8f2a1c3d5e7f9a0b2c4d6e8f0a2b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6

# 2. 실행 결과 검증
$ docker ps
CONTAINER ID   IMAGE           COMMAND           CREATED         STATUS          PORTS                                         NAMES
b4e8f2a1c3d5   guestbook:1.0   "python app.py"   5 seconds ago   Up 4 seconds    0.0.0.0:8082->5000/tcp, [::]:8082->5000/tcp   guestbook-app
# ↑ PORTS 컬럼에 0.0.0.0:8082->5000/tcp 표시 → 포트 매핑 정상 ✅

$ curl http://localhost:8082/health
{
  "status": "ok"
}
# ✅ 방법 B 해결 완료 — 8082 포트 정상 응답 확인
```






### 🚨 Issue 1: EC2 배포 방식 개선 - 빌드는 PC에서, EC2는 실행만!

```bash
🔴 문제
EC2에서 git clone → Dockerfile로 이미지 빌드 → 컨테이너 실행 방식은
번거롭고, EC2와 컨테이너 내부에 프로젝트 파일이 중복되는 것 같음

🤔 원인 가설
실무에서는 EC2에서 직접 docker image를 빌드하지 않고
Docker Hub에서 이미지를 pull해서 바로 실행하지 않을까?

✅ 확인
실습을 통해 Docker Hub에서 pull만으로 image를 가져와서 컨테이너 실행이 가능함을 확인

🛠️ 해결 검증 로그
# ✅ EC2에서 git clone 없이 실행 성공
# ✅ EC2에 프로젝트 파일 없음 확인

# ----[개발자 PC]---------------------
# 이미지 빌드 
$ docker build -t 내아이디/guestbook:1.0 .
[+] Building 2.0s (12/12) FINISHED
...
...
...                                                
 => => naming to docker.io/내아이디/guestbook:1.0                                                                 0.0s
 => => unpacking to docker.io/내아이디/guestbook:1.0  

# 이미지 push
$ docker push 내아이디/guestbook:1.0:1.0
The push refers to repository [docker.io/내아이디/guestbook]
37fb86c2a767: Pushed
53daecf7856e: Pushed
44136fa355b3: Pushed
1fc977ef6419: Pushed
7efbce397249: Pushed
bbe7053e6def: Pushed
5fd629c46c96: Pushed
b3606bd450c8: Pushed
979733a6438f: Pushed
37b044d3f935: Pushed
fca544126c32: Pushed
1.0: digest: sha256:b6eda1a172a4d71c2a700ff632fca524b5da095eebe0f06a95686ca9802c3838 size: 856
# -----------------------------------


# ----[EC2]--------------------------
# (프로젝트 파일 없음 = 이미지만으로 실행 가능 확인 ✅)
$ ls 


# 이미지 pull
$ docker pull 내아이디/guestbook:1.0
1.0: Pulling from 내아이디/guestbook
fca544126c32: Pull complete
bbe7053e6def: Pull complete
37fb86c2a767: Pull complete
53daecf7856e: Pull complete
7efbce397249: Pull complete
b3606bd450c8: Pull complete
5fd629c46c96: Pull complete
979733a6438f: Pull complete
37b044d3f935: Pull complete
44136fa355b3: Download complete
1fc977ef6419: Download complete
Digest: sha256:b6eda1a172a4d71c2a700ff632fca524b5da095eebe0f06a95686ca9802c3838
Status: Downloaded newer image for 내아이디/guestbook:1.0
docker.io/내아이디guestbook:1.0

# 컨테이너 생성
$ docker run -d -p 8080:5000 --name guestbook-app 내아이디/guestbook:1.0
8c909827339ae84e190180d3e9481b438c9b25a0233c84dfade860753c3b68b1

# 컨테이너 동작 확인
$ curl http://localhost:80/health
{
  "status": "ok"
}
# -----------------------------------
```

![도커허브 레포 생김](img/도커허브%20레포지토리.png)


> 🔄 **이미지 업데이트 시 컨테이너 교체 절차**
>  
> 앱 코드가 변경되어 이미지를 새로 빌드했다면,
> 기존 컨테이너를 교체해야 변경사항이 반영됩니다.

> [개발자 PC]
> ```bash
> # 1. 새 이미지 빌드 & push
> $ docker build -t 내아이디/myapp:2.0 .
> $ docker push 내아이디/myapp:2.0
> ```

> [EC2 서버]
>  
> ```bash
> # 2. 새 이미지 pull
> $ docker pull 내아이디/myapp:2.0
>  
> # 3. 기존 컨테이너 중지 & 삭제
> $ docker stop myapp
> $ docker rm myapp
>  
> # 4. 새 이미지로 컨테이너 재생성
> $ docker run -d -p 80:8080 --name myapp 내아이디/myapp:2.0


```bash
# ⚠️ 주의: docker restart는 이미지를 교체하지 않습니다.
# 코드 변경을 반영하려면 반드시 컨테이너를 삭제 후 재생성해야 합니다.
$ docker restart myapp         # ❌ 이미지 그대로 (코드 변경 미반영)
$ docker rm → docker run       # ✅ 새 이미지 적용 (코드 변경 반영)
```





---

### 🚨 Issue 2: 소셜 로그인(Google) 계정 docker login 비밀번호 오류 해결


```dockerfile
🔴 문제
docker login 시도 시 비밀번호를 알 수 없음
→ Docker Hub 가입을 Google 소셜 로그인으로 했기 때문

🤔 원인 가설
소셜 로그인 계정은 별도 비밀번호가 없으므로
Access Token이 비밀번호 대신 사용 가능하지 않을까?

✅ 확인
Docker Hub 공식 문서에서
Access Token을 비밀번호 대신 사용할 수 있음을 확인

🛠️ 해결
1. Docker Hub 로그인 (웹 브라우저)
2. Account Settings → Security → Access Tokens
3. [Generate New Token] 클릭
4. 토큰 복사 후 아래 명령어 실행
```

```bash
# 해결 검증 로그
# ✅ 로그인
$ docker login -u 내아이디
USING WEB-BASED LOGIN

i Info → To sign in with credentials on the command line, use 'docker login -u <username>'


Your one-time device confirmation code is: MRDQ-JTMF
Press ENTER to open your browser or submit your device code here: https://login.docker.com/activate

Waiting for authentication in the browser…
^Clogin canceled
redna@js:~$ docker login -u genie1486

i Info → A Personal Access Token (PAT) can be used instead.
         To create a PAT, visit https://app.docker.com/settings


Password:
Login Succeeded

# ✅ push 정상 동작 확인
$ docker push 내아이디/guestbook:1.0
The push refers to repository [docker.io/내아이디/guestbook]
53daecf7856e: Pushed
8e191f69611d: Pushed
37fb86c2a767: Pushed
44136fa355b3: Pushed
7efbce397249: Pushed
bbe7053e6def: Pushed
5fd629c46c96: Pushed
b3606bd450c8: Pushed
979733a6438f: Pushed
37b044d3f935: Pushed
fca544126c32: Pushed
1.0: digest: sha256:e4460f0613bdb52b60b697067a3416a8b3aacb247601e943766649ca0a54e5de size: 856
```



---

## 7. 보너스 과제 (추후 학습 예정)

| 상태 | 과제 | 배움 포인트 | 링크 |
|------|------|------------|------|
| [ ] | Docker Compose 기초 | 컨테이너 실행 명령이 "문서화된 실행 설정"으로 바뀌는 이유 | [#]() |
| [ ] | Docker Compose 멀티 컨테이너 | 네트워크/서비스 디스커버리 개념 맛보기 | [#]() |
| [ ] | Compose 운영 명령어 습득 | 운영 관점의 "상태 확인 루틴" 만들기 | [#]() |
| [ ] | 환경 변수 활용 | 설정과 코드의 분리 | [#]() |
| [ ] | GitHub SSH 키 설정 | 인증 방식 차이와 보안 습관 | [#]() |