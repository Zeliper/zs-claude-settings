# MCP (Model Context Protocol) 개요

## MCP란 무엇인가?

**MCP (Model Context Protocol)**는 AI 애플리케이션을 외부 시스템에 연결하기 위한 오픈 소스 표준입니다. 마치 **AI를 위한 USB-C 포트**와 같이, 다양한 도구와 서비스에 표준화된 방식으로 연결할 수 있습니다.

### MCP가 연결할 수 있는 것들

| 유형 | 예시 |
|------|------|
| **데이터 소스** | 로컬 파일, 데이터베이스, API |
| **도구** | 검색 엔진, 계산기, 외부 서비스 |
| **워크플로우** | 특화된 프롬프트, 절차 |

## MCP로 할 수 있는 일

### 1. 이슈 트래커 연동

```
"JIRA 이슈 ENG-4521에 설명된 기능을 구현하고 PR을 만들어줘"
```

### 2. 모니터링 데이터 분석

```
"Sentry와 Statsig에서 ENG-4521 기능의 사용 현황을 확인해줘"
```

### 3. 데이터베이스 쿼리

```
"PostgreSQL에서 ENG-4521 기능을 사용한 10명의 사용자 이메일을 찾아줘"
```

### 4. 디자인 연동

```
"Slack에서 공유된 새 Figma 디자인을 기반으로 이메일 템플릿을 업데이트해줘"
```

### 5. 워크플로우 자동화

```
"이 10명의 사용자를 피드백 세션에 초대하는 Gmail 초안을 만들어줘"
```

## MCP 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                      Claude Code                            │
│                    (MCP 클라이언트)                          │
└─────────────────────────────┬───────────────────────────────┘
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
           ▼                  ▼                  ▼
    ┌────────────┐     ┌────────────┐     ┌────────────┐
    │ MCP Server │     │ MCP Server │     │ MCP Server │
    │  (GitHub)  │     │  (Sentry)  │     │  (PostgreSQL)│
    └─────┬──────┘     └─────┬──────┘     └─────┬──────┘
          │                  │                  │
          ▼                  ▼                  ▼
    ┌────────────┐     ┌────────────┐     ┌────────────┐
    │ GitHub API │     │ Sentry API │     │ PostgreSQL │
    └────────────┘     └────────────┘     └────────────┘
```

## Transport 유형

MCP 서버는 세 가지 연결 방식을 지원합니다:

### 1. HTTP (권장)

원격 클라우드 기반 서비스용:

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

**특징**:
- 원격 서비스에 적합
- OAuth 인증 지원
- 가장 일반적인 방식

### 2. SSE (Server-Sent Events)

스트리밍 연결용 (레거시):

```bash
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

**특징**:
- 실시간 스트리밍
- HTTP보다 덜 권장됨

### 3. Stdio (Standard I/O)

로컬 프로세스 및 스크립트용:

```bash
claude mcp add --transport stdio db \
  --env DB_URL="postgresql://user:pass@localhost/db" \
  -- npx -y @some/mcp-server
```

**특징**:
- 로컬 실행
- 사용자 정의 서버에 적합
- 환경 변수 지원

## MCP 서버 스코프

### 1. Local Scope (기본값)

```bash
# ~/.claude.json에 저장
claude mcp add my-server ...
```

- 현재 프로젝트에서 본인만 사용
- 민감한 자격 증명에 적합
- 개인 개발 서버용

### 2. Project Scope

```bash
# .mcp.json에 저장 (프로젝트 루트)
claude mcp add --scope project team-server ...
```

- 버전 관리에 포함
- 팀 전체 공유
- 사용 전 승인 필요

### 3. User Scope

```bash
# ~/.claude.json에 저장
claude mcp add --scope user global-server ...
```

- 모든 프로젝트에서 사용
- 개인 계정에 비공개
- 자주 사용하는 서비스용

### 스코프 우선순위

```
1. Local-scoped 서버 (최우선)
2. Project-scoped 서버 (.mcp.json)
3. User-scoped 서버
4. Enterprise managed 서버 (최하위)
```

## MCP 관리 명령어

### 서버 추가

```bash
# HTTP 서버
claude mcp add --transport http github https://api.github.com/mcp/

# Stdio 서버
claude mcp add --transport stdio db \
  --env DATABASE_URL="..." \
  -- npx -y @db/mcp-server

# 헤더와 함께
claude mcp add --transport http api https://api.example.com/mcp \
  --header "Authorization: Bearer token"
```

### 서버 관리

```bash
# 목록 보기
claude mcp list

# 상세 정보
claude mcp get github

# 제거
claude mcp remove github

# 프로젝트 승인 초기화
claude mcp reset-project-choices
```

### Claude Code 내부에서

```
# MCP 상태 확인
> /mcp

# 리소스 참조
> @github:issue://123

# MCP 프롬프트 실행
> /mcp__github__list_prs
```

## MCP 구성 요소

### 1. Resources (리소스)

서버가 노출하는 데이터:

```
@github:issue://123
@postgres:schema://users
@docs:file://README.md
```

### 2. Tools (도구)

Claude가 호출할 수 있는 함수:

```
list_issues()
query_database()
send_email()
```

### 3. Prompts (프롬프트)

미리 설정된 워크플로우:

```
/mcp__github__pr_review 456
/mcp__jira__create_issue "제목" high
```

## 인증

### OAuth 인증 플로우

```bash
# 1. 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. Claude Code에서 인증
> /mcp
# "Authenticate" 선택
# 브라우저에서 로그인

# 3. 사용
> "최근 24시간 에러 보여줘"
```

### 인증 관리

```
> /mcp
# 서비스 선택
# "Clear authentication" 선택  ← 인증 삭제
# "Authenticate" 선택          ← 재인증
```

## Windows 특이사항

Windows(WSL 아님)에서 Stdio MCP 서버 사용 시:

```bash
# 올바른 방법 (cmd /c 래퍼 필요)
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package

# 잘못된 방법 (Connection closed 에러 발생)
claude mcp add --transport stdio my-server -- npx -y @some/package
```

## MCP vs 다른 기능

| 기능 | 용도 | 연결 대상 |
|------|------|----------|
| **MCP** | 외부 시스템 연결 | API, DB, 클라우드 서비스 |
| **Skills** | 지식 주입 | 내부 문서, 패턴 |
| **Agents** | 작업 위임 | 내부 Sub-Agent |
| **Hooks** | 이벤트 트리거 | 로컬 스크립트 |

## 다음 단계

1. [MCP 서버 제작](02-creating-servers.md) - 커스텀 MCP 서버 구현
2. [MCP 설정](03-configuration.md) - 상세 설정 방법
3. [MCP 예제](04-examples.md) - 실전 사용 예제
