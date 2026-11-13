# 🌿 환경 변수 활용 실습

> **배움 포인트: 설정과 코드의 분리**  
> Compose에서 환경 변수를 주입해 서버 포트/모드를 바꿔본다.

---

## 1. 왜 "설정과 코드의 분리"인가?

### 🤔 문제 상황 — 하드코딩의 함정

```python
# ❌ 하드코딩된 코드
app.run(host='0.0.0.0', port=5000, debug=True)
```

| 상황 | 하드코딩이면? |
|------|-------------|
| 개발 → 운영 배포 | 코드를 직접 수정해야 함 |
| 포트를 바꾸고 싶을 때 | 코드를 수정 → 이미지 재빌드 |
| 팀원마다 환경이 다를 때 | 각자 코드를 다르게 수정 → 충돌 |
| 비밀번호·API 키 | 코드에 노출 → 보안 사고 |

### ✅ 해결 — 환경 변수로 분리

```python
# ✅ 환경 변수로 읽기
PORT = int(os.getenv('PORT', 5000))   # 없으면 기본값 5000
DEBUG = os.getenv('DEBUG', 'False').lower() == 'true'
MODE = os.getenv('MODE', 'production')

app.run(host='0.0.0.0', port=PORT, debug=DEBUG)
```

```
코드(app.py)     →  "어떻게 동작하는가"  →  변경 없음
설정(Compose)    →  "어떤 값으로 동작하는가"  →  환경마다 다르게
```

> 💡 **핵심 원칙**: 코드는 그대로, 값만 바꾼다.  
> 이미지를 새로 빌드하지 않아도 동작이 달라진다!

---

## 2. 실습 구조

```
📁 프로젝트 폴더
├── app.py
├── Dockerfile
├── docker-compose.yml          ← 기본 (개발용)
├── docker-compose.prod.yml     ← 운영용 오버라이드
└── .env                        ← 환경 변수 파일 (선택)
```

---

## 3. 실습 1 — Compose 환경 변수로 포트/모드 바꾸기

### 현재 docker-compose.yml 확인

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: guestbook:1.0
    container_name: guestbook-app
    ports:
      - "8080:5000"          # 호스트 8080 → 컨테이너 5000
    volumes:
      - guestbook-data:/app/data
    environment:
      MODE: development      # ← 여기서 주입
      DEBUG: "True"
      PORT: 5000
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  guestbook-data:
```

### 실행 및 로그 확인

```bash
# 컨테이너 실행
$ docker-compose up -d

# 🔍 핵심: 시작 로그 확인
$ docker-compose logs app
```

**예상 출력:**
```
guestbook-app  | 🚀 Starting app in development mode on port 5000
guestbook-app  |  * Running on http://0.0.0.0:5000
guestbook-app  |  * Debug mode: on        ← DEBUG=True 반영됨
```

> ✅ `MODE: development`, `DEBUG: True`, `PORT: 5000` 이 주입된 것을 로그로 확인!

---

## 4. 실습 2 — 운영 모드로 바꾸기 (이미지 재빌드 없이!)

### 운영용 오버라이드 파일 작성

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    ports:
      - "80:8080"            # 운영은 80포트
    environment:
      MODE: production       # ← 모드 변경
      DEBUG: "False"         # ← 디버그 끄기
      PORT: 8080             # ← 포트 변경
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
```

> ⚠️ `app.py`의 `PORT` 환경 변수를 컨테이너 내부 포트로 사용하므로,  
> `ports`의 컨테이너 쪽 번호와 `PORT` 값을 맞춰야 합니다.

### 운영 모드로 실행

```bash
# 기존 컨테이너 중지
$ docker-compose down

# 운영 설정 오버라이드로 실행
# -f 는 파일 지정 옵션
# -f 없으면? → 자동으로 docker-compose.yml 찾음
# 오버라이드 해서 docker-compose.prod.yml서 새로 정의된 것만 덮어씀
$ docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 로그 확인
$ docker-compose -f docker-compose.yml -f docker-compose.prod.yml logs app
```

**예상 출력:**
```
guestbook-app  | 🚀 Starting app in production mode on port 8080
guestbook-app  |  * Running on http://0.0.0.0:8080
guestbook-app  |  * Debug mode: off       ← DEBUG=False 반영됨
```

> ✅ **같은 이미지**, **다른 설정** → 동작이 달라졌다!


```bash
# 오버라이드 해서 컨테이너 띄운 경우에 컨테이너 중지하려면
# 띄울 때랑 똑같이 파일 지정해서 내려야 해요
$ docker-compose -f docker-compose.yml -f docker-compose.prod.yml down
```

---

## 5. 실습 3 — .env 파일로 변수 관리

환경 변수가 많아지면 `docker-compose.yml` 안에 직접 쓰는 것보다  
`.env` 파일로 분리하는 것이 깔끔합니다.

### .env 파일 작성

```bash
# .env
MODE=development
DEBUG=True
PORT=5000
HOST_PORT=8080
```

### docker-compose.yml 수정

```yaml
# docker-compose.yml (변수 참조 방식)
version: '3.8'

services:
  app:
    image: guestbook:1.0
    container_name: guestbook-app
    ports:
      - "${HOST_PORT}:${PORT}"   # ← .env 값 참조
    volumes:
      - guestbook-data:/app/data
    environment:
      MODE: ${MODE}              # ← .env 값 참조
      DEBUG: ${DEBUG}
      PORT: ${PORT}
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${PORT}/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  guestbook-data:
```

### 실행 및 변수 주입 확인

```bash
# .env 변수가 잘 반영됐는지 먼저 확인 (실행 전 점검)
$ docker-compose config
# → 출력된 yml에서 ${변수} 자리에 실제 값이 들어갔는지 확인

# 실행
$ docker-compose up -d

# 로그 확인
$ docker-compose logs app
```

**예상 출력:**
```
guestbook-app  | 🚀 Starting app in development mode on port 5000
```

### ⚠️ .env 파일 보안 주의

```bash
# .gitignore에 반드시 추가!
$ echo ".env" >> .gitignore

# 대신 예시 파일은 공유
$ cp .env .env.example
# .env.example 안의 실제 값은 지우고 키 이름만 남기기
```

```bash
# .env.example
MODE=
DEBUG=
PORT=
HOST_PORT=
```

---

## 6. 실습 결과 비교표

| 항목 | 개발 모드 | 운영 모드 |
|------|----------|----------|
| `MODE` | development | production |
| `DEBUG` | True | False |
| `PORT` (컨테이너) | 5000 | 8080 |
| 호스트 포트 | 8080 | 80 |
| 이미지 재빌드 | ❌ 불필요 | ❌ 불필요 |
| 로그 확인 | `🚀 Starting app in development mode on port 5000` | `🚀 Starting app in production mode on port 8080` |

---

## 7. 핵심 정리

```
┌─────────────────────────────────────────────┐
│           설정과 코드의 분리                  │
├─────────────────────────────────────────────┤
│                                             │
│  app.py          → "어떻게" 동작하는가       │
│  (코드)             변경 없음, 재빌드 없음    │
│                                             │
│  docker-compose  → "어떤 값으로" 동작하는가  │
│  .env               환경마다 다르게 설정      │
│                                             │
└─────────────────────────────────────────────┘
```

| 개념 | 설명 |
|------|------|
| `os.getenv('KEY', 기본값)` | 환경 변수 읽기, 없으면 기본값 사용 |
| `environment:` | Compose에서 컨테이너에 변수 주입 |
| `.env` 파일 | 변수를 파일로 분리해 관리 |
| `docker compose config` | 변수 치환 결과 미리 확인 |
| 오버라이드 파일 | `-f` 옵션으로 환경별 설정 덮어쓰기 |