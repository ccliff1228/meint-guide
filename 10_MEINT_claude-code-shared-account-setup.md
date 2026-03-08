# Ubuntu 공유 계정에서 Claude Code 개인 인증 분리 사용법

## 개요

하나의 Ubuntu 계정(예: `meint0x`)을 여러 팀원이 공유하면서, Claude Code는 각자의 Anthropic 계정으로 인증하는 구성 가이드이다.

## 문제: 인증 충돌

```
Ubuntu Account: meint0x (공유)
├── ~/.claude/              ← Claude Code 설정/인증 저장 위치
│   ├── credentials.json    ← 인증 토큰 (충돌 지점!)
│   ├── settings.json       ← 개인 설정
│   └── projects/           ← 프로젝트별 메모리
└── ~/Projects/             ← 공유 작업 디렉터리
```

`~/.claude/`가 하나이므로, 그냥 쓰면 마지막으로 로그인한 사람의 인증이 덮어써진다.

---

## 해결: 환경변수로 Claude 설정 디렉터리 분리

`CLAUDE_CONFIG_DIR` 환경변수로 팀원별 설정 디렉터리를 분리한다. 대부분의 경우 이 방법만으로 충분하다.

### 1단계: 팀원별 디렉터리 생성

```bash
mkdir -p ~/.claude-profiles/alice
mkdir -p ~/.claude-profiles/bob
mkdir -p ~/.claude-profiles/charlie

>> alice, bob, charlie 는 팀원 이름으로 변경한다. 이 디렉터리들은 각 팀원의 개인 설정과 인증 정보를 저장하는 루트가 된다.
```

### 2단계: 팀원별 쉘 프로파일 스크립트 작성

```bash
cat > ~/.claude-profiles/alice/env.sh << 'EOF'
export CLAUDE_CONFIG_DIR="$HOME/.claude-profiles/alice/.claude"
export PS1="(alice) $PS1"
echo "Claude Code profile: alice"
EOF
```

각 팀원에 대해 동일하게 생성한다. 이 구문의 상세 설명은 [부록 A: cat + heredoc 구문 설명](#부록-a-cat--heredoc-구문-설명)을 참고한다.

### 3단계: 접속 시 프로파일 선택

#### 방법 A: SSH 접속 시 자동 적용 (authorized_keys 활용)

> **이 방법은 SSH 공개키 인증에서만 동작한다.** `authorized_keys`의 `environment=` 옵션은 공개키 매칭 시 적용되는 메커니즘이므로, 아이디/비밀번호 인증으로는 사용할 수 없다. 비밀번호 인증은 공유 계정에서 모든 팀원이 동일한 username + password를 사용하므로, 서버가 누가 접속했는지 구분할 수 없기 때문이다. 비밀번호 인증 환경에서는 아래 방법 B(수동 선택)를 사용한다.

각 팀원의 SSH 공개키에 환경변수를 지정한다.

```bash
# ~/.ssh/authorized_keys
environment="CLAUDE_CONFIG_DIR=/home/meint0x/.claude-profiles/alice/.claude" ssh-ed25519 AAAA... alice@laptop
environment="CLAUDE_CONFIG_DIR=/home/meint0x/.claude-profiles/bob/.claude" ssh-ed25519 AAAA... bob@laptop
```

각 줄은 세 부분으로 구성된다. 공개키는 팀원마다 다르며, 같은 `meint0x` 계정으로 접속하더라도 **누구의 키로 접속했느냐에 따라 다른 환경변수가 설정**된다.

| 부분 | 예시 | 역할 |
|------|------|------|
| 옵션 | `environment="CLAUDE_CONFIG_DIR=..."` | 이 키로 접속 시 자동 설정할 환경변수 |
| 공개키 | `ssh-ed25519 AAAA...` | 팀원 본인 노트북에서 생성한 고유 키 |
| 코멘트 | `alice@laptop` | 누구의 키인지 식별용 (동작에 영향 없음) |

각 팀원은 자기 노트북에서 `ssh-keygen`으로 키 쌍을 생성하고, 공개키만 서버의 `authorized_keys`에 등록한다. 서버는 접속 시 매칭되는 공개키를 찾아 해당 줄의 `environment=` 옵션을 자동 실행한다.

```
alice notebook (private key: alice's ssh-ed25519)
    |  ssh meint0x@server
    v
Server: find matching public key in authorized_keys
    |  matched -> line 1
    v
CLAUDE_CONFIG_DIR=.../alice/.claude  (auto-set)
```

`/etc/ssh/sshd_config`에서 `PermitUserEnvironment` 활성화가 필요하다:

```bash
sudo sed -i 's/^#*PermitUserEnvironment.*/PermitUserEnvironment yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

방법 A만으로 `CLAUDE_CONFIG_DIR`이 자동 설정되므로, 아래 방법 B의 `claude-profile()` 함수 없이도 Claude Code는 올바른 설정 디렉터리를 사용한다. 방법 B는 SSH 자동 적용이 안 되는 상황(로컬 접속, 수동 전환 등)을 위한 폴백이다.

#### 방법 B: 접속 후 수동 선택 (선택사항)

`~/.bashrc` 맨 아래에 다음 스크립트를 추가한다. 이 스크립트의 상세 동작은 [부록 B: claude-profile 스크립트 상세 설명](#부록-b-claude-profile-스크립트-상세-설명)을 참고한다.

```bash
# === Claude Code 팀 프로파일 시스템 ===
CLAUDE_PROFILES_DIR="$HOME/.claude-profiles"

claude-profile() {
    local name="$1"
    if [ -z "$name" ]; then
        echo "=== Claude Code 프로파일 ==="
        echo "현재: ${CLAUDE_PROFILE:-미설정}"
        echo "사용 가능:"
        ls -1 "$CLAUDE_PROFILES_DIR" 2>/dev/null | sed 's/^/  - /'
        echo ""
        echo "사용법: claude-profile <이름>"
        return 0
    fi

    mkdir -p "$CLAUDE_PROFILES_DIR/$name/.claude"

    export CLAUDE_CONFIG_DIR="$CLAUDE_PROFILES_DIR/$name/.claude"
    export CLAUDE_PROFILE="$name"

    # 프롬프트에 프로파일 표시
    export PS1="(\$CLAUDE_PROFILE) ${BASE_PS1:-$PS1}"

    echo "Claude 프로파일 활성화: $name"
}

# 기본 PS1 저장
BASE_PS1="$PS1"

# SSH 키 기반 자동 감지 (authorized_keys에서 CLAUDE_CONFIG_DIR 설정 시)
if [ -n "$CLAUDE_CONFIG_DIR" ]; then
    CLAUDE_PROFILE=$(basename $(dirname "$CLAUDE_CONFIG_DIR"))
    echo "자동 감지된 Claude 프로파일: $CLAUDE_PROFILE"
fi
```

사용:

```bash
ssh meint0x@server
claude-profile alice    # 내 프로파일 활성화
claude                  # Claude Code 실행 -> alice의 인증 사용
```

> **주의:** 프로필은 반드시 **Claude Code를 시작하기 전에** 적용해야 한다. Claude Code가 이미 실행 중인 상태에서는 환경변수를 변경해도 반영되지 않는다. `env.sh`를 직접 사용하는 경우도 동일하다:
>
> ```bash
> source ~/.claude-profiles/alice/env.sh
> claude
> ```
>
> 또는 한 줄로:
>
> ```bash
> CLAUDE_CONFIG_DIR="$HOME/.claude-profiles/alice/.claude" claude
> ```

### 4단계: 각 팀원 최초 로그인

```bash
claude-profile alice
claude   # 최초 실행 시 로그인 프롬프트 -> 자기 계정으로 인증
```

### 분리 범위: 무엇이 격리되고, 무엇이 공유되는가

`CLAUDE_CONFIG_DIR`을 변경하면 **사용자 수준(User-level)** 파일만 격리된다. **프로젝트 수준(Project-level)** 파일은 프로젝트 디렉터리에 고정되어 프로필과 무관하게 공유된다.

**프로필별로 격리되는 파일 (사용자 수준):**

| 파일 | 경로 | 용도 |
|------|------|------|
| settings.json | `$CLAUDE_CONFIG_DIR/settings.json` | 사용자 개인 설정 |
| CLAUDE.md | `$CLAUDE_CONFIG_DIR/CLAUDE.md` | 사용자 개인 지시사항 |
| rules/ | `$CLAUDE_CONFIG_DIR/rules/` | 사용자 개인 규칙 |
| keybindings.json | `$CLAUDE_CONFIG_DIR/keybindings.json` | 키보드 단축키 |
| auto-memory | `$CLAUDE_CONFIG_DIR/projects/<project>/memory/` | 프로젝트별 자동 메모리 |
| 인증 정보 | `$CLAUDE_CONFIG_DIR/` 내부 | 로그인 토큰 |

**프로필과 무관하게 공유되는 파일 (프로젝트 수준):**

| 파일 | 경로 | 용도 |
|------|------|------|
| CLAUDE.md (프로젝트) | `<project>/CLAUDE.md` | 팀 공유 프로젝트 지시사항 |
| settings.json (프로젝트) | `<project>/.claude/settings.json` | 팀 공유 프로젝트 설정 |
| settings.local.json | `<project>/.claude/settings.local.json` | 로컬 프로젝트 설정 (gitignore) |
| rules/ (프로젝트) | `<project>/.claude/rules/` | 팀 공유 프로젝트 규칙 |

```
~/.claude-profiles/
├── alice/.claude/           ← alice 전용 (격리)
│   ├── settings.json
│   ├── CLAUDE.md
│   └── projects/
│       └── <project>/memory/
└── bob/.claude/             ← bob 전용 (격리)
    ├── settings.json
    ├── CLAUDE.md
    └── projects/
        └── <project>/memory/

~/Projects/myapp/            ← 프로젝트 (공유)
├── CLAUDE.md                ← 모든 프로필이 공유
└── .claude/
    ├── settings.json        ← 모든 프로필이 공유
    └── rules/               ← 모든 프로필이 공유
```

즉, 각 팀원은 **자신만의 인증, 설정, 메모리**를 갖되, **프로젝트의 공통 규칙과 설정은 함께 사용**하는 구조이다.

---

## 보안 고려사항

```bash
# 프로파일 디렉터리 권한 설정 (공유 계정이므로 완전한 격리는 불가)
chmod 700 ~/.claude-profiles/alice
chmod 700 ~/.claude-profiles/bob
```

동일 Ubuntu 계정을 공유하므로 `root` 권한이나 같은 UID를 가진 사용자는 모든 프로파일에 접근 가능하다. 민감한 환경에서는 인증 토큰이 포함된 `.claude/` 디렉터리의 권한 관리에 주의해야 한다.

```bash
# 프로젝트 .gitignore에 추가
.claude-profiles/
```

> `CLAUDE_CONFIG_DIR` 환경변수의 정확한 동작은 Claude Code 버전에 따라 다를 수 있다. 최초 설정 후 `claude --version`과 설정 경로를 확인할 것.

---

## 부록 A: cat + heredoc 구문 설명

```bash
cat > ~/.claude-profiles/alice/env.sh << 'EOF'
export CLAUDE_CONFIG_DIR="$HOME/.claude-profiles/alice/.claude"
export PS1="(alice) $PS1"
echo "Claude Code profile: alice"
EOF
```

| 구문 | 의미 |
|------|------|
| `cat >` | cat의 출력을 파일로 리다이렉션 (파일이 있으면 덮어씀) |
| `~/.claude-profiles/alice/env.sh` | 생성할 파일 경로 |
| `<< 'EOF'` | 여기서부터 `EOF`가 나올 때까지가 파일 내용 (heredoc) |

**`'EOF'` vs `EOF` 차이:**

- `'EOF'` (따옴표 있음): 내용 안의 `$변수`를 **그대로 문자열로** 저장
- `EOF` (따옴표 없음): 내용 안의 `$변수`를 **현재 값으로 치환**해서 저장

여기서는 `'EOF'`를 사용했으므로 `$HOME`, `$PS1`이 치환되지 않고 문자 그대로 파일에 기록된다.

**생성되는 파일의 각 줄 역할:**

| 줄 | 역할 |
|----|------|
| `export CLAUDE_CONFIG_DIR=...` | Claude Code 설정 경로를 해당 팀원 전용 디렉터리로 변경 |
| `export PS1="(alice) $PS1"` | 쉘 프롬프트 앞에 `(alice)` 표시를 추가하여 현재 프로파일 식별 |
| `echo "Claude Code profile: alice"` | 프로파일 로드 시 확인 메시지 출력 |

이 파일은 `source ~/.claude-profiles/alice/env.sh`로 실행하여 환경변수를 현재 쉘에 적용한다.

---

## 부록 B: claude-profile 스크립트 상세 설명

### 프로파일 기본 경로

```bash
CLAUDE_PROFILES_DIR="$HOME/.claude-profiles"
```

모든 팀원의 프로파일이 저장되는 루트 디렉터리를 변수로 지정한다.

### `claude-profile` 함수

**인자 없이 호출 시 — 상태 조회:**

```bash
local name="$1"
if [ -z "$name" ]; then
```

- `$1`: 첫 번째 인자 (팀원 이름)
- `-z "$name"`: 인자가 비어있으면 참

```bash
echo "현재: ${CLAUDE_PROFILE:-미설정}"
```

- `${변수:-기본값}`: `CLAUDE_PROFILE`이 미설정이면 `"미설정"` 출력

```bash
ls -1 "$CLAUDE_PROFILES_DIR" 2>/dev/null | sed 's/^/  - /'
```

- `ls -1`: 한 줄에 하나씩 디렉터리 목록 출력
- `2>/dev/null`: 디렉터리가 없을 때 에러 숨김
- `sed 's/^/  - /'`: 각 줄 앞에 `  - ` 접두어 추가 (목록 형태)

**인자 있을 때 — 프로파일 활성화:**

```bash
mkdir -p "$CLAUDE_PROFILES_DIR/$name/.claude"
```

프로파일 디렉터리가 없으면 자동 생성한다. `-p`는 중간 경로까지 한번에 생성.

```bash
export CLAUDE_CONFIG_DIR="$CLAUDE_PROFILES_DIR/$name/.claude"
export CLAUDE_PROFILE="$name"
```

- `CLAUDE_CONFIG_DIR`: Claude Code가 인증/설정을 읽는 경로를 해당 팀원 전용으로 변경
- `CLAUDE_PROFILE`: 현재 활성 프로파일 이름 저장 (프롬프트 표시용)

```bash
export PS1="(\$CLAUDE_PROFILE) ${BASE_PS1:-$PS1}"
```

- `\$CLAUDE_PROFILE`: 백슬래시 이스케이프로 변수 평가를 **프롬프트 표시 시점까지 지연** (프로파일 변경 시 자동 반영)
- `${BASE_PS1:-$PS1}`: 원본 프롬프트가 저장되어 있으면 사용, 없으면 현재 PS1 사용

### 원본 프롬프트 저장

```bash
BASE_PS1="$PS1"
```

`.bashrc` 로드 시점의 원래 프롬프트를 저장한다. `claude-profile`을 여러 번 호출해도 프롬프트가 `(alice)(alice)(alice)...`처럼 중첩되는 것을 방지한다.

### CLAUDE_CONFIG_DIR 자동 감지

```bash
if [ -n "$CLAUDE_CONFIG_DIR" ]; then
    CLAUDE_PROFILE=$(basename $(dirname "$CLAUDE_CONFIG_DIR"))
    echo "자동 감지된 Claude 프로파일: $CLAUDE_PROFILE"
fi
```

- `-n "$CLAUDE_CONFIG_DIR"`: 변수가 비어있지 않으면 참
- `.bashrc`가 로드되는 시점에 `CLAUDE_CONFIG_DIR`이 이미 설정되어 있으면 프로파일 이름을 자동 추출한다. 주로 방법 A의 `authorized_keys`의 `environment=` 옵션에 의해 설정되지만, 수동으로 `export`한 경우에도 동일하게 동작한다.
- `dirname` + `basename`으로 경로에서 프로파일 이름을 추출:

```
CLAUDE_CONFIG_DIR=/home/meint0x/.claude-profiles/alice/.claude
                                                 | dirname
                  /home/meint0x/.claude-profiles/alice
                                                 | basename
                                                 alice
```
