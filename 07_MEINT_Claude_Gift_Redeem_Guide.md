# Claude 선물 구독 사용(Redeem) 가이드

## 개요

이 문서는 Claude 선물 구독을 받은 수신자가 선물을 활성화하고 Claude Code까지 사용하는 전체 과정을 안내한다.

---

## 1. 선물 Redeem 절차

### Step 1: 선물 확인

선물은 두 가지 형태 중 하나로 전달된다.

- **이메일** — Claude로부터 선물 안내 메일 수신 (스팸 폴더도 확인)
- **공유 링크** — 선물 발송자가 카카오톡, 문자 등으로 링크를 직접 전달

### Step 2: Redeem 페이지 접속

다음 중 하나의 방법으로 접속한다.

- 이메일 또는 링크에서 **"Redeem your gift"** 버튼 클릭
- 또는 브라우저에서 직접 접속: **https://claude.ai/gift/redeem**

### Step 3: Claude 계정 로그인

- 기존 계정이 있으면 로그인
- 계정이 없으면 **신규 가입** (이메일 또는 Google 계정으로 가입 가능)

### Step 4: 선물 코드 입력 및 확인

- 링크를 통해 접속했으면 자동으로 코드가 적용됨
- 직접 redeem 페이지에 접속한 경우 **선물 코드를 수동 입력**
- "Redeem" 버튼 클릭으로 활성화 완료

### Step 5: 구독 활성화 확인

- 좌측 하단 프로필 아이콘 클릭 → 현재 플랜이 Pro/Max로 변경되었는지 확인
- **신용카드 등록 불필요** — 선물 기간 동안 결제 정보 없이 사용 가능

---

## 2. Claude Code 설치 및 사용

선물로 Pro 이상 플랜이 활성화되면 Claude Code를 바로 사용할 수 있다. 상세한 설치 방법은 `08_MEINT_Claude_Code_설치_및_사용법.md`를 참조하고, 여기서는 빠른 시작 절차만 안내한다.

### 사전 준비: Git for Windows 설치 (Windows만 해당)

Windows에는 Git이 기본 설치되어 있지 않다. Claude Code는 내부적으로 `git`과 `bash`를 사용하므로, 먼저 Git for Windows를 설치해야 한다.

1. [git-scm.com](https://git-scm.com) 에서 다운로드 및 설치
2. 설치 완료 후 PowerShell에서 확인:

```powershell
git --version
```

버전이 출력되면 정상 설치된 것이다. Git for Windows가 제공하는 Git Bash가 Claude Code의 내부 셸 환경으로 사용된다.

### Step 1: Claude Code 설치

**Windows (PowerShell):**

```powershell
irm https://claude.ai/install.ps1 | iex
```

> 반드시 **PowerShell**에서 실행해야 한다. CMD(명령 프롬프트)에서는 `irm` 명령이 동작하지 않는다.

**macOS / Linux:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### Step 2: 설치 확인

새 PowerShell 창을 열고 다음 명령을 실행한다.

```powershell
claude --version
```

버전 번호가 출력되면 설치가 완료된 것이다.

### Step 3: 로그인

PowerShell에서 다음 명령을 실행한다.

```powershell
claude
```

브라우저가 자동으로 열리면 Claude.ai 계정으로 로그인한다. 선물 구독이 활성화된 동일 계정으로 로그인해야 한다.

> 브라우저가 자동으로 열리지 않으면 `c` 키를 눌러 인증 URL을 복사한 뒤, 브라우저에 직접 붙여넣는다.

### Step 4: 사용 시작

PowerShell에서 프로젝트 디렉토리로 이동한 뒤 실행:

```powershell
cd my-project
claude
```

Claude Code가 시작되면 내부 셸은 자동으로 **bash**(Git Bash)로 전환된다. 이후 Claude Code 안에서의 명령은 bash 문법을 따른다.

---

## 3. 기존 구독 상태별 처리

수신자의 현재 구독 상태에 따라 선물 적용 방식이 다르다.

### 신규 또는 무료 사용자

- 즉시 선물 플랜(Pro/Max)으로 업그레이드
- 추가 조치 불필요

### 웹에서 Pro 구독 중인 경우

- **Pro 선물 수신** → 기존 구독 기간에 선물 기간이 **연장**됨
- **Max 선물 수신** → Max로 임시 업그레이드, 선물 기간 종료 후 기존 Pro로 복귀

### 웹에서 Max 구독 중인 경우

- **동일 또는 상위 플랜 선물** → 구독 기간 연장 또는 업그레이드
- **하위 플랜 선물** → 크레딧으로 변환되어 향후 청구액에 적용

### 모바일 앱(iOS/Android) 구독 중인 경우

- 모바일 구독이 **종료된 후**에만 선물 redeem 가능
- 모바일 앱에서 자동 갱신을 먼저 해제해야 함

---

## 4. 주의사항

- **유효기간** — 선물은 구매일로부터 **365일 이내**에 사용해야 함 (기간 경과 시 만료)
- **중복 적용 불가** — 여러 선물을 동시에 redeem할 수 없음 (하나씩 순차 사용)
- **웹에서만 가능** — redeem은 **브라우저**(claude.ai)에서만 가능, 모바일 앱에서는 불가
- **계정 무관** — 선물 이메일을 받은 주소와 다른 Claude 계정으로도 redeem 가능

---

## 5. 문제 해결

### 선물 이메일이 보이지 않는 경우

1. 스팸/정크 폴더 확인
2. 선물 발송자에게 수신 이메일 주소 재확인 요청
3. 발송자가 공유 링크 방식으로 재전달

### Redeem 시 오류가 발생하는 경우

1. 모바일 앱이 아닌 **웹 브라우저**에서 시도
2. 기존 모바일 구독이 활성 상태인지 확인 (활성이면 해지 후 시도)
3. 이미 다른 선물이 적용 중인지 확인

### 선물 기간 종료 후

- 자동으로 무료 플랜으로 전환됨
- 별도 결제 정보를 등록하지 않았다면 **자동 과금 없음**
- 계속 사용하려면 직접 유료 플랜 구독 필요

---

## 참고 자료

- [Claude 선물 구독 Redeem 방법 (공식)](https://support.claude.com/en/articles/12938695-how-to-redeem-a-claude-gift-subscription)
- [Claude 선물 구독 구매 방법 (공식)](https://support.claude.com/en/articles/12938627-how-to-gift-a-claude-subscription)
- [Claude Code 설정 가이드 (공식)](https://code.claude.com/docs/en/setup.md)
