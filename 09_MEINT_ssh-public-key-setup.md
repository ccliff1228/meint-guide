# SSH 공개키 로그인 설정 가이드

## 개요

비밀번호 대신 SSH 공개키(ed25519)를 사용하여 원격 서버에 로그인하는 방법을 단계별로 안내한다. 비밀번호 접속 확인부터 키 생성, 서버 등록, 접속 편의 설정, 비밀번호 로그인 비활성화까지 전체 과정을 다룬다.

각 단계에서 **PowerShell**(Windows PC)과 **Linux**(서버)에서 실행하는 작업을 구분하여 표시한다.

## 목차

- [서버 정보](#서버-정보)
- [1단계: 비밀번호 로그인 확인](#1단계-비밀번호-로그인-확인) — PowerShell
- [2단계: SSH 키 생성](#2단계-ssh-키-생성) — PowerShell
- [3단계: 공개키를 서버에 등록](#3단계-공개키를-서버에-등록) — PowerShell + Linux
- [4단계: 공개키 로그인 확인](#4단계-공개키-로그인-확인) — PowerShell
- [5단계: SSH 접속 편의 설정 (선택)](#5단계-ssh-접속-편의-설정-선택) — PowerShell
- [6단계: 비밀번호 로그인 비활성화 (서버 관리자)](#6단계-비밀번호-로그인-비활성화-서버-관리자) — Linux

---

## 서버 정보

- **서버 IP**: 223.195.110.31
- **SSH 포트**: 4922
- **계정**: meint01 ~ meint04 (초기 비밀번호: 계정명과 동일)

---

## 1단계: 비밀번호 로그인 확인

> **실행 환경: PowerShell (Windows PC)**

공개키를 등록하기 전에, 먼저 아이디/비밀번호로 서버에 접속되는지 확인한다.

```powershell
ssh -p 4922 meint01@223.195.110.31
```

- 비밀번호 입력 프롬프트가 나오면 초기 비밀번호(`meint01`)를 입력한다.
- 정상 접속되면 `exit`로 빠져나온다.

### 접속이 안 되는 경우 확인사항

- **`Connection refused`**: 서버의 SSH 서비스가 중지되었거나 포트(4922)가 열려있지 않음
- **`Connection timed out`**: 방화벽에서 포트 4922가 차단됨, 네트워크 연결 확인
- **`Permission denied`**: 아이디 또는 비밀번호가 틀림

> 접속이 확인된 후 다음 단계로 진행하세요.

---

## 2단계: SSH 키 생성

> **실행 환경: PowerShell (Windows PC)**

각 사용자가 **자신의 Windows PC**에서 실행한다.

```powershell
ssh-keygen -t ed25519 -C "meint01@mypc"
```

- 저장 경로: 기본값(Enter) → `C:\Users\사용자명\.ssh\id_ed25519`
- 암호(passphrase): 선택사항 (보안 강화 시 설정)

생성 결과 확인:

```powershell
ls ~/.ssh/
# id_ed25519       ← 개인키 (절대 공유 금지)
# id_ed25519.pub   ← 공개키 (서버에 등록할 파일)
```

---

## 3단계: 공개키를 서버에 등록

Windows에는 `ssh-copy-id`가 없으므로 수동으로 등록한다.

### 3-1. 공개키 내용 확인

> **실행 환경: PowerShell (Windows PC)**

```powershell
cat ~/.ssh/id_ed25519.pub
```

출력된 공개키 전체(`ssh-ed25519 AAAA...`)를 복사한다.

### 3-2. 서버에 접속하여 공개키 등록

> **실행 환경: PowerShell → Linux (서버 접속 후)**

```powershell
# PowerShell에서 서버에 접속
ssh -p 4922 meint01@223.195.110.31
```

접속 후 서버(Linux)에서 다음을 실행한다:

```bash
# 서버(Linux)에서 실행
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "복사한_공개키_내용" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

---

## 4단계: 공개키 로그인 확인

> **실행 환경: PowerShell (Windows PC)**

```powershell
ssh -p 4922 meint01@223.195.110.31
```

**비밀번호를 묻지 않고** 바로 접속되면 성공이다.

만약 여전히 비밀번호를 묻는다면 디버그 모드로 원인을 파악한다:

```powershell
ssh -v -p 4922 meint01@223.195.110.31
```

- **`Offering public key` 후 `rejected`**: 서버의 `authorized_keys`에 공개키가 제대로 등록되지 않음
- **공개키 시도 자체가 없음**: Windows PC에 `~/.ssh/id_ed25519` 파일이 없거나 경로가 다름
- **서버 권한 문제**: 서버에서 `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` 실행

---

## 5단계: SSH 접속 편의 설정 (선택)

> **실행 환경: PowerShell (Windows PC)**

Windows PC의 `~/.ssh/config` 파일에 아래 내용을 추가하면 간단하게 접속할 수 있다:

```
Host meint
    HostName 223.195.110.31
    Port 4922
    User meint01
    IdentityFile ~/.ssh/id_ed25519
```

이후 아래 명령만으로 접속 가능:

```powershell
ssh meint
```

---

## 6단계: 비밀번호 로그인 비활성화 (서버 관리자)

> **실행 환경: Linux (서버)**

> **주의:** 반드시 모든 계정(meint01~04)의 공개키 로그인이 정상 동작하는 것을 확인한 후 진행하세요.
> 이 설정 후에는 공개키가 없으면 접속이 불가능합니다.

서버에서 실행:

```bash
sudo nano /etc/ssh/sshd_config
```

다음 항목을 수정:

```
Port 4922
PasswordAuthentication no
PubkeyAuthentication yes
```

SSH 서비스 재시작:

```bash
sudo systemctl restart sshd
```

**현재 세션을 유지한 채** 새 터미널(PowerShell)에서 공개키 접속이 되는지 반드시 확인한다:

```powershell
ssh -p 4922 meint01@223.195.110.31
```
