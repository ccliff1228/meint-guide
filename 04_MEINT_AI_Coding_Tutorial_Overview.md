---
marp: true
theme: default
paginate: true
backgroundColor: #fff
style: |
  section {
    font-family: 'Noto Sans KR', sans-serif;
  }
  h1 { color: #1a5f7a; }
  h2 { color: #2c3e50; }
  table { font-size: 0.85em; }
  code { font-size: 0.85em; }
  blockquote { font-size: 0.9em; border-left: 4px solid #1a5f7a; }
  .analogy { background: #f0f7fa; padding: 12px; border-radius: 8px; margin: 8px 0; }
---

<!-- _class: lead -->
# AI Coding Tutorial

## Claude Code로 시작하는 AI 코딩

MEINT 프로젝트 사전 준비(Prerequisite)

---

# 학습 목표

1. **Claude Code**를 설치하고 VSCode에서 사용할 수 있다
2. 프로젝트에 맞게 **CLAUDE.md와 권한**을 설정할 수 있다
3. AI 코딩의 5단계 파이프라인(**이해→설계→구현→검증→디버깅**)을 직접 실습한다

> **이 슬라이드는 MEINT 프로젝트의 모든 실습에 앞서**
> "AI 도구를 어떻게 사용하는지"를 배우는 **사전 준비** 과정입니다

---

# 목차

| # | 섹션 | 한 줄 요약 |
|---|------|-----------|
| 1 | [Claude Code란](#1-claude-code란) | AI 코딩 에이전트 소개와 MEINT에서의 역할 |
| 2 | [설치와 인증](#2-설치와-인증) | 운영체제별 설치, 계정 인증 |
| 3 | [VSCode 연동](#3-vscode-연동) | 확장 설치, 채팅 패널 열기 |
| 4 | [프로젝트 설정](#4-프로젝트-설정) | CLAUDE.md, 권한 설정 |
| 5 | [기본 사용법](#5-기본-사용법) | 프롬프트, @참조, 권한 모드, 슬래시 명령 |
| 6 | [MCP 서버](#6-mcp-서버) | 외부 도구 연결 프로토콜 |
| 7 | [AI 코딩 파이프라인](#7-ai-코딩-파이프라인) | **핵심** — 5단계 개발 워크플로우 + 실습 |
| 8 | [Tips & 다음 단계](#8-tips--다음-단계) | 문제 해결, MEINT 프로젝트 연결 |

---

<!-- _class: lead -->
# 1. Claude Code란

AI 코딩 에이전트 소개

---

# Claude Code = AI 코딩 비서

Anthropic이 만든 **AI 코딩 에이전트(Agent)**

```
┌──────────────────────────────────────────┐
│  You (Korean)                            │
│  "이 함수에 에러 처리를 추가해줘"           │
└──────────┬───────────────────────────────┘
           │  natural language
           ▼
┌──────────────────────────────────────────┐
│  Claude Code                             │
│  - reads code         - writes code      │
│  - runs tests         - searches docs    │
│  - manages git        - connects tools   │
└──────────┬───────────────────────────────┘
           │  code changes (diff)
           ▼
┌──────────────────────────────────────────┐
│  Your Project                            │
│  Accept / Reject each change             │
└──────────────────────────────────────────┘
```

> **비유:** 코드를 읽고 쓸 줄 아는 **AI 비서**에게 한국어로 업무를 지시하는 것

---

# 왜 MEINT 실습 전에 배우나?

MEINT 프로젝트의 **모든 실습**이 Claude Code를 통해 이루어진다:

| MEINT 실습 | Claude Code가 하는 일 |
|-----------|---------------------|
| 데모 데이터 설치 | `/erpnext-demo:setup-demo` 명령 실행 |
| 공장↔사무실 동기화 | `/mes4-erpnext:sync` 동기화 프로그램 실행 |
| 대시보드 만들기 | `/react-implement:implement-project` 코드 자동 생성 |
| 문서 작성 | `/rewrite-doc:slides` 슬라이드 자동 생성 |

> **Introduction 슬라이드**에서 "AI가 무엇을 했는지"를 배웠다면,
> 이 튜토리얼에서는 **"AI를 어떻게 사용하는지"**를 배운다

---

<!-- _class: lead -->
# 2. 설치와 인증

Claude Code 설치하기

---

# 사전 요구사항

| 항목 | Windows | Linux |
|------|---------|-------|
| **OS** | Windows 10 이상 | Ubuntu 20.04+ / Debian 10+ |
| **Git** | Git for Windows 설치 필요 | 대부분 사전 설치됨 |
| **VSCode** | 1.98 이상 | 1.98 이상 |
| **계정** | Anthropic Pro / Max / Team 또는 API Key | 동일 |

> **참고:** Native Installer를 사용하면 Node.js가 필요 없다

---

# 설치 방법

### Windows (PowerShell 권장)

```powershell
irm https://claude.ai/install.ps1 | iex
```

### Linux / macOS (Native Installer 권장)

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### 설치 확인

```bash
claude --version    # 버전 확인
claude doctor       # 설치 상태 종합 진단
```

> `claude doctor`는 설치, 인증, 네트워크, MCP 서버를 한 번에 진단한다

---

# 인증 (Authentication)

### 브라우저 인증 (Pro / Max / Team / Enterprise)

```bash
claude    # 처음 실행하면 브라우저가 열림 → 로그인 → 완료
```

### API Key 인증 (Console 사용자)

```bash
claude
# 프롬프트에서 "Use an API key" 선택 → API Key 입력
```

또는 환경변수로 설정:

```bash
# Linux / macOS
export ANTHROPIC_API_KEY="sk-ant-..."

# Windows PowerShell
$env:ANTHROPIC_API_KEY = "sk-ant-..."
```

---

<!-- _class: lead -->
# 3. VSCode 연동

에디터에서 AI와 함께 일하기

---

# VSCode 확장 설치 및 열기

### 설치 (3단계)

1. VSCode에서 `Ctrl+Shift+X` → Extensions 패널
2. **"Claude Code"** 검색
3. **Anthropic** 게시자의 확장을 설치 (`anthropic.claude-code`)

### 열기 (3가지 방법)

| 방법 | 설명 |
|------|------|
| **Spark 아이콘** | 에디터 우측 상단 도구 모음의 반짝임(✦) 아이콘 클릭 |
| **상태바** | 하단 상태바의 Claude Code 항목 클릭 |
| **Command Palette** | `Ctrl+Shift+P` → "Claude Code" 입력 |

> 확장이 열리면 사이드바에 **Claude 채팅 패널**이 나타난다
> 여기에 자연어로 지시를 입력하면 AI가 코드를 읽고, 수정하고, 실행한다

---

<!-- _class: lead -->
# 4. 프로젝트 설정

AI에게 프로젝트를 알려주기

---

# CLAUDE.md = AI의 취급설명서

프로젝트 루트에 위치하는 **AI 지침 파일**

```bash
# 자동 생성: 프로젝트 구조를 분석하여 CLAUDE.md 작성
/init
```

### CLAUDE.md에 담는 내용

| 항목 | 예시 |
|------|------|
| 기술 스택 | Python, React, ERPNext v17 |
| 빌드/실행 명령 | `bench start`, `npm run dev` |
| 코딩 컨벤션 | 한국어 주석, Black 포매터 사용 |
| 프로젝트 구조 | `tools/` 동기화 도구, `apps/` ERPNext |

> **비유:** 새 직원에게 주는 **"프로젝트 안내 가이드"** — AI가 이걸 읽고 맥락을 파악한다

---

# 권한 설정 (Permissions)

### .claude/settings.json (팀 공유)

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

**규칙 평가 순서:** `deny` → `ask` → `allow` (deny가 항상 우선)

### .claude/settings.local.json (개인 설정)

`.gitignore`에 추가하여 개인 환경에 특화된 설정을 분리한다

---

<!-- _class: lead -->
# 5. 기본 사용법

AI와 대화하는 법

---

# 프롬프트 입력 & @참조

### 자연어로 요청

```
> 이 프로젝트의 테스트를 실행해줘
```

### @로 특정 파일 참조

```
> @src/config.py 에서 DB 연결 부분을 리뷰해줘
> @src/main.py#10-25 이 부분의 에러 핸들링을 개선해줘
```

### 코드 변경 리뷰

Claude가 코드를 변경하면 **diff 뷰**가 표시된다:

- **Accept** — 변경 적용
- **Reject** — 변경 거부
- 각 파일별로 **개별 승인/거부** 가능

> **비유:** AI가 수정안을 "제안"하고, 여러분이 **결재(승인/반려)** 하는 구조

---

# 권한 모드 & 슬래시 명령

### 세 가지 권한 모드

| 모드 | 설명 | 사용 시점 |
|------|------|----------|
| **Default** | 도구 사용 시마다 확인 요청 | 일반 작업 |
| **Plan** | 읽기만 허용, 쓰기 차단 | 코드 분석, 설계 수립 |
| **Auto-accept** | 모든 도구 사용 자동 승인 | 신뢰할 수 있는 반복 작업 |

### 자주 쓰는 슬래시 명령

| 명령어 | 설명 |
|--------|------|
| `/help` | 사용 가능한 명령어 표시 |
| `/clear` | 대화 컨텍스트 초기화 |
| `/compact` | 대화 기록 압축 (컨텍스트 절약) |
| `/plan` | 플랜 모드 진입 |
| `/review` | 코드 리뷰 수행 |
| `/model` | 모델 전환 (sonnet, opus, haiku) |

---

# 키보드 단축키

### VSCode 확장

| 단축키 (Win/Linux) | 설명 |
|---------------------|------|
| `Ctrl+Esc` | 에디터 ↔ Claude 프롬프트 포커스 전환 |
| `Ctrl+Shift+Esc` | Claude를 에디터 탭으로 열기 |
| `Alt+K` | `@` 파일 참조 삽입 |

### CLI 모드

| 단축키 | 설명 |
|--------|------|
| `Esc` (2회) | 대화 되감기 (이전 시점으로 복원) |
| `Shift+Tab` | 모드 전환 |
| `Tab` | Thinking 토글 |

> **Tip:** `?`를 입력하면 사용 가능한 모든 단축키를 확인할 수 있다

---

<!-- _class: lead -->
# 6. MCP 서버

AI에게 외부 도구를 연결하기

---

# MCP = AI의 팔과 다리

**Model Context Protocol** — AI가 외부 시스템(DB, API)에 접근하는 프로토콜

```
┌────────────────────┐     ┌────────────────────┐
│  Claude Code       │     │  MCP Server        │
│  (AI brain)        │────►│  (external tool)   │
│                    │◄────│                    │
└────────────────────┘     └─────────┬──────────┘
                                     │
                           ┌─────────▼──────────┐
                           │  DB / API / Files  │
                           └────────────────────┘
```

> **비유:** Claude Code가 **두뇌**라면, MCP 서버는 **팔과 다리**
> 두뇌만으로는 물건을 집을 수 없듯, MCP가 있어야 외부 데이터에 접근한다

---

# MCP 서버 추가하기

### CLI 명령으로 추가

```bash
# 로컬 서버 추가 (기본 scope: local)
claude mcp add my-server -- node /path/to/server.js

# 프로젝트 scope (팀 공유, .mcp.json에 저장)
claude mcp add -s project my-server -- node server.js

# 사용자 scope (모든 프로젝트에서 사용)
claude mcp add -s user my-server -- node server.js
```

### Scope 종류

| Scope | 저장 위치 | 공유 범위 |
|-------|----------|----------|
| **local** (기본) | 로컬 설정 | 현재 사용자, 현재 프로젝트 |
| **project** | `.mcp.json` | 팀 전체 (소스 컨트롤) |
| **user** | `~/.claude.json` | 현재 사용자, 모든 프로젝트 |

---

# MEINT의 MCP 서버들

MEINT 프로젝트에서는 3개의 MCP 서버가 연결되어 있다:

| MCP 서버 | 역할 | 명령 수 |
|---------|------|---------|
| **meint-mcp** | 제조 전용 도구 (동기화, 데모 관리, BI) | 61개 |
| **mariadb-mcp** | MES4 DB 직접 조회 (테이블 탐색, SQL 실행) | 5개 |
| **context7** | 기술 문서 검색 (라이브러리 최신 문서 조회) | 2개 |

```bash
# 등록된 MCP 서버 확인
claude mcp list
```

> MEINT에서의 MCP 서버 등록은 별도 가이드를 따른다
> 이 튜토리얼에서는 **MCP의 개념**만 이해하면 충분하다

---

<!-- _class: lead -->
# 7. AI 코딩 파이프라인

5단계 개발 워크플로우

---

# 파이프라인 전체 구조

```mermaid
flowchart LR
    A["1. Comprehend"] --> B["2. Design"] --> C["3. Implement"] --> D["4. Verify"] --> E["5. Debug"]
    style A fill:#1a5f7a,color:#fff,stroke:#1a5f7a
    style B fill:#1a5f7a,color:#fff,stroke:#1a5f7a
    style C fill:#2c3e50,color:#fff,stroke:#2c3e50
    style D fill:#2c3e50,color:#fff,stroke:#2c3e50
    style E fill:#2c3e50,color:#fff,stroke:#2c3e50
```

| 단계 | 모드 | 핵심 기능 |
|------|------|----------|
| 1. 이해(Comprehend) | Plan | `@` 파일 참조, `/compact` |
| 2. 설계(Design) | Plan | 설계안 작성, 패턴 참고 |
| 3. 구현(Implement) | Default | diff 리뷰, Accept/Reject |
| 4. 검증(Verify) | Default | `/review`, pytest |
| 5. 디버깅(Debug) | Default | 에러 분석, MCP 조회 |

- 이해(Comprehend)·설계(Design): **Plan 모드** (읽기 전용)
- 구현(Implement)·검증(Verify)·디버깅(Debug): **Default 모드** (코드 변경 가능)

> 각 단계에서 **Claude Code의 다른 기능**을 활용한다
> 이 파이프라인은 MEINT의 모든 개발 작업에 반복적으로 적용된다

---

# 실습 준비

`tutorial-examples/` 폴더에 실습 파일 3개가 준비되어 있다:

| 파일 | 용도 | 설명 |
|------|------|------|
| `inventory.py` | 이해/설계/구현 | 소규모 공장 재고 관리 모듈 (함수 6개) |
| `test_inventory.py` | 검증 | pytest 테스트 3개 + TODO 4개 |
| `buggy_inventory.py` | 디버깅 | 의도적 버그 4개가 숨겨진 버전 |

### 환경 설정

```bash
cd tutorial-examples
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt   # pytest만 설치
```

---

# inventory.py — 실습 대상 모듈

소규모 공장의 재고를 관리하는 간단한 Python 모듈:

```python
SAMPLE_ITEMS = [
    {"code": "RM-001", "name": "알루미늄 판재",  "category": "원자재", "price": 15000, "stock": 100},
    {"code": "RM-002", "name": "스테인리스 볼트", "category": "원자재", "price": 500,   "stock": 2000},
    {"code": "SA-001", "name": "모터 어셈블리",  "category": "부자재", "price": 45000, "stock": 30},
    {"code": "FG-001", "name": "소형 선풍기",    "category": "완제품", "price": 89000, "stock": 20},
    # ...
]
```

| 함수 | 역할 |
|------|------|
| `add_item()` | 새 아이템 추가 (중복·음수 검증) |
| `find_item()` | 코드로 아이템 검색 |
| `update_stock()` | 입고(+) / 출고(-) 처리 |
| `get_low_stock_items()` | 재고 부족 아이템 조회 |
| `calculate_total_value()` | 전체 재고 가치 계산 |
| `get_stock_report()` | 재고 현황 리포트 출력 |

---

# Stage 1: 코드 이해 (Comprehension)

기존 코드베이스를 파악하고 구조를 이해하는 단계

**권장 모드:** Plan 모드 (`/plan` — 읽기 전용, 코드 변경 없음)

### 프로젝트 실무 예시

```
> 이 프로젝트의 전체 아키텍처를 설명해줘.
> 진입점, 주요 모듈간 의존 관계, 데이터 흐름을 정리해줘.
```

```
> @lib/shared/db_connector.py 이 모듈의 역할과
> 커넥션 풀링 방식을 분석해줘.
```

> **핵심 포인트:** Plan 모드에서는 AI가 **코드를 변경하지 않는다**
> 안전하게 코드를 분석하고 질문할 수 있다

---

# 실습 — 코드 이해

VSCode에서 Claude Code 패널을 열고 다음 프롬프트를 입력해 보세요:

### 실습 1: 전체 구조 분석

```
> @tutorial-examples/inventory.py 이 모듈의 전체 구조를 분석해줘.
> 데이터 구조, 함수 목록, 함수 간 호출 관계를 정리해줘.
```

### 실습 2: 에러 처리 패턴 분석

```
> @tutorial-examples/inventory.py 에서 에러 처리는 어떤 패턴으로
> 되어있어? ValueError가 발생하는 모든 경우를 정리해줘.
```

### 실습 3: 데이터 분석

```
> @tutorial-examples/inventory.py 의 SAMPLE_ITEMS 데이터를 보고
> 카테고리별 아이템 수와 총 재고 가치를 계산해줘.
```

---

# Stage 2: 코드 설계 (Design)

새 기능이나 변경사항의 구조를 **구현 전에** 설계하는 단계

**권장 모드:** Plan 모드 (`/plan`)

### 프로젝트 실무 예시

```
> 새로운 MES4 엔티티 "ToolMaster"를 동기화하는 매퍼를 추가하려고 해.
> 기존 PartMapper 패턴을 참고해서 설계안을 작성해줘.
> - 클래스 구조
> - MES4 테이블 → ERPNext DocType 매핑
> - 필요한 파일 목록
```

> **핵심 포인트:** 설계를 먼저 보고 승인한 뒤 구현으로 넘어가면
> 불필요한 재작업을 크게 줄일 수 있다

---

# 실습 — 코드 설계

### 실습 4: 입출고 이력 추적 기능 설계

```
> @tutorial-examples/inventory.py 를 분석하고,
> "입출고 이력 추적" 기능을 추가하는 설계안을 작성해줘.
> - 이력 데이터 구조 (딕셔너리 형태)
> - 필요한 함수 목록과 각 함수의 입출력
> - 기존 update_stock 함수와의 연동 방법
```

### 실습 5: 카테고리별 통계 기능 설계

```
> @tutorial-examples/inventory.py 에 카테고리별 통계 기능을 추가하려고 해.
> 카테고리별 아이템 수, 총 재고량, 총 가치를 보여주는 함수를 설계해줘.
```

> **Tip:** Plan 모드에서 설계를 마친 뒤, **대화를 유지한 채** Default 모드로 전환하면
> 설계 컨텍스트가 그대로 이어진다

---

# Stage 3: 코드 구현 (Implementation)

설계를 기반으로 실제 코드를 작성하는 단계

**권장 모드:** Default 모드 (diff 확인 → Accept/Reject)

### 프로젝트 실무 예시

```
> @lib/mes4_erpnext_sync/mappers/part_mapper.py 와 동일한 패턴으로
> ToolMasterMapper 클래스를 구현해줘.
```

### 모범 사례

| 원칙 | 이유 |
|------|------|
| 한 번에 하나의 모듈/기능만 요청 | AI의 응답 품질이 높아진다 |
| diff를 꼼꼼히 확인 후 승인 | 불필요한 변경을 방지한다 |
| `@` 참조로 기존 코드를 함께 전달 | 프로젝트 스타일을 일관성 있게 유지 |

---

# 실습 — 코드 구현

### 실습 6: 입출고 이력 기능 구현

```
> @tutorial-examples/inventory.py 에 입출고 이력 추적 기능을 구현해줘.
> - record_transaction(items, history, code, quantity, reason) 함수 추가
> - get_transaction_history(history, code) 함수 추가
> - history는 딕셔너리 리스트로 관리
> - 기존 update_stock을 호출하되, 이력도 함께 기록
```

### 실습 7: CSV 내보내기 구현

```
> @tutorial-examples/inventory.py 에 아이템을 CSV 파일로 내보내는
> export_to_csv(items, filename) 함수를 추가해줘.
```

> Claude가 코드를 수정하면 **diff 뷰**가 표시된다
> 각 변경을 확인하고 **Accept**(승인) 또는 **Reject**(거부)를 선택한다

---

# Stage 4: 코드 검증 (Verification)

구현된 코드의 품질과 정확성을 확인하는 단계

**권장 모드:** Default 모드

### 세 가지 검증 방법

| 방법 | 명령 / 프롬프트 | 확인하는 것 |
|------|----------------|-----------|
| **코드 리뷰** | `/review` | 논리 오류, 엣지 케이스, 스타일 |
| **테스트 실행** | `pytest -v` 요청 | 기능이 올바르게 동작하는지 |
| **테스트 작성** | "테스트를 작성해줘" | 아직 검증되지 않은 부분 |

### test_inventory.py 구성

```python
# 통과하는 테스트 3개 (이미 작성됨)
def test_add_item(sample_items): ...
def test_update_stock(sample_items): ...
def test_calculate_total_value(sample_items): ...

# TODO 테스트 4개 (직접 작성할 것!)
# def test_get_low_stock_items(sample_items): ...
# def test_update_stock_item_not_found(sample_items): ...
# def test_add_duplicate_item(sample_items): ...
# def test_update_stock_insufficient(sample_items): ...
```

---

# 실습 — 코드 검증

### 실습 8: 기존 테스트 실행

```
> @tutorial-examples/test_inventory.py 를 실행해줘.
> 결과를 분석하고 테스트 커버리지를 설명해줘.
```

### 실습 9: TODO 테스트 구현 (핵심 실습!)

```
> @tutorial-examples/test_inventory.py 의 TODO 주석으로 표시된
> 테스트 4개를 구현해줘. 기존 테스트 패턴(Given/When/Then)을 따라줘.
```

### 실습 10: 코드 리뷰

```
> @tutorial-examples/inventory.py 를 리뷰해줘.
> 에러 핸들링, 엣지 케이스, 개선할 점을 정리해줘.
```

> **Tip:** `.claude/settings.json`에 `"Bash(pytest *)"` 를 allow에 추가하면
> 테스트 실행 시 매번 승인하지 않아도 된다

---

# Stage 5: 코드 디버깅 (Debugging)

테스트 실패나 런타임 오류를 추적하고 수정하는 단계

**권장 모드:** Default 모드

### 디버깅 활용 기능

| 기능 | 사용법 | 비유 |
|------|--------|------|
| **에러 붙여넣기** | 에러 메시지를 프롬프트에 복사 | 의사에게 증상 설명 |
| **MCP 데이터 조회** | 실시간 DB 상태 확인 | 환자의 검사 결과 확인 |
| **Esc 되감기** | `Esc` 2회 → 이전 시점 복원 | 실수를 Ctrl+Z로 되돌리기 |

> **비유:** 에러 메시지는 "환자의 증상"이고, AI는 "의사" 역할을 한다
> 증상을 정확하게 전달할수록 진단이 빨라진다

---

# buggy_inventory.py — 버그 4개를 찾아라!

`buggy_inventory.py`에는 **의도적으로 4개의 버그**가 숨겨져 있다:

| Bug | 난이도 | 힌트 |
|-----|--------|------|
| #1 | ★☆☆☆ | `find_item` 함수의 비교 연산자 |
| #2 | ★★☆☆ | `update_stock` 함수의 검증 누락 |
| #3 | ★★☆☆ | `calculate_total_value` 함수의 연산자 |
| #4 | ★☆☆☆ | `get_low_stock_items` 함수의 비교 방향 |

> **도전:** 먼저 직접 코드를 읽으며 버그를 찾아보자!
> 찾은 뒤에 Claude에게 검증을 요청하면 **학습 효과가 더 크다**

---

# 실습 — 코드 디버깅

### 실습 11: 비교 분석으로 버그 찾기

```
> @tutorial-examples/buggy_inventory.py 와 @tutorial-examples/inventory.py 를
> 비교해서 차이점을 모두 찾아줘. 각 차이가 어떤 문제를 일으키는지 설명해줘.
```

### 실습 12: 실행 결과로 버그 추론하기

```
> @tutorial-examples/buggy_inventory.py 를 실행해줘.
> 출력 결과에서 이상한 점을 찾고, 원인이 되는 코드를 분석해줘.
```

### 실습 13: 버그 수정하기

```
> @tutorial-examples/buggy_inventory.py 의 버그 4개를 모두 수정해줘.
> 각 버그마다 수정 전/후 코드를 보여줘.
```

---

# 파이프라인 요약

| 단계 | 권장 모드 | 핵심 기능 | 대표 프롬프트 |
|------|----------|----------|-------------|
| **1. 이해** | Plan | `@` 참조, `/plan` | "이 모듈의 구조를 분석해줘" |
| **2. 설계** | Plan | 설계안 제시 | "기존 패턴을 참고해서 설계안을 작성해줘" |
| **3. 구현** | Default | diff Accept/Reject | "동일한 패턴으로 구현해줘" |
| **4. 검증** | Default | `/review`, pytest | "테스트를 작성하고 실행해줘" |
| **5. 디버깅** | Default | 에러 분석, MCP | "이 에러를 분석하고 수정해줘" |

```
이해 ──→ 설계 ──→ 구현 ──→ 검증 ──→ 디버깅
                                    │
                                    └──→ (문제 발견 시 이전 단계로)
```

> 이 파이프라인은 한 번에 끝나는 것이 아니라 **반복(iterate)** 된다

---

<!-- _class: lead -->
# 8. Tips & 다음 단계

문제 해결과 MEINT 연결

---

# 자주 묻는 문제 (Troubleshooting)

| 문제 | 해결 방법 |
|------|----------|
| `claude: command not found` | 새 터미널을 열거나 `source ~/.bashrc` 실행 |
| 브라우저 인증 실패 | `claude doctor` 실행 후 안내에 따라 조치 |
| VSCode 확장이 안 보임 | VSCode 1.98+ 확인, 확장 재설치 |
| MCP 서버 연결 실패 | `MCP_TIMEOUT` 환경변수 증가 (기본 25,000ms) |
| Windows npx MCP 오류 | `.mcp.json`에서 `cmd /c` 래퍼 사용 |
| 대화가 느려짐 | `/compact`로 압축 또는 `/clear`로 초기화 |

### 진단 도구

```bash
claude doctor    # 설치, 인증, 네트워크, MCP를 종합 진단
```

---

# 컨텍스트 관리 Tips

AI 대화가 길어지면 **컨텍스트 윈도우**(AI의 작업 기억)가 가득 찬다:

| 명령어 | 하는 일 | 언제 사용 |
|--------|---------|----------|
| `/compact` | 대화를 압축 (핵심만 유지) | 대화가 길어졌을 때 |
| `/clear` | 대화 완전 초기화 | 새로운 주제로 넘어갈 때 |
| `/context` | 컨텍스트 사용량 확인 | 남은 여유를 확인할 때 |

> **비유:** `/compact` = 책상 위 메모를 **요약 정리**하는 것
> `/clear` = 책상을 **완전히 치우고** 새로 시작하는 것

### 긴 디버깅 세션 Tip

`/compact` 후에는 에러 메시지와 스택 트레이스를 **다시 붙여넣어** 줘야 한다
핵심 정보가 압축 과정에서 유실될 수 있기 때문

---

# MEINT 프로젝트와의 연결

이 튜토리얼에서 배운 기술이 MEINT에서 **어떻게 활용되는지:**

| 이 튜토리얼에서 배운 것 | MEINT에서 사용하는 곳 |
|----------------------|---------------------|
| `@` 파일 참조 | 동기화 코드 분석 시 매퍼 파일 참조 |
| Plan 모드 | 새 동기화 엔티티 설계 |
| diff Accept/Reject | AI가 생성한 대시보드 코드 검토 |
| `/review` | 동기화 로직 코드 리뷰 |
| pytest 실행 | 동기화/매퍼 단위 테스트 |
| 에러 붙여넣기 | MES4 DB 연결 오류 디버깅 |
| MCP 서버 | 실시간 공장 DB 조회, 동기화 명령 실행 |

> 이 튜토리얼은 **"도구 사용법"**이고,
> MEINT 실습은 그 도구로 **"실제 문제를 해결하는 것"**이다

---

# 다음 단계: MEINT 학습 경로

```mermaid
graph LR
    A["1. 환경 구축"]
    B["2. AI 도구 배우기<br/>(이 튜토리얼)"]
    C["3. 데모 데이터 설치"]
    D["4. 제조 업무 흐름"]
    E["5. 시스템 통합"]
    F["6. 대시보드 만들기"]
    G["7. 나만의 프로젝트"]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G

    style B fill:#1a5f7a,color:#fff,stroke:#1a5f7a
```

여러분은 지금 **2단계를 완료**했습니다!

다음은 `/erpnext-demo:setup-demo` 명령으로 **데모 데이터를 설치**하고
실제 제조 업무 흐름을 체험합니다.

---

<!-- _class: lead -->
# Thank You

## AI Coding Tutorial
### Claude Code로 시작하는 AI 코딩

MEINT 프로젝트 사전 준비 완료!


