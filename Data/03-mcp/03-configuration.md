# MCP 설정 가이드

## 기본 설정

### 서버 추가

#### HTTP 서버

```bash
# 기본 추가
claude mcp add --transport http github https://api.github.com/mcp/

# 인증 헤더 포함
claude mcp add --transport http api https://api.example.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"

# 여러 헤더
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer TOKEN" \
  --header "X-Custom-Header: value"
```

#### Stdio 서버

```bash
# npm 패키지
claude mcp add --transport stdio playwright \
  -- npx -y @playwright/mcp@latest

# 환경 변수 포함
claude mcp add --transport stdio db \
  --env DATABASE_URL="postgresql://user:pass@localhost/mydb" \
  -- npx -y @db/mcp-server

# 로컬 스크립트
claude mcp add --transport stdio local-server \
  -- node /path/to/my-server/index.js

# Python 서버
claude mcp add --transport stdio python-server \
  -- python /path/to/server.py
```

#### Windows 특수 처리

```bash
# Windows에서는 cmd /c 래퍼 필요
claude mcp add --transport stdio my-server \
  -- cmd /c npx -y @some/package

# PowerShell 사용
claude mcp add --transport stdio ps-server \
  -- powershell -Command "node server.js"
```

## 스코프 설정

### Local Scope (기본값)

```bash
# ~/.claude.json에 저장
claude mcp add my-server ...
```

**사용 사례**:
- 개인 개발 서버
- 민감한 자격 증명
- 프로젝트 특화 설정

### Project Scope

```bash
# .mcp.json에 저장
claude mcp add --scope project team-server ...
```

**.mcp.json 예시**:
```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.github.com/mcp/"
    },
    "internal-api": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@company/mcp-server"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**특징**:
- 버전 관리 포함
- 팀 공유
- 사용 전 승인 필요

### User Scope

```bash
# ~/.claude.json에 저장 (cross-project)
claude mcp add --scope user global-util ...
```

**사용 사례**:
- 여러 프로젝트에서 공통 사용
- 개인 유틸리티 서버
- 자주 사용하는 서비스

## 환경 변수

### 설정 파일에서 환경 변수 사용

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    },
    "local-db": {
      "type": "stdio",
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_URL": "${DB_CONNECTION_STRING}",
        "LOG_LEVEL": "${LOG_LEVEL:-info}"
      }
    }
  }
}
```

### 환경 변수 패턴

| 패턴 | 설명 | 예시 |
|------|------|------|
| `${VAR}` | 변수 값 사용 | `${API_KEY}` |
| `${VAR:-default}` | 기본값 지정 | `${PORT:-8080}` |
| `${CLAUDE_PLUGIN_ROOT}` | 플러그인 루트 경로 | 플러그인 내 파일 참조 |

### 환경 변수 관리

```bash
# .env 파일 (프로젝트 루트)
API_KEY=your-api-key
DATABASE_URL=postgresql://user:pass@localhost/db
LOG_LEVEL=debug

# 시스템 환경 변수
export API_KEY="your-api-key"

# Claude Code 실행 시 전달
API_KEY="key" claude
```

## 인증 설정

### OAuth 2.0 인증

```bash
# 1. 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. Claude Code에서 인증
> /mcp
# Sentry 선택 → "Authenticate" 클릭
# 브라우저에서 OAuth 로그인

# 3. 인증 확인
> /mcp
# "Authenticated: ✓" 표시 확인
```

### 토큰 기반 인증

```bash
# 헤더에 토큰 포함
claude mcp add --transport http github https://api.github.com/mcp/ \
  --header "Authorization: token YOUR_GITHUB_TOKEN"

# Bearer 토큰
claude mcp add --transport http api https://api.example.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

### 인증 관리 명령어

```
# Claude Code 내부
> /mcp

# 옵션:
# - "Authenticate": 인증 시작
# - "Clear authentication": 인증 삭제
# - "Refresh": 토큰 갱신
```

## 출력 및 토큰 관리

### 환경 변수 설정

```bash
# MCP 출력 최대 토큰
export MAX_MCP_OUTPUT_TOKENS=50000  # 기본값: 25,000

# MCP 타임아웃
export MCP_TIMEOUT=10000  # 밀리초

# 디버그 모드
export MCP_DEBUG=true
```

### 대용량 출력 처리

```typescript
// 서버 측: 페이지네이션 구현
async function listItems(cursor?: string, limit = 100) {
  const results = await db.query(`
    SELECT * FROM items
    WHERE id > $1
    ORDER BY id
    LIMIT $2
  `, [cursor || '0', limit]);

  return {
    items: results.rows,
    nextCursor: results.rows.length === limit
      ? results.rows[results.rows.length - 1].id
      : null,
  };
}
```

## 플러그인 MCP 설정

### 플러그인 내 MCP 서버

```json
// plugin.json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "mcpServers": {
    "plugin-api": {
      "type": "stdio",
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"],
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

### 플러그인 구조

```
my-plugin/
├── plugin.json
├── servers/
│   ├── api-server           # 실행 파일
│   └── api-server.js        # 또는 스크립트
├── config.json
└── README.md
```

## 엔터프라이즈 설정

### 관리형 MCP (managed-mcp.json)

시스템 관리자가 배포하는 고정 서버 세트:

```bash
# 파일 위치
# macOS: /Library/Application Support/ClaudeCode/managed-mcp.json
# Linux: /etc/claude-code/managed-mcp.json
# Windows: C:\Program Files\ClaudeCode\managed-mcp.json
```

```json
{
  "mcpServers": {
    "company-api": {
      "type": "http",
      "url": "https://internal.company.com/mcp"
    }
  }
}
```

**특징**:
- 사용자가 수정 불가
- 모든 사용자에게 자동 적용

### 정책 기반 제어

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": ["npx", "-y", "approved-package"] },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" },
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

### 패턴 매칭

| 패턴 | 설명 | 예시 |
|------|------|------|
| `serverName` | 정확한 이름 매칭 | `"github"` |
| `serverCommand` | 명령어 배열 매칭 | `["npx", "-y", "pkg"]` |
| `serverUrl` | URL 와일드카드 | `"https://*.company.com/*"` |

## 트러블슈팅

### 일반적인 문제

#### 1. 연결 실패

```bash
# 문제: "Connection closed" 에러

# 해결: Windows에서 cmd /c 래퍼 사용
claude mcp add --transport stdio server -- cmd /c npx -y package

# 해결: 경로 확인
which npx  # 경로가 올바른지 확인
```

#### 2. 인증 실패

```bash
# 문제: "Authentication failed" 에러

# 해결 1: 인증 초기화
> /mcp
# "Clear authentication" 선택 후 재인증

# 해결 2: 토큰 갱신
# 환경 변수의 토큰이 만료되지 않았는지 확인
```

#### 3. 타임아웃

```bash
# 문제: "Request timed out" 에러

# 해결: 타임아웃 증가
export MCP_TIMEOUT=30000  # 30초

# 서버 측: 응답 최적화 또는 페이지네이션 구현
```

### 디버깅

```bash
# 디버그 모드 활성화
export MCP_DEBUG=true
claude

# 서버 로그 확인
# Stdio 서버는 stderr로 로그 출력
node server.js 2> server.log
```

### MCP 상태 확인

```
# Claude Code 내부
> /mcp

# 표시 정보:
# - 연결된 서버 목록
# - 인증 상태
# - 사용 가능한 도구
# - 리소스
```

## 설정 파일 참조

### ~/.claude.json 구조

```json
{
  "mcpServers": {
    "local-server": {
      "type": "stdio",
      "command": "/path/to/server",
      "args": ["--flag"],
      "env": {
        "KEY": "value"
      }
    },
    "remote-server": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer token"
      }
    }
  },
  "mcpServerScopes": {
    "local-server": "local",
    "remote-server": "user"
  }
}
```

### .mcp.json 구조 (프로젝트)

```json
{
  "mcpServers": {
    "project-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@company/mcp-server"],
      "env": {
        "PROJECT_ID": "${PROJECT_ID}"
      }
    }
  }
}
```

## 다음 단계

- [MCP 예제](04-examples.md) - 실전 사용 예제
