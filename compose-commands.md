# 📋 Docker Compose 운영 명령어 실습

## 🧠 개념 정리

> 컨테이너를 **올리고 · 내리고 · 확인하고 · 로그 보는** 것  
> 이 4가지가 운영의 기본이다

| 명령어 | 역할 |
|--------|------|
| `up -d` | 컨테이너 백그라운드 실행 |
| `down` | 컨테이너 + 네트워크 제거 |
| `ps` | 현재 실행 중인 컨테이너 상태 확인 |
| `logs` | 컨테이너 로그 확인 |

---

## 🛠️ 실습

### 1. 실행
```bash
$ docker-compose up -d

# 결과
[+] up 2/2
 ✔ Network guestbook_default Created                                                                                                                                                      
 ✔ Container guestbook-app   Started  
```

### 2. 상태 확인
```bash
$ docker-compose ps

# 결과
NAME            IMAGE           COMMAND           SERVICE   CREATED          STATUS                    PORTS
guestbook-app   guestbook:1.0   "python app.py"   app       27 seconds ago   Up 27 seconds (healthy)   0.0.0.0:8080->5000/tcp, [::]:8080->5000/tcp
```

### 3. 로그 확인
```bash
# 전체 로그 한번에 보기
$ docker-compose logs

# 실시간으로 따라가며 보기
$ docker-compose logs -f

# 특정 서비스만 보기
$ docker-compose logs app
```

### 4. 종료
```bash
$ docker-compose down

# 결과
[+] down 2/2
 ✔ Container guestbook-app   Removed 
 ✔ Network guestbook_default Removed  
```

### 5. 변경사항 반영할 때
```bash
# ❌ 이렇게 하면 안 먹힘
$ docker-compose up -d

# ✅ 이렇게 해야 반영됨
$ docker-compose down
$ docker-compose up -d

# ✅ 또는 한방에
$ docker-compose up -d --force-recreate
```

---

## 🔄 운영 관점의 상태 확인 루틴

```
서비스 배포 후
      ↓
① ps  → 컨테이너 정상적으로 떠있나?
      ↓
② logs → 에러 메시지 없나?
      ↓
③ 문제 있으면 → down → 수정 → up -d
      ↓
④ 문제 없으면 → logs -f 로 실시간 모니터링
```

### 루틴 명령어 순서
```bash
# Step 1. 상태 확인
$ docker-compose ps

# Step 2. 로그 확인
$ docker-compose logs

# Step 3. 이상 없으면 실시간 모니터링
$ docker-compose logs -f

# Step 4. 문제 있으면 재시작
$ docker-compose down && docker-compose up -d
```

---

## 💡 핵심 포인트

```
ps   → "떠있나?"
logs → "문제없나?"
down → "내리자"
up   → "올리자"

이 4개가 운영의 전부다 🎯
```

> ✅ `.env` 바꿨을 때 그냥 `up -d` 하면 반영 안 됨!  
> ✅ `down → up -d` 또는 `--force-recreate` 잊지 말기!

---