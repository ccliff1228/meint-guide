# MEINT Guide

생성형 AI 도구를 활용한 제조 AX(AI Transformation) 프로젝트 — **MEINT(MES-ERP Integration)** 의 학습 가이드 문서 모음이다.

## 문서 목록

- **01 ERP와 MES 개요** — ERP와 MES의 핵심 차이, 역할, 분리 운영의 문제점과 통합 방향
- **02 프로젝트 시나리오** — ERP-MES 데이터 단절 문제를 AI 기반 방법론(AX)으로 해소하는 MEINT 시나리오
- **03 프로젝트 오버뷰** — MEINT의 전체 구조, 핵심 구성 요소, AI 기반 개발 방법론 상세 강의 노트
- **04 시스템 아키텍처** — MES4-ERPNext를 연결하는 6계층 스택 아키텍처 (AI 개발 계층 포함)
- **05 AI 코딩 튜토리얼** — Claude Code 설치, 설정, MEINT 프로젝트 사전 준비 단계
- **06 VSCode 설치 및 사용법** — Visual Studio Code 설치, 확장 프로그램, 기본 사용법
- **07 Claude 선물 사용 가이드** — Claude 선물 구독 활성화(Redeem) 절차와 Claude Code 설정
- **08 Claude Code 설치 및 사용법** — Claude Code CLI 설치(Node.js, Git for Windows 포함)와 기본 사용법
- **09 SSH 공개키 로그인 설정** — ed25519 키 생성부터 서버 등록, 비밀번호 로그인 비활성화까지 (PowerShell/Linux 구분)
- **10 Claude Code 공유 계정 설정** — Ubuntu 공유 계정에서 팀원별 Claude Code 인증 분리 (`CLAUDE_CONFIG_DIR`)
- **11 GitHub Private Repo 가이드** — 팀 협업을 위한 Collaborator 초대, 브랜치 보호 규칙, PAT 생성, 작업 흐름

## 추천 학습 순서

```
01 ERP와 MES 개요
 └─► 02 프로젝트 시나리오
      └─► 03 프로젝트 오버뷰
           └─► 04 시스템 아키텍처
                └─► 05 AI 코딩 튜토리얼
```

06~11은 환경 설정 시 필요할 때 참고한다.

## MEINT 프로젝트 개요

- **MES4** (Festo) — 공장 현장 장비 제어 및 생산 공정 데이터
- **ERPNext** v17 — 구매, 판매, 재고, 제조, 회계 통합 관리
- **Claude Code** + MCP Server — AI 기반 개발 환경 (61개 슬래시 명령)
- **React Dashboard** — 제조 현황 실시간 모니터링

## 관련 링크

- [ERPNext 공식 문서](https://docs.erpnext.com)
- [Frappe Framework 공식 문서](https://frappeframework.com/docs)
- [Claude Code 공식 문서](https://docs.anthropic.com/en/docs/claude-code)
