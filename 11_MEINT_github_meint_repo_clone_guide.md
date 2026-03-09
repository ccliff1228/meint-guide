# GitHub Private Repo 팀 협업 가이드

## 개요

GitHub Private Repository에 팀원을 초대하고, 브랜치 보호 규칙으로 코드 품질을 관리하는 방법을 정리한다. 소유자(Owner)와 팀원(Collaborator)의 역할별로 구분하여 안내한다.

## 목차

- [Part 1. 소유자 (Owner)](#part-1-소유자-owner)
  - [1-1. 팀원 초대](#1-1-팀원-초대)
  - [1-2. 브랜치 보호 규칙 설정](#1-2-브랜치-보호-규칙-설정)
  - [1-3. Bypass List 추가 (소유자 우회 설정)](#1-3-bypass-list-추가-소유자-우회-설정)
- [Part 2. 팀원 (Collaborator)](#part-2-팀원-collaborator)
  - [2-1. 사전 준비](#2-1-사전-준비)
  - [2-2. Personal Access Token(PAT) 생성](#2-2-personal-access-tokenpat-생성)
  - [2-3. Repository Clone](#2-3-repository-clone)
  - [2-4. 작업 흐름](#2-4-작업-흐름)
- [참고](#참고)

---

## Part 1. 소유자 (Owner)

### 1-1. 팀원 초대

1. GitHub에서 해당 repo → **Settings** → **Collaborators** → **Add people**
2. 팀원의 GitHub 사용자명 또는 이메일 입력 후 초대
3. 팀원이 이메일로 받은 초대를 Accept하면 완료

> Collaborator는 push/pull, 브랜치 생성/삭제, Issues/PR 관리 권한을 갖는다. 저장소 삭제나 Collaborator 추가/제거는 소유자만 가능하다.

### 1-2. 브랜치 보호 규칙 설정

main 브랜치에 직접 push를 차단하고, PR을 통한 merge만 허용한다.

**설정 경로:**

1. GitHub repo → **Settings** → **Branches**
2. **Add branch protection rule** 클릭

**권장 설정 항목:**

- **Branch name pattern** — `main` 입력
- **Require a pull request before merging** — 체크
  - **Require approvals** — 체크 (최소 1명 승인 필수)
- **Require status checks to pass before merging** — CI/CD가 있으면 체크
- **Do not allow bypassing the above settings** — 소유자 포함 전원 적용 시 체크

설정 완료 후 **Create** 클릭

**적용 결과:**

- 모든 팀원(소유자 포함)이 main에 직접 push 불가
- 작업 브랜치에서 개발 → PR 생성 → 리뷰 승인 → merge 흐름이 강제됨

### 1-3. Bypass List 추가 (소유자 우회 설정)

소유자는 PR 없이 main에 직접 push할 수 있도록 예외를 설정한다.

1. GitHub repo → **Settings** → **Branches**
2. 기존 보호 규칙의 **Edit** 클릭
3. **Allow specified actors to bypass required pull requests** 항목 찾기
4. 소유자 본인의 GitHub 사용자명 추가
5. **Save changes** 클릭

**적용 결과:**

- **소유자** — main에 직접 push 가능, PR 생략 가능
- **팀원** — 기존대로 PR 생성 + 승인 필수

---

## Part 2. 팀원 (Collaborator)

### 2-1. 사전 준비

1. GitHub 계정 생성 (https://github.com)
2. 소유자로부터 초대 이메일 수신
3. 이메일에서 **Accept invitation** 클릭

### 2-2. Personal Access Token(PAT) 생성

GitHub는 비밀번호 직접 인증을 지원하지 않으므로 PAT이 필요하다.

1. GitHub → 우측 상단 프로필 → **Settings**
2. 좌측 하단 **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. **Generate new token** → **repo** 권한 체크 → 토큰 생성
4. 토큰 복사 (한 번만 표시되므로 즉시 저장)

### 2-3. Repository Clone

```bash
git clone https://github.com/ccliff1228/meint.git
```

- 비밀번호 자리에 위에서 생성한 **PAT** 입력
- Windows에서 매번 입력이 번거로우면 자격 증명 저장:

```bash
git config --global credential.helper manager
```

### 2-4. 작업 흐름

브랜치 보호 규칙이 적용된 후 팀원의 표준 작업 흐름:

```bash
# 1. 최신 코드 가져오기
git pull origin main

# 2. 작업 브랜치 생성
git checkout -b feature/기능이름

# 3. 작업 후 커밋
git add .
git commit -m "작업 내용 설명"

# 4. 작업 브랜치 push
git push origin feature/기능이름

# 5. GitHub에서 Pull Request 생성
#    base: main ← compare: feature/기능이름

# 6. 소유자 리뷰 승인 후 Merge
```

---

## 참고

- [GitHub Collaborator 권한 문서](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-access-to-your-personal-repositories/inviting-collaborators-to-a-personal-repository)
- [Branch Protection Rules 문서](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-a-branch-protection-rule/about-branch-protection-rules)
