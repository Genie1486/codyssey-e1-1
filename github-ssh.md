# 🚀 GitHub SSH 키 설정 실습

HTTPS 방식 대신 SSH로 Github 원격 저장소에 Push하는 과정을 함께 진행해봅시다.

---

## 📋 사전 확인

먼저 당신의 상황을 파악하겠습니다.

```bash
# 1. SSH 키가 이미 생성되었나?
$ ls -la ~/.ssh/

# 출력 예시(SSH 키가 생성 되지 않았을 때):
# ls: cannot access '/home/username/.ssh/': No such file or directory

# 출력 예시(SSH 키가 생성 된 상태일 때):
# total 24
# drwx------  2 username username 4096 Jan 15 10:30 .
# drwxr-xr-x 20 username username 4096 Jan 15 10:00 ..
# -rw-------  1 username username  419 Jan 15 10:30 id_ed25519       ← 개인키
# -rw-r--r--  1 username username  102 Jan 15 10:30 id_ed25519.pub   ← 공개키

# 만약 id_ed25519가 없으면 위의 "Step 1: SSH 키 생성" 진행
```

---

## ✅ Step 1: SSH 키 생성 (아직 안 했으면)

```bash
# Ubuntu 터미널에서
$ ssh-keygen -t ed25519 -C "your_email@example.com"

# 엔터 3번 (기본값 사용, 패스프레이즈 없음)
# 또는 패스프레이즈 입력 후 엔터 2번
# 패스프레이즈는 개인키를 보호하는 비밀번호 입니다.
# 누군가 개인키 파일을 훔쳐도 패스프레이즈 없으면 사용 불가능

# -t는 암호알고리즘을 정하는 옵션입니다. 실습에서는 ed25519 알고리즘을 사용했습니다.

# -C는 커멘트 입니다. 공개키 파일(id_ed25519.pub)의 끝에 붙는 설명입니다. 
#   - 여러 SSH 키가 있을 때 어느 키인지 구분
#   - GitHub에 등록할 때 어느 컴퓨터의 키인지 표시
#   - 보안 감시: "이 이메일의 키가 언제 생성됐나?" 추적 용이
```

---

## ✅ Step 2: 공개키를 GitHub에 등록

```bash
# 공개키 확인
$ cat ~/.ssh/id_ed25519.pub

# 출력:
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKp8... your_email@example.com
```

**이 전체 내용을 복사해서:**
1. https://github.com/settings/keys 접속
2. [New SSH key] 클릭
3. Title: `Ubuntu SSH Key`
4. Key type: Authentication Key
4. Key: 위 내용 붙여넣기
5. [Add SSH key] 클릭

---

## ✅ Step 3: SSH 연결 테스트

```bash
$ ssh -T git@github.com

# 처음 연결 시:
# The authenticity of host 'github.com (140.82.113.3)' can't be established.
# ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqE.
# Are you sure you want to continue connecting (yes/no/fingerprint)?

yes  # 입력

# 성공:
# Hi your_username! You've successfully authenticated, but GitHub does not provide shell access.
```

✅ SSH 연결 성공!

---

## 🎯 Step 4: test_ssh 저장소 준비

### 4-1) 저장소 클론 (아직 안 했으면)

```bash
# SSH 주소로 클론
$ git clone git@github.com:your_username/test_ssh.git
$ cd test_ssh

# 또는 이미 HTTPS로 클론했으면 다음 진행
```

### 4-2) 기존 저장소를 HTTPS → SSH로 변경

```bash
# 현재 리모트 확인
$ git remote -v

# 출력 예시:
# origin  https://github.com/your_username/test_ssh.git (fetch)
# origin  https://github.com/your_username/test_ssh.git (push)

# SSH로 변경
$ git remote set-url origin git@github.com:your_username/test_ssh.git

# 변경 확인
$ git remote -v

# 출력:
# origin  git@github.com:your_username/test_ssh.git (fetch)
# origin  git@github.com:your_username/test_ssh.git (push)
```

✅ 저장소 설정 완료!

---

## 🚀 Step 5: 실제 Push 실습

### 5-1) 테스트 파일 생성

```bash
# E1-1 디렉토리에서
$ cd ~/test_ssh
# SSH 테스트 파일 생성
$ cat > SSH_TEST.md << 'EOF'
# SSH Push 테스트

## 목표
- SSH 인증으로 Push 성공
- 재전송 공격 방어 메커니즘 이해

## 진행 상황
- ✅ SSH 키 생성
- ✅ GitHub에 공개키 등록
- ✅ SSH 연결 테스트
- ✅ 저장소 SSH로 변경
- 🔄 Push 실행 중...

## 타임스탐프
생성: 2026-11-14 00:30:45
EOF

# 파일 확인
$ cat SSH_TEST.md
```

### 5-2) Git 스테이징 & 커밋

```bash
# 파일 추가
$ git add SSH_TEST.md

# 상태 확인
$ git status

# 출력:
# On branch main
# Changes to be committed:
#   (use "git restore --cached <file>..." to unstage)
#         new file:   SSH_TEST.md

# 커밋
$ git commit -m "SSH 테스트: 타임스탐프와 논스 검증 실습"

# 출력:
# [main abc1234] SSH 테스트: 타임스탐프와 논스 검증 실습
#  1 file changed, 12 insertions(+)
#  create mode 100644 SSH_TEST.md
```

### 5-3) 🎯 SSH Push 실행!

```bash
# Push 명령어
$ git push origin main

# 🔍 실시간 출력 분석:
```

---

## 📊 Push 과정 상세 분석

```bash
$ git push origin main

# ========== 1단계: SSH 연결 ==========
# Enumerating objects: 3, done.
# → 로컬: "GitHub야, 3개 객체를 보낼게"
# → SSH 세션 확립 (타임스탐프 + 논스 포함)

# ========== 2단계: 데이터 압축 ==========
# Counting objects: 100% (3/3), done.
# Delta compression using up to 8 threads
# Compressing objects: 100% (2/2), done.
# → 로컬: "변경사항 압축 완료"

# ========== 3단계: 데이터 전송 ==========
# Writing objects: 100% (2/2), 279 bytes | 279.00 KiB/s, done.
# Total 2 (delta 1), reused 0 (delta 0), reusing all objects
# → 로컬: "279바이트 전송 완료"

# ========== 4단계: 참조 업데이트 ==========
# To github.com:your_username/test_ssh.git
#    abc1234..def5678  main -> main
# → GitHub: "main 브랜치 업데이트 완료"
# → abc1234 커밋에서 def5678 커밋으로 진행

# ========== 성공! ==========
```

✅ **Push 성공!** 🎉

---

## 🔐 이 과정에서 일어난 보안 메커니즘

### 🔍 당신이 본 것:

```
$ git push origin main
```

### 🔐 뒤에서 일어난 것:

```
1️⃣ SSH 세션 시작
   로컬 ←→ GitHub (TCP 연결)
   
2️⃣ GitHub의 호스트 공개키 수신
   로컬: "너 정말 GitHub 맞아?"
   
3️⃣ 사용자 인증 시작
   GitHub: "너는 누구야?" → 논스 전송
           (예: nonce=a7f3k9m2x1)
   
4️⃣ 로컬이 개인키로 서명
   로컬: "나는 your_username"
         "시간: 2024-01-15 10:30:45"
         "논스: a7f3k9m2x1"
         → 개인키로 서명
   
5️⃣ GitHub가 공개키로 검증
   GitHub: ✅ 서명 유효?
           ✅ 논스 일회용?
           ✅ 타임스탐프 최근?
           ✅ 세션 일치?
           → 모두 통과!
   
6️⃣ 인증 완료 → Push 진행
   데이터 전송 → 커밋 저장
```

### 🚫 공격자가 도청했다면?

```
공격자: "아, 이 서명값을 재전송하면 되겠네!"
        → 도청한 서명값 재전송

GitHub: 검증
  ✅ 서명 유효? (네, 맞습니다)
  ❌ 논스 일회용? (아니, 이미 사용됨!)
  → "거부!" 🚫

또는

GitHub: 검증
  ✅ 서명 유효? (네, 맞습니다)
  ✅ 논스 일회용? (네, 처음입니다)
  ❌ 타임스탐프 최근? (아니, 30분 전!)
  → "거부!" 🚫
```

---

## ✅ Step 6: GitHub에서 확인

```bash
# 1. GitHub 웹사이트 접속
# https://github.com/your_username/E1-1

# 2. SSH_TEST.md 파일이 보이나요?
# 3. 커밋 메시지 확인
#    "SSH 테스트: 타임스탐프와 논스 검증 실습"
```

---

## 📝 실습 정리

| 단계 | 명령어 | 의미 |
|------|--------|------|
| 1 | `ssh-keygen -t ed25519 -C "email"` | SSH 키 생성 |
| 2 | GitHub Settings에 공개키 등록 | 공개키 등록 |
| 3 | `ssh -T git@github.com` | SSH 연결 테스트 |
| 4 | `git remote set-url origin git@...` | 저장소를 SSH로 변경 |
| 5 | `git add`, `git commit` | 파일 커밋 |
| 6 | `git push origin main` | **SSH로 Push** |

---




##  🎓 인증 방식 차이와 보안 습관

### 🔴 HTTPS 방식의 문제점

```
1. 매번 비밀번호 입력 필요
   → 피로도 높음
   → 자동 저장 유혹 (위험!)

2. 비밀번호 탈취 위험
   - 키로거 감염 시 비밀번호 노출
   - 공용 PC에서 입력 시 위험
   - GitHub 계정 전체 장악 가능

3. 토큰 관리 복잡
   - Personal Access Token 필요
   - 만료 관리 필요
   - 재생성 번거로움
```

### ✅ SSH 방식의 장점

```
1. 비밀번호 입력 없음
   → 편리함
   → 자동화 가능 (CI/CD)

2. 개인키 탈취 위험 낮음
   - 개인키는 로컬에만 존재
   - 네트워크 전송 안 함
   - 필요시 특정 키만 삭제 가능

3. 다중 키 관리 가능
   - 노트북용 키
   - 데스크톱용 키
   - CI/CD용 키
   → 각각 독립적으로 관리/삭제 가능

4. 보안 강화 가능
   - 개인키에 패스프레이즈 설정
   - 키 로테이션 용이
```

### 🛡️ 보안 습관 체크리스트

```bash
# ✅ 해야 할 것

# 1. 개인키 권한 확인 (600 = 소유자만 읽기/쓰기)
$ ls -la ~/.ssh/id_ed25519
-rw------- 1 username username 419 Jan 15 10:30 /home/username/.ssh/id_ed25519
#  ↑ 이 부분이 rw------- 이어야 함

# 2. 공개키만 공유 (id_ed25519.pub는 OK)
$ ls -la ~/.ssh/id_ed25519.pub
-rw-r--r-- 1 username username 102 Jan 15 10:30 /home/username/.ssh/id_ed25519.pub
#  ↑ 이 부분은 rw-r--r-- 이어도 OK

# 3. 주기적으로 키 로테이션 (1년마다)
# - 새 키 생성
# - GitHub에 새 키 등록
# - 로컬에서 새 키 사용 확인
# - 기존 키 삭제

# 4. 사용하지 않는 키 삭제
$ rm ~/.ssh/old_key
$ rm ~/.ssh/old_key.pub
```

```bash
# ❌ 하지 말아야 할 것

# 1. 개인키를 GitHub에 업로드
#    → 절대 금지! 계정 탈취됨

# 2. 개인키를 이메일/메시지로 전송
#    → 절대 금지!

# 3. 개인키를 공용 PC에 저장
#    → 위험!

# 4. 개인키 권한을 644로 변경
$ chmod 644 ~/.ssh/id_ed25519  # ❌ 위험!
#    → 다른 사용자가 읽을 수 있음

# 5. 패스프레이즈 없이 방치
#    → 누군가 PC 접근 시 개인키 사용 가능
```

---

## 💡 실전 팁

### 🔧 SSH 에이전트로 편의성 높이기

```bash
# SSH 에이전트 시작 (자동으로 개인키 관리)
$ eval "$(ssh-agent -s)"
Agent pid 12345

# 개인키를 에이전트에 등록
$ ssh-add ~/.ssh/id_ed25519
Enter passphrase for /home/username/.ssh/id_ed25519: 
Identity added: /home/username/.ssh/id_ed25519 (your_email@example.com)

# 이제 패스프레이즈 한 번만 입력하면 됨
# 세션 동안 계속 사용 가능
```

### 📋 여러 GitHub 계정 관리

```bash
# 키 생성 (계정별로)
$ ssh-keygen -t ed25519 -C "personal@gmail.com" -f ~/.ssh/id_personal
$ ssh-keygen -t ed25519 -C "work@company.com" -f ~/.ssh/id_work

# ~/.ssh/config 파일 생성
$ cat > ~/.ssh/config << EOF
# Personal Account
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_personal

# Work Account
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_work
EOF

# 권한 설정
$ chmod 600 ~/.ssh/config

# 사용 예시
$ git clone git@github.com-personal:your_username/personal_repo.git
$ git clone git@github.com-work:your_company/work_repo.git
```

---

## 📝 정리

| 항목 | 내용 |
|------|------|
| **SSH 핵심** | 공개키-개인키 쌍으로 인증 (비밀번호 불필요) |
| **보안 강점** | 개인키는 로컬만, 네트워크 전송 안 함 |
| **편의성** | 한 번 설정 후 계속 사용 가능 |
| **실무** | 거의 모든 개발자가 SSH 사용 |
| **필수 습관** | 개인키 보호, 주기적 로테이션 |