# Ubuntu 공유 계정에서 Claude Code 개인 인증 분리 사용법

## 개요

하나의 Ubuntu 계정(예: `meint0x`)을 여러 팀원이 공유하면서, Claude Code는 각자의 Anthropic 계정으로 인증하는 구성 가이드이다. `CLAUDE_CONFIG_DIR` 환경변수를 활용하여 팀원별 설정 디렉터리를 분리하고, SSH 공개키 기반으로 접속 시 자동 적용하는 방식을 제공한다.

## 목차

- [문제: 인증 충돌](#문제-인증-충돌)
- [해결: 환경변수로 Claude 설정 디렉터리 분리](#해결-환경변수로-claude-설정-디렉터리-분리)
  - [1단계: 팀원별 디렉터리 생성](#1단계-팀원별-디렉터리-생성)
  - [2단계: 팀원별 쉘 프로파일 스크립트 작성](#2단계-팀원별-쉘-프로파일-스크립트-작성)
  - [3단계: SSH 접속 시 자동 적용 (authorized_keys 활용)](#3단계-ssh-접속-시-자동-적용-authorized_keys-활용)
  - [4단계: 각 팀원 최초 로그인](#4단계-각-팀원-최초-로그인)
  - [분리 범위: 무엇이 격리되고, 무엇이 공유되는가](#분리-범위-무엇이-격리되고-무엇이-공유되는가)
- [보안 고려사항](#보안-고려사항)
- [부록 A: cat + heredoc 구문 설명](#부록-a-cat--heredoc-구문-설명)


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

### 3단계: SSH 접속 시 자동 적용 (authorized_keys 활용)

> **이 방법은 SSH 공개키 인증에서만 동작한다.** `authorized_keys`의 `environment=` 옵션은 공개키 매칭 시 적용되는 메커니즘이므로, 아이디/비밀번호 인증으로는 사용할 수 없다.

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

이 설정만으로 `CLAUDE_CONFIG_DIR`이 자동 설정되므로, Claude Code는 접속한 팀원에 맞는 설정 디렉터리를 자동으로 사용한다.

### 4단계: 각 팀원 최초 로그인

SSH 공개키로 서버에 접속하면 `CLAUDE_CONFIG_DIR`이 자동 설정된다. 최초 실행 시 로그인 프롬프트가 나타나며, 각자의 Anthropic 계정으로 인증한다.

```bash
ssh meint0x@server       # authorized_keys에 의해 CLAUDE_CONFIG_DIR 자동 설정
claude                   # 최초 실행 시 로그인 프롬프트 -> 자기 계정으로 인증
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

