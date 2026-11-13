# 🐳 Docker Compose 기초 실습

---

## 1️⃣ 이 명령어... 외울 수 있겠어?

지금 Guestbook 앱을 실행하려면 이렇게 해야 해:

```bash
$ docker run -d \
  --name guestbook-app \
  -p 8080:5000 \
  -v guestbook-data:/app/data \
  -e MODE=development \
  -e DEBUG=True \
  -e PORT=5000 \
  --restart always \
  --health-cmd="curl -f http://localhost:5000/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  guestbook:1.0
```

**문제점 3가지:**
- 😤 길고 복잡해서 실수하기 쉬움
- 😤 팀원한테 "이 명령어 그대로 쳐봐" 하기 어려움
- 😤 Git에 저장이 안 됨 → 6개월 뒤 내가 봐도 모름

> 💡 이걸 해결하는 게 **Docker Compose**

---

## 2️⃣ 저 명령어를 docker-compose.yml로 변환

변환 대응표:

| docker run 옵션 | docker-compose.yml |
|---|---|
| `--name guestbook-app` | `container_name: guestbook-app` |
| `-p 8080:5000` | `ports: - "8080:5000"` |
| `-v guestbook-data:/app/data` | `volumes: - guestbook-data:/app/data` |
| `-e MODE=development` | `environment: MODE: development` |
| `--restart always` | `restart: always` |
| `--health-cmd=...` | `healthcheck: test: [...]` |

변환 결과:

```yaml
version: '3.8'                          # ① Compose 파일 버전

services:                               # ② 실행할 컨테이너 목록
  app:                                  #    서비스 이름 (마음대로 지정)
    image: guestbook:1.0                #    어떤 이미지 쓸지
    container_name: guestbook-app       #    컨테이너 이름
    ports:                              #    포트 연결 (호스트:컨테이너)
      - "8080:5000"
    volumes:                            #    볼륨 마운트
      - guestbook-data:/app/data
    environment:                        #    환경변수
      MODE: development
      DEBUG: "True"
      PORT: 5000
    restart: always                     #    재시작 정책
    healthcheck:                        #    헬스체크
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:                                # ③ 볼륨 목록 (볼륨마운트 쓸 때 여기 선언)
  guestbook-data:
```

> **핵심 구조는 딱 3개:**  
> `version` → `services` → `volumes`

---

## 3️⃣ **docker-compose 패키지 설치하기**


```bash
# 🔍 먼저 확인해보기
docker-compose --version
```

**만약:**
```
Command 'docker-compose' not found
```

→ **설치 안 됐다!** ❌





```bash
# ✅ 설치 방법
# 1단계: 최신 버전 다운로드
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 2단계: 실행 권한 주기
sudo chmod +x /usr/local/bin/docker-compose

# 3단계: 확인
docker-compose --version

Docker Compose version v5.7.1
```

**출력:**
```
Docker Compose version v2.xx.x
```






> [참고] **Windows (Docker Desktop)**  
> Docker Desktop 설치하면 자동으로 포함됨! ✅

---


## 4️⃣ 직접 서버 띄워보기


### Step 1: 이미지 먼저 빌드
```bash
$ docker build -t guestbook:1.0 .
```

### Step 2: docker-compose.yml 생성
```bash
$ vi docker-compose.yml
# 위 내용 붙여넣기 후 저장
```

### Step 3: 실행
```bash
$ docker-compose up -d
```

### 출력 예시
```
Creating network "ssh_test_default" with the default driver
Creating volume "ssh_test_guestbook-data" with default driver
Creating guestbook-app ... done
```
> 터미널 바로 돌아옴 ✅

### Step 4: 잘 떴는지 확인
```bash
$ docker-compose ps
```
```
     Name           Command        State           Ports
----------------------------------------------------------
guestbook-app   python app.py     Up      0.0.0.0:8080->5000/tcp
```

### Step 5: 실제로 요청 날려보기
```bash
$ curl http://localhost:8080/
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>방명록</title>
</head>
<body>
    <!-- 바인드마운트 실습: 이 제목을 로컬에서 바꿔보기 -->
    <h1>📖 방명록</h1>
+100
    <!-- 글쓰기 폼 -->
    <form action="/write" method="POST">
        <input type="text" name="name" placeholder="이름" />
        <input type="text" name="message" placeholder="메시지" />
        <button type="submit">등록</button>
    </form>

    <hr>

    <!-- 방명록 목록 -->

        <p>아직 방명록이 없어요. 첫 글을 남겨보세요!</p>

</body>
```

### Step 6: 로그 확인
```bash
$ docker-compose logs -f

WARN[0000] /home/username/guestbook/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
guestbook-app  | 🚀 Starting app in development mode on port 5000
guestbook-app  |  * Serving Flask app 'app'
guestbook-app  |  * Debug mode: on
guestbook-app  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
guestbook-app  |  * Running on all addresses (0.0.0.0)
guestbook-app  |  * Running on http://127.0.0.1:5000
guestbook-app  |  * Running on http://172.18.0.2:5000
guestbook-app  | Press CTRL+C to quit
guestbook-app  |  * Restarting with stat
guestbook-app  |  * Debugger is active!
guestbook-app  |  * Debugger PIN: 418-504-562
guestbook-app  | 127.0.0.1 - - [13/Nov/2026 19:07:11] "GET /health HTTP/1.1" 200 -
guestbook-app  | 172.18.0.1 - - [13/Nov/2026 19:07:31] "GET / HTTP/1.1" 200 -
guestbook-app  | 127.0.0.1 - - [13/Nov/2026 19:07:41] "GET /health HTTP/1.1" 200 -
guestbook-app  | 127.0.0.1 - - [13/Nov/2026 19:08:11] "GET /health HTTP/1.1" 200 -
```

### Step 7: 종료
```bash
$ docker-compose down
```

---

## 5️⃣ 배움 포인트 정리

> **"컨테이너 실행 명령이 문서화된 실행 설정으로 바뀌는 이유"**

| | docker run | docker-compose |
|---|---|---|
| **형태** | 터미널 명령어 | YAML 파일 |
| **공유** | 말로 전달, 복붙 | Git 커밋으로 공유 |
| **재현성** | 옵션 빠뜨릴 수 있음 | 항상 동일하게 실행 |
| **가독성** | 한 줄에 다 때려넣음 | 항목별로 명확히 분리 |
| **수정** | 명령어 다시 외워야 함 | 파일 열어서 수정 |

**한 줄 요약:**
> `docker run` = 일회성 명령  
> `docker-compose.yml` = 팀이 함께 쓰는 **실행 설명서**

---

