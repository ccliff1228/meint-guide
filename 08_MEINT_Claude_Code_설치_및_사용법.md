# Claude Code 설치 및 기본 사용법

## 개요

Claude Code는 Anthropic이 만든 **AI 코딩 에이전트**다. 터미널이나 VS Code에서 자연어로 지시하면, 코드를 읽고 수정하고 실행까지 해주는 AI 비서 역할을 한다. MEINT 프로젝트의 모든 실습은 Claude Code를 통해 이루어진다.

**Claude Code를 사용하는 이유:**

- **자연어 개발** — 한국어로 지시하면 코드를 자동 생성/수정
- **전체 프로젝트 이해** — 파일 하나가 아닌 프로젝트 전체를 읽고 맥락을 파악
- **도구 통합** — Git, 테스트, 외부 DB까지 하나의 인터페이스에서 처리
- **MCP 연동** — 외부 시스템(MES4 DB, ERPNext API)에 AI가 직접 접근 가능

---

## 1. 사전 준비

Claude Code를 설치하기 전에 다음 항목을 확인한다.

### 필수 요구사항

- **운영체제** — Windows 10 1809 이상, macOS 13.0 이상, Ubuntu 20.04 이상
- **메모리** — 4GB RAM 이상
- **인터넷** — 상시 연결 필요
- **Claude 계정** — Pro, Max, 또는 Team 구독 필요 (무료 플랜에서는 사용 불가)

### Windows 추가 요구사항

- **Git for Windows** — Claude Code는 내부적으로 `git`과 `bash`를 사용한다. [git-scm.com](https://git-scm.com) 에서 다운로드하여 설치한 뒤, PowerShell에서 `git --version`으로 확인한다. Git for Windows가 제공하는 Git Bash가 Claude Code의 내부 셸 환경으로 사용된다.

### 권장 사항

- **VS Code** — 1.98 이상 설치 권장 (Claude Code 확장과 연동)
- **VS Code 설치 방법** — `08_MEINT_VSCode_설치_및_사용법.md` 참조

> Claude Pro/Max 구독이 없다면 `06_MEINT_Claude_Gift_Redeem_Guide.md`를 참고하여 선물 구독을 활성화한다.

---

## 2. 설치 방법

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

**대안 설치 방법:**

- **Homebrew (macOS)** — `brew install --cask claude-code`
- **WinGet (Windows)** — `winget install Anthropic.ClaudeCode`

### Step 2: 설치 확인

새 PowerShell 창을 열고 다음 명령을 실행한다.

```powershell
claude --version
```

버전 번호가 출력되면 설치가 완료된 것이다.

### Step 3: 설치 진단

문제가 있다면 PowerShell에서 종합 진단 도구를 실행한다.

```powershell
claude doctor
```

`claude doctor`는 설치 상태, 인증, 네트워크, MCP 서버를 한 번에 진단하고 해결 방법을 안내한다.

---

## 3. 인증(로그인)

Claude Code를 사용하려면 Claude 계정으로 인증해야 한다. PowerShell에서 Claude Code를 처음 실행하면 자동으로 브라우저가 열린다.

```powershell
claude
```

1. 브라우저가 열리면 Claude.ai 계정으로 로그인
2. 인증 완료 후 터미널로 돌아오면 사용 준비 완료

> 브라우저가 자동으로 열리지 않으면 `c` 키를 눌러 인증 URL을 복사한 뒤, 브라우저에 직접 붙여넣는다.

### 인증 상태 확인 및 관리

PowerShell에서 다음 명령을 실행한다.

```powershell
claude auth status    # 현재 인증 상태 확인
claude auth login     # 다시 로그인
claude auth logout    # 로그아웃
```

---

## 4. 기본 사용법

### 4-1. CLI 실행 및 프롬프트 입력

PowerShell에서 프로젝트 디렉토리로 이동한 뒤 `claude`를 실행하면 대화형 세션이 시작된다.

```powershell
cd my-project
claude
```

> Claude Code가 시작되면 내부 셸은 자동으로 **bash**(Git Bash)로 전환된다. 이후 Claude Code 안에서 AI가 실행하는 명령은 bash 문법을 따른다.

프롬프트가 나타나면 자연어로 요청을 입력한다.

```
> 이 프로젝트의 구조를 설명해줘
> README.md를 한글로 번역해줘
> 테스트를 실행해줘
```

**초기 프롬프트와 함께 실행 (PowerShell에서):**

```powershell
claude "이 프로젝트의 구조를 설명해줘"
```

### 4-2. @ 파일 참조

`@`를 사용하면 특정 파일을 AI에게 직접 전달할 수 있다.

```
> @src/config.py 에서 DB 연결 부분을 설명해줘
> @src/main.py#10-25 이 부분의 에러 핸들링을 개선해줘
```

> `@` 참조는 AI가 정확한 파일을 읽도록 보장한다. 파일명만 언급하는 것보다 정확도가 높다.

### 4-3. 코드 변경 승인/거부

Claude가 코드를 수정하면 **diff 뷰**가 표시된다.

- **Accept** — 변경 적용
- **Reject** — 변경 거부
- 각 파일별로 **개별 승인/거부** 가능

> AI가 수정안을 "제안"하고, 여러분이 "결재(승인/반려)"하는 구조다. 항상 변경 내용을 확인한 뒤 승인한다.

### 4-4. 권한 모드

세 가지 권한 모드로 AI의 행동 범위를 조절한다.

- **Default** — 도구 사용 시마다 확인을 요청한다. 일반 작업에 적합
- **Plan** — 읽기만 허용하고 코드 변경을 차단한다. 코드 분석이나 설계 수립에 적합
- **Auto-accept** — 모든 도구 사용을 자동 승인한다. 신뢰할 수 있는 반복 작업에 적합

모드 전환 방법:

- CLI에서 `Shift+Tab` 키로 전환
- `/plan` 슬래시 명령으로 Plan 모드 진입

### 4-5. 슬래시 명령

대화 중 `/`로 시작하는 명령을 사용할 수 있다.

**자주 사용하는 명령:**

- `/help` — 사용 가능한 명령어 표시
- `/clear` — 대화 컨텍스트 초기화 (새 주제로 넘어갈 때)
- `/compact` — 대화 기록 압축 (대화가 길어져 느려질 때)
- `/plan` — Plan 모드 진입 (읽기 전용)
- `/model` — 모델 전환 (sonnet, opus, haiku)
- `/commit` — 변경 사항을 Git 커밋
- `/pr` — Pull Request 생성
- `/review` — 코드 리뷰 수행
- `/memory` — 메모리 파일 확인/편집
- `/doctor` — 설치 및 설정 진단
- `/thinking` — Extended Thinking 토글 (복잡한 문제에 유용)
- `/usage` — 사용량 확인

### 4-6. 대화 이어가기

PowerShell에서 이전 대화를 이어서 작업할 수 있다.

```powershell
claude -c              # 가장 최근 대화 이어가기
claude -r "session-id" # 특정 세션 재개
```

---

## 5. VS Code 연동

VS Code에서 Claude Code를 사용하면 에디터와 AI를 한 화면에서 오갈 수 있다.

### Step 1: 확장 설치

1. VS Code에서 `Ctrl+Shift+X` → Extensions 패널 열기
2. **"Claude Code"** 검색
3. **Anthropic** 게시자의 확장을 설치 (`anthropic.claude-code`)

### Step 2: Claude Code 열기

설치 후 다음 세 가지 방법으로 연다.

- **Spark 아이콘** — 에디터 우측 상단 도구 모음의 반짝임 아이콘 클릭
- **상태바** — 하단 상태바의 Claude Code 항목 클릭
- **명령 팔레트** — `Ctrl+Shift+P` → "Claude Code: Open in New Tab" 입력

### Step 3: 사용하기

사이드바에 Claude 채팅 패널이 나타난다. 여기에 자연어로 지시를 입력하면 된다.

**VS Code 확장의 주요 기능:**

- diff 뷰로 코드 변경 사항 확인
- `@` 파일 참조 (에디터에서 열린 파일을 쉽게 참조)
- 대화 기록 저장 및 재개
- 여러 대화를 탭으로 동시 관리
- 체크포인트로 되감기 가능

---

## 6. 프로젝트 설정

### 6-1. CLAUDE.md 생성

`CLAUDE.md`는 AI에게 프로젝트의 맥락을 알려주는 지침 파일이다. 프로젝트 루트에 두면 Claude Code가 매 세션 시작 시 자동으로 읽는다.

```bash
# 프로젝트 구조를 분석하여 자동 생성
/init
```

**CLAUDE.md에 담는 내용:**

- **기술 스택** — Python, React, ERPNext v17 등
- **빌드/실행 명령** — `bench start`, `npm run dev` 등
- **코딩 컨벤션** — 한국어 주석, Black 포매터 사용 등
- **프로젝트 구조** — 주요 디렉토리와 역할 설명

> CLAUDE.md는 새 직원에게 주는 "프로젝트 안내 가이드"와 같다. AI가 이것을 읽고 프로젝트 맥락을 파악한다.

### 6-2. 권한 설정

`.claude/settings.json`에서 AI가 사용할 수 있는 도구를 제어한다.

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Bash(git *)",
      "Bash(npm run *)",
      "Bash(pytest *)"
    ],
    "deny": [
      "Read(./.env)",
      "Write(./production.config.*)"
    ]
  }
}
```

- **allow** — 확인 없이 자동 허용할 도구
- **deny** — 항상 차단할 도구
- **평가 순서** — `deny` → `ask` → `allow` (deny가 항상 우선)

**설정 파일 위치:**

- `~/.claude/settings.json` — 사용자 전체 설정
- `.claude/settings.json` — 프로젝트 공유 설정 (Git에 포함)
- `.claude/settings.local.json` — 개인 로컬 설정 (Git에서 제외)

---

## 7. 실습 튜토리얼: 파이썬 코딩 파이프라인

이 섹션에서는 **가위바위보 게임**을 만들면서 AI 코딩의 5단계 파이프라인을 체험한다. 별도 파일 없이 빈 폴더에서 Claude Code 프롬프트만으로 진행한다.

**파이프라인 개요:**

- **이해** → **설계** → **구현** → **검증** → **디버깅**

### 사전 준비: 파이썬 설치 확인

이 튜토리얼은 파이썬(Python)으로 코딩한다. 먼저 파이썬이 설치되어 있는지 확인한다.

**설치 여부 확인 (PowerShell에서 실행):**

```powershell
python --version
```

버전 번호(예: `Python 3.12.x`)가 출력되면 설치가 완료된 상태다. **Python 3.6 이상**이면 이 튜토리얼을 수행할 수 있다. 표준 라이브러리(`random` 모듈)만 사용하므로 추가 패키지 설치는 불필요하다.

**`python`을 찾을 수 없다는 오류가 나오는 경우:**

1. [python.org/downloads](https://www.python.org/downloads/) 에서 최신 버전을 다운로드
2. 설치 시 **"Add python.exe to PATH"** 체크박스를 반드시 선택
3. 설치 완료 후 **PowerShell을 새로 열고** `python --version`으로 재확인

> **PATH 체크박스를 놓쳤다면:** 설치 파일을 다시 실행 → "Modify" 선택 → "Add Python to environment variables" 체크 후 완료. 또는 Microsoft Store에서 "Python"을 검색하여 설치하면 PATH가 자동으로 설정된다.

**프로젝트 폴더 생성 및 Claude Code 시작:**

```powershell
mkdir rps-game
cd rps-game
claude
```

### 7-1. 이해 — 요구사항 분석

먼저 `Shift+Tab`으로 **Plan 모드**로 전환한다. Plan 모드에서는 AI가 코드를 수정하지 않고 분석과 설계만 수행한다.

> 가위바위보 게임을 파이썬으로 만들고 싶어. 어떤 기능이 필요한지 정리해줘

**기대 결과:** Claude가 필요한 기능 목록을 정리해준다 — 사용자 입력 받기, 컴퓨터 랜덤 선택, 승패 판정, 결과 출력, 재시작 여부 등.

**학습 포인트:**

- `Shift+Tab`으로 Plan 모드 전환 → AI가 읽기만 하고 코드를 건드리지 않음
- 자연어로 요구사항을 전달하면 AI가 구조화된 분석 결과를 제공

### 7-2. 설계 — 코드 구조 잡기

Plan 모드를 유지한 채 설계를 요청한다.

> 방금 정리한 기능을 바탕으로 코드 구조를 설계해줘

**기대 결과:** Claude가 파일 구성, 함수 목록, 게임 흐름(루프 구조)을 제안한다.

**학습 포인트:**

- Plan 모드에서 이전 대화 맥락을 이어가며 설계 진행
- AI의 설계 제안을 검토하고, 마음에 들지 않으면 추가 요청으로 수정 가능

### 7-3. 구현 — 코드 생성

`Shift+Tab`으로 **Default 모드**로 전환한 뒤 구현을 요청한다.

> 설계대로 구현해줘

**기대 결과:** Claude가 `rps_game.py` 파일을 생성하고, **diff 뷰**로 코드 내용을 보여준다. Accept를 눌러 승인하면 파일이 생성된다.

**학습 포인트:**

- Default 모드에서는 AI가 파일을 생성/수정할 수 있음
- diff 뷰에서 코드를 확인한 뒤 **Accept**(승인) 또는 **Reject**(거부) 선택
- AI가 제안하고 사람이 결재하는 구조 — 항상 내용을 확인한 뒤 승인

### 7-4. 검증 — 실행 및 테스트

게임을 실행하고, 자동화 테스트도 만들어본다.

> 실행해봐

Claude가 `python rps_game.py`를 실행한다. 직접 가위/바위/보를 입력하여 게임을 해본다.

> 테스트 코드를 만들고 실행해줘

`pytest`가 설치되어 있지 않으면 Claude가 자동으로 `pip install pytest`를 실행한다. 승인하면 된다.

**기대 결과:** Claude가 테스트 파일을 생성하고 `pytest`로 실행한다. 승/패/무승부 판정이 올바른지 자동으로 검증된다.

**학습 포인트:**

- Claude Code가 Bash 도구로 `python`, `pytest` 등을 직접 실행
- 실행 결과가 터미널에 바로 표시됨
- 테스트 자동 생성으로 수동 확인 없이 로직 검증 가능

### 7-5. 디버깅 — 버그 수정 체험

의도적으로 버그를 넣은 뒤, AI에게 찾아서 고치도록 요청한다.

> 무승부일 때 '패배'라고 출력되도록 버그를 넣어줘

Claude가 코드를 수정하여 의도적인 버그를 삽입한다. Accept로 승인한다.

> 게임을 실행해서 테스트해봐

테스트가 실패하는 것을 확인한다.

> 버그를 찾아서 고쳐줘

**기대 결과:** Claude가 코드를 분석하여 무승부 판정 로직의 오류를 찾아내고, 수정안을 diff 뷰로 보여준다.

**학습 포인트:**

- AI가 코드를 읽고 버그의 원인을 분석하는 과정 확인
- 수정 전후를 diff 뷰로 비교하여 어디가 바뀌었는지 명확히 파악
- 실제 개발에서 "에러 메시지를 붙여넣고 고쳐달라"는 식으로 디버깅 가능

### 실습 정리

5단계 파이프라인과 Claude Code 기능의 대응:

- **이해** — Plan 모드 + 자연어 대화로 요구사항 분석
- **설계** — Plan 모드에서 구조 설계 및 검토
- **구현** — Default 모드에서 코드 생성 + diff 뷰 승인
- **검증** — Bash 도구로 실행 및 테스트 수행
- **디버깅** — 코드 분석 + 버그 식별 + 수정 제안

> 이 5단계는 가위바위보뿐 아니라, MEINT 프로젝트의 모든 개발 작업에 동일하게 적용된다. 이후 실습에서도 이 파이프라인을 반복하게 된다.

---

## 8. 필수 단축키 정리

### CLI 모드

- `Shift+Tab` — 권한 모드 전환 (Default / Plan / Auto-accept)
- `Tab` — Extended Thinking 토글
- `Esc` (2회) — 대화 되감기 (이전 시점으로 복원)
- `?` — 사용 가능한 단축키 확인
- `/` — 슬래시 명령 목록 표시

### VS Code 확장

- `Ctrl+Esc` — 에디터 ↔ Claude 프롬프트 포커스 전환
- `Ctrl+Shift+Esc` — Claude를 에디터 탭으로 열기
- `Alt+K` — `@` 파일 참조 삽입
- `Ctrl+N` — 새 대화 시작
- `Shift+Enter` — 여러 줄 프롬프트 입력

---

## 9. 문제 해결

### 자주 발생하는 문제

**`claude: command not found` (명령어를 찾을 수 없음)**

1. 새 터미널 창을 열고 다시 시도
2. PATH에 `~/.local/bin`이 포함되어 있는지 확인
3. Windows PowerShell에서 PATH 추가:

```powershell
$currentPath = [Environment]::GetEnvironmentVariable('PATH', 'User')
[Environment]::SetEnvironmentVariable('PATH', "$currentPath;$env:USERPROFILE\.local\bin", 'User')
```

**`irm`이 인식되지 않음 (Windows)**

- CMD가 아닌 **PowerShell**에서 실행해야 한다

**브라우저 인증이 실패하는 경우**

1. `claude doctor` 실행 후 안내에 따라 조치
2. `claude auth logout` 후 `claude auth login`으로 재시도
3. 브라우저가 열리지 않으면 `c` 키를 눌러 URL 복사 후 수동 접속

**VS Code 확장이 보이지 않는 경우**

1. VS Code 버전이 1.98 이상인지 확인 (좌측 하단 기어 아이콘 → 정보)
2. Extensions에서 "Claude Code"를 다시 검색하여 재설치
3. VS Code를 완전히 종료 후 재시작

**대화가 느려지는 경우**

- `/compact` — 대화를 압축하여 핵심만 유지
- `/clear` — 대화를 완전히 초기화하고 새로 시작

**Windows에서 MCP 서버 연결 실패**

- `.mcp.json`에서 `cmd /c` 래퍼를 사용하여 명령을 감싸야 할 수 있다

### 진단 도구

문제가 발생하면 PowerShell에서 가장 먼저 실행할 명령:

```powershell
claude doctor
```

설치, 인증, 네트워크, MCP 서버, 설정 파일 오류를 종합적으로 진단한다.

### 업데이트

PowerShell에서 Claude Code를 최신 버전으로 업데이트:

```powershell
claude update
```

---

## 참고 자료

- [Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Claude Code 설치 가이드](https://docs.anthropic.com/en/docs/claude-code/getting-started)
- [Claude Code CLI 사용법](https://docs.anthropic.com/en/docs/claude-code/cli-usage)
- [VS Code 확장 연동](https://docs.anthropic.com/en/docs/claude-code/ide-integrations)
- [설정 및 CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/settings)
- [문제 해결](https://docs.anthropic.com/en/docs/claude-code/troubleshooting)
