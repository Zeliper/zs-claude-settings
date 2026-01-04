# MCP 실전 예제

## 예제 1: Sentry 에러 모니터링

### 설정

```bash
# 서버 추가
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 인증
> /mcp
# Sentry 선택 → "Authenticate" → 브라우저 로그인
```

### 사용 예시

```
> "최근 24시간의 가장 흔한 에러를 보여줘"

> "에러 ID abc123의 스택 트레이스를 보여줘"

> "어떤 배포에서 이 새로운 에러들이 발생했어?"

> "프로덕션 환경의 에러 트렌드를 분석해줘"
```

### 워크플로우

```
1. 에러 현황 파악
   > "이번 주 critical 에러 목록"

2. 상세 분석
   > "가장 많이 발생하는 에러의 상세 정보"

3. 코드 수정
   > "이 에러를 수정하는 코드 작성해줘"

4. 확인
   > "수정 후 에러 발생률 확인"
```

---

## 예제 2: GitHub 통합

### 설정

```bash
# 서버 추가
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 또는 토큰 방식
claude mcp add --transport http github https://api.github.com/mcp/ \
  --header "Authorization: token YOUR_GITHUB_TOKEN"
```

### 사용 예시

```
> "PR #456을 리뷰하고 개선점을 제안해줘"

> "방금 발견한 버그에 대한 이슈를 생성해줘"

> "나에게 할당된 모든 열린 PR을 보여줘"

> "이번 주 머지된 PR 목록"
```

### 워크플로우: PR 리뷰

```
1. PR 확인
   > "PR #123의 변경사항을 분석해줘"

2. 자동 리뷰
   > "이 PR의 코드 품질과 보안을 검토해줘"

3. 코멘트 작성
   > "발견된 이슈에 대해 리뷰 코멘트를 작성해줘"

4. 승인/변경 요청
   > "문제가 없으면 승인해줘"
```

### 워크플로우: 이슈 관리

```
1. 이슈 조회
   > "bug 라벨이 붙은 열린 이슈들"

2. 이슈 분석
   > "@github:issue://456 을 분석해줘"

3. 이슈 생성
   > "새 버그 이슈 생성: 로그인 시 세션 만료 문제"

4. 이슈 업데이트
   > "이슈 #456에 재현 단계를 추가해줘"
```

---

## 예제 3: PostgreSQL 데이터베이스

### 설정

```bash
# 읽기 전용 연결 권장
claude mcp add --transport stdio db \
  --env DATABASE_URL="postgresql://readonly:pass@prod.db.com:5432/analytics" \
  -- npx -y @bytebase/dbhub
```

### 보안 주의사항

```
⚠️ 보안 권장사항:
- 읽기 전용 사용자 사용
- 민감한 테이블 접근 제한
- 쿼리 로깅 활성화
- IP 화이트리스트 설정
```

### 사용 예시

```
> "이번 달 총 매출은 얼마야?"

> "orders 테이블의 스키마를 보여줘"

> "90일 동안 구매하지 않은 고객 찾기"

> "일별 활성 사용자 수 추이"
```

### 워크플로우: 데이터 분석

```
1. 스키마 탐색
   > "데이터베이스의 모든 테이블 목록"
   > "users 테이블의 구조"

2. 기본 쿼리
   > "최근 가입한 사용자 10명"

3. 복잡한 분석
   > "월별 사용자 증가율과 이탈률"

4. 리포트 생성
   > "이 데이터로 마케팅 리포트 작성해줘"
```

---

## 예제 4: Playwright 브라우저 자동화

### 설정

```bash
claude mcp add --transport stdio playwright \
  -- npx -y @playwright/mcp@latest
```

### 사용 예시

```
> "example.com의 스크린샷을 찍어줘"

> "이 페이지에서 모든 링크를 추출해줘"

> "로그인 폼을 찾아서 테스트 계정으로 로그인해줘"

> "페이지 로드 성능을 측정해줘"
```

### 워크플로우: E2E 테스트 보조

```
1. 페이지 탐색
   > "staging.example.com/login 페이지 구조 분석"

2. 요소 식별
   > "로그인 버튼과 입력 필드의 셀렉터"

3. 액션 수행
   > "test@example.com으로 로그인 시도"

4. 결과 확인
   > "로그인 후 대시보드 페이지 확인"
```

---

## 예제 5: 커스텀 내부 API 서버

### 서버 구현

```typescript
// internal-api-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "internal-api", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 직원 검색 도구
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "search_employees",
      description: "직원 검색",
      inputSchema: {
        type: "object",
        properties: {
          query: { type: "string" },
          department: { type: "string" },
        },
        required: ["query"],
      },
    },
    {
      name: "get_org_chart",
      description: "조직도 조회",
      inputSchema: {
        type: "object",
        properties: {
          department: { type: "string" },
        },
      },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "search_employees") {
    const results = await searchInternalAPI("/employees", args);
    return { content: [{ type: "text", text: JSON.stringify(results) }] };
  }

  // ... 기타 도구 처리
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main();
```

### 설정

```bash
claude mcp add --transport stdio internal \
  --env API_KEY="${INTERNAL_API_KEY}" \
  -- node /path/to/internal-api-server.js
```

### 사용 예시

```
> "개발팀의 조직도를 보여줘"

> "김철수 님의 연락처 정보"

> "마케팅팀 인원은 몇 명이야?"
```

---

## 예제 6: Slack 통합

### 설정

```bash
claude mcp add --transport http slack https://mcp.slack.com/mcp

# 인증
> /mcp
# Slack 선택 → OAuth 로그인
```

### 사용 예시

```
> "#development 채널의 최근 메시지"

> "오늘 나에게 온 DM 요약"

> "팀 채널에 배포 완료 알림 보내줘"

> "@john.doe 에게 회의 요청 메시지 보내줘"
```

---

## 예제 7: Notion 문서 관리

### 설정

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

### 사용 예시

```
> "프로젝트 문서 목록을 보여줘"

> "회의록 템플릿으로 새 페이지 생성해줘"

> "API 문서에서 인증 방법 찾아줘"

> "이번 주 할 일 목록 업데이트"
```

---

## MCP 리소스 활용

### @ 멘션으로 리소스 참조

```
# GitHub 이슈 참조
> "@github:issue://123 을 분석하고 수정 방법을 제안해줘"

# 데이터베이스 스키마 참조
> "@postgres:schema://users 를 기반으로 API 엔드포인트 설계"

# 여러 리소스 비교
> "@github:pr://456 의 변경사항이 @docs:file://API.md 와 일치하는지 확인"
```

### 리소스 탐색

```
# 사용 가능한 리소스 보기
> @  # @ 입력 후 자동완성 확인

# 리소스 목록
> /mcp
# Resources 섹션 확인
```

---

## MCP Prompts 활용

### 프롬프트 실행

```
# MCP 프롬프트 목록 보기
> /  # / 입력 후 mcp__ 로 시작하는 항목 확인

# PR 리뷰 프롬프트
> /mcp__github__pr_review 456

# 이슈 생성 프롬프트
> /mcp__jira__create_issue "버그 제목" high

# 코드 분석 프롬프트
> /mcp__sentry__analyze_errors production
```

---

## 복합 워크플로우 예제

### 버그 수정 전체 과정

```
1. Sentry에서 에러 확인
   > "production에서 가장 많이 발생하는 에러"

2. GitHub 이슈 확인
   > "이 에러와 관련된 GitHub 이슈 찾기"

3. 코드 분석
   > "에러가 발생하는 코드 위치 찾기"

4. 수정 및 테스트
   > "수정 코드 작성 후 Playwright로 테스트"

5. PR 생성
   > "수정 내용으로 PR 생성"

6. 알림
   > "Slack에 수정 완료 알림 전송"
```

### 기능 개발 과정

```
1. 요구사항 확인
   > "@jira:issue://ENG-123 분석"

2. 설계
   > "기존 코드베이스 패턴에 맞게 설계"

3. 데이터 확인
   > "@postgres:schema://users 기반 API 설계"

4. 구현
   > "설계에 따라 코드 작성"

5. 문서화
   > "Notion에 API 문서 생성"

6. PR 생성
   > "구현 내용으로 PR 생성 및 리뷰 요청"
```

---

## 자주 사용하는 MCP 서버 목록

| 서비스 | 용도 | 설정 방법 |
|--------|------|----------|
| **GitHub** | 코드, 이슈, PR | HTTP + OAuth |
| **Sentry** | 에러 모니터링 | HTTP + OAuth |
| **Slack** | 팀 커뮤니케이션 | HTTP + OAuth |
| **Notion** | 문서 관리 | HTTP + OAuth |
| **PostgreSQL** | 데이터베이스 | Stdio + 환경변수 |
| **Playwright** | 브라우저 자동화 | Stdio (npx) |
| **Jira** | 이슈 트래킹 | HTTP + OAuth |
| **Figma** | 디자인 | HTTP + OAuth |
| **Gmail** | 이메일 | HTTP + OAuth |
| **Linear** | 이슈 트래킹 | HTTP + OAuth |

---

## 베스트 프랙티스

### 1. 보안

```bash
# 읽기 전용 권한 사용
# 민감한 토큰은 환경 변수로
# 프로덕션 DB는 읽기 전용 사용자
```

### 2. 성능

```bash
# 타임아웃 적절히 설정
export MCP_TIMEOUT=30000

# 대용량 데이터는 페이지네이션
# 캐싱 활용
```

### 3. 조직

```bash
# 팀 공유 서버는 .mcp.json에
# 개인 서버는 ~/.claude.json에
# 서버 이름은 설명적으로: prod-database, github-api
```

### 4. 문서화

```markdown
# 프로젝트 README에 MCP 설정 문서화
## MCP 서버 설정

프로젝트에서 사용하는 MCP 서버:
- github: PR/이슈 관리
- db: 읽기 전용 DB 접근

설정 방법:
1. .env 파일에 환경 변수 설정
2. claude mcp add로 서버 추가
```
