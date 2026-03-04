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

- **Git for Windows** — 설치 필요 ([git-scm.com](https://git-scm.com) 에서 다운로드)

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

새 터미널을 열고 다음 명령을 실행한다.

```bash
claude --version
```

버전 번호가 출력되면 설치가 완료된 것이다.

### Step 3: 설치 진단

문제가 있다면 종합 진단 도구를 실행한다.

```bash
claude doctor
```

`claude doctor`는 설치 상태, 인증, 네트워크, MCP 서버를 한 번에 진단하고 해결 방법을 안내한다.

---

## 3. 인증(로그인)

Claude Code를 사용하려면 Claude 계정으로 인증해야 한다.

### 방법 1: 브라우저 인증 (Pro / Max / Team)

가장 일반적인 방법이다. 터미널에서 Claude Code를 처음 실행하면 자동으로 브라우저가 열린다.

```bash
claude
```

1. 브라우저가 열리면 Claude.ai 계정으로 로그인
2. 인증 완료 후 터미널로 돌아오면 사용 준비 완료

> 브라우저가 자동으로 열리지 않으면 `c` 키를 눌러 인증 URL을 복사한 뒤, 브라우저에 직접 붙여넣는다.

### 방법 2: API Key 인증 (Console 사용자)

Anthropic Console에서 API Key를 발급받은 경우 사용한다.

```bash
claude
# 인증 선택에서 "Use an API key" 선택 → API Key 입력
```

또는 환경변수로 설정:

```bash
# Linux / macOS
export ANTHROPIC_API_KEY="sk-ant-..."

# Windows PowerShell
$env:ANTHROPIC_API_KEY = "sk-ant-..."
```

### 인증 상태 확인 및 관리

```bash
claude auth status    # 현재 인증 상태 확인
claude auth login     # 다시 로그인
claude auth logout    # 로그아웃
```

---

## 4. 기본 사용법

### 4-1. CLI 실행 및 프롬프트 입력

프로젝트 디렉토리에서 `claude`를 실행하면 대화형 세션이 시작된다.

```bash
cd my-project
claude
```

프롬프트가 나타나면 자연어로 요청을 입력한다.

```
> 이 프로젝트의 구조를 설명해줘
> README.md를 한글로 번역해줘
> 테스트를 실행해줘
```

**초기 프롬프트와 함께 실행:**

```bash
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

이전 대화를 이어서 작업할 수 있다.

```bash
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

## 7. 필수 단축키 정리

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

## 8. 문제 해결

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

문제가 발생하면 가장 먼저 실행할 명령:

```bash
claude doctor
```

설치, 인증, 네트워크, MCP 서버, 설정 파일 오류를 종합적으로 진단한다.

### 업데이트

Claude Code를 최신 버전으로 업데이트:

```bash
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
