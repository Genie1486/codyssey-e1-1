# 🐳 Docker Compose 멀티 컨테이너 + 네트워크 통신 실습

---

## 학습 목표

```
✅ 웹 서버 + 보조 서비스(Redis) 2개를 Compose로 함께 실행한다
✅ 컨테이너 간 네트워크 통신이 가능한지 확인한다
✅ 배움 포인트: 네트워크 / 서비스 디스커버리 개념 맛보기
```

---

## 1. 전체 구성 한눈에 보기

```
[ 브라우저 ]
     │  HTTP 요청
     ▼
┌─────────────────────────────────────────┐
│           docker-compose 네트워크        │
│                                         │
│  ┌──────────────┐    ┌───────────────┐  │
│  │   app (Flask)│───▶│  redis        │  │
│  │   port: 5000 │    │  port: 6379   │  │
│  └──────────────┘    └───────────────┘  │
│                                         │
│  서비스명 = DNS 이름 (자동 등록!)         │
└─────────────────────────────────────────┘
```

- `app` 컨테이너와 `redis` 컨테이너는 **같은 네트워크** 안에 있음
- 별도 IP 설정 없이 **서비스명(redis)** 으로 바로 통신 가능

---

## 2. docker-compose.yml 설정

```yaml
version: '3.8'

services:
  app:
    build: .
    image: guestbook:1.0
    container_name: guestbook-app
    ports:
      - "${HOST_PORT}:${PORT}"  
    volumes:
      - guestbook-data:/app/data
    environment:
      MODE: ${MODE}             
      DEBUG: ${DEBUG}
      PORT: ${PORT}
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${PORT}/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    depends_on:
      - redis          # ← redis 먼저 실행되어야 함

  redis:
    image: redis:7-alpine     # ← 빌드 없이 이미지 그대로 사용
    container_name: guestbook-redis   # ← 이거 추가!
    ports:
      - "6379:6379"

volumes:
  guestbook-data:
```

| 항목 | 설명 |
|------|------|
| `services` | 띄울 컨테이너 목록 |
| `build: .` | 현재 디렉토리 Dockerfile로 빌드 |
| `ports` | 호스트 ↔ 컨테이너 포트 연결 |
| `depends_on` | 실행 순서 보장 (redis 먼저) |
| `image` | 빌드 없이 공식 이미지 사용 |

---

## 3. Redis 연결 코드 — 어떻게 동작하나?

### 3-1. Redis 연결 객체 만들기

```python
# app.py 
r = redis.Redis(
    host='redis',            # ← 서비스명으로 연결 (IP 아님!)
    port=6379,
    decode_responses=True    # ← bytes 대신 문자열로 자동 변환
)
```

> **host='redis'** 가 핵심!  
> Docker 내부 DNS가 `redis` 라는 이름을 자동으로 해당 컨테이너 IP로 변환해줌  
> → IP를 몰라도, 하드코딩 안 해도 됨 ✅

---

### 3-2. 연결 확인 — ping

```python
r.ping()
redis_status = "ok"
```

| 동작 | 설명 |
|------|------|
| `r.ping()` | Redis 서버에 "살아있니?" 신호 전송 |
| 응답 `True` | Redis 정상 작동 중 |
| 응답 없음 / 에러 | 연결 실패 → host 이름 또는 네트워크 확인 필요 |

---

### 3-3. 방문자 수 카운트 — incr

```python
visit_count = r.incr("visit_count")
```

| 단계 | 상황 | 동작 |
|------|------|------|
| 첫 요청 | `visit_count` 키 없음 | Redis가 **0으로 초기화 후 +1** → 결과: `1` |
| 두 번째 요청 | `visit_count = 1` | **+1** → 결과: `2` |
| N번째 요청 | `visit_count = N-1` | **+1** → 결과: `N` |

> `incr` = **increment** (증가)
> 키가 없어도 알아서 0부터 시작해줌 → 초기화 코드 따로 안 써도 됨 ✅

---

### 3-4. 전체 흐름 정리

```
브라우저 요청
     │
     ▼
Flask (web 컨테이너)
     │
     ├─ r.ping()                  → Redis야, 살아있니?
     │       └─ True              ← 응답
     │
     ├─ r.incr("visit_count")     → 방문자 수 +1 해줘
     │       └─ 3                 ← 현재 값 반환
     │
     └─ HTML 응답 → 브라우저에 방문자 수 표시
```

---

## 4. 실행 & 확인

```bash
# 빌드 후 백그라운드 실행
docker compose up --build -d

# 컨테이너 상태 확인
docker compose ps

# 로그 확인
docker compose logs -f web

# 브라우저 또는 curl로 확인
curl http://localhost:5000
```

### [새로고침 전 방문자수]
![새로 고침 전 방문자수](img/새로%20고침%20전%20방문자%20수.png)  

### [새로고침 후 방문자수]   
![img](img/새로%20고침%20후%20방문자%20수.png)

---

## 5. 컨테이너 간 통신 — 어떻게 가능한가?

### 5-1. 같은 네트워크 = 직접 통신 가능

```
docker compose up 실행
        │
        ▼
Docker가 가상 네트워크를 자동 생성
        │
        ▼
┌──────────────────────────────────────┐
│         myapp_default (가상 LAN)      │
│                                      │
│   [app 컨테이너]      [redis 컨테이너] │
│   172.18.0.3    ◀──▶  172.18.0.2    │
│                                      │
│   같은 네트워크 → 직접 통신 가능 ✅    │
└──────────────────────────────────────┘
```

 같은 `docker-compose.yml` 안에 있으면 → **자동으로 같은 네트워크!**

---

### 5-2. 그런데 IP는 어떻게 아나? — Docker 내장 DNS

IP를 직접 쓰면 컨테이너가 재시작될 때마다 바뀔 수 있어요.  
그래서 Docker는 **서비스명을 DNS 이름으로 자동 등록**해줍니다.

```
app 컨테이너 안에서 host='redis' 로 연결 시도
        │
        ▼  "redis 가 어디야?"
┌───────────────────────────┐
│   /etc/resolv.conf        │  ← Docker가 자동 설정
│   nameserver 127.0.0.11   │  ← Docker 내장 DNS 서버 주소
└───────────────────────────┘
        │
        ▼  127.0.0.11 에게 질의
┌───────────────────────────┐
│   Docker 내장 DNS         │
│   (127.0.0.11)            │
└───────────────────────────┘
        │
        ▼  "redis = 172.18.0.2 야"
        │
        ▼
   172.18.0.2 로 연결 성공 ✅
```

---

### 5-3. 실제로 확인해보면

```bash
# web 컨테이너 접속
docker-compose exec app bash

# Docker가 설정한 DNS 서버 확인
cat /etc/resolv.conf
# nameserver 127.0.0.11   ← Docker 내장 DNS

# 서비스명 → IP 변환 확인
getent hosts redis
# 172.18.0.2    redis      ← DNS가 변환해준 결과

getent hosts app
# 172.18.0.3    app        ← 내 app 컨테이너도 등록되어 있음
```

> `getent hosts redis` 결과가 나온다는 것 =  
> **Docker DNS가 `redis` 라는 이름을 실제 IP로 알고 있다**는 증거! 🎉

---

### 5-4. 통신 가능 범위 정리

| 상황 | 통신 방법 |
|------|----------|
| **같은 Compose** | 서비스명으로 바로 연결 (`host='redis'`) ✅ |
| **다른 Compose** | 포트를 열고 IP로 연결해야 함 ⚠️ |
| **외부 인터넷 → 컨테이너** | 포트 매핑 필수 (`ports: 5000:5000`) 🔒 |

---

## 🎓 배움 포인트: 서비스 디스커버리 개념 맛보기

> **"어디 있는지 몰라도, 이름만 알면 찾아갈 수 있다"**

```
host='redis'  한 줄로 연결되는 것
      │
      └─▶ 이게 바로 서비스 디스커버리의 가장 기본 형태!
```

| 환경 | 서비스 디스커버리 방식 |
|------|----------------------|
| Docker Compose | 서비스명 → 내장 DNS (127.0.0.11) |
| Kubernetes | Service 이름 → CoreDNS |
| 클라우드 MSA | Consul, Eureka 등 |

---



