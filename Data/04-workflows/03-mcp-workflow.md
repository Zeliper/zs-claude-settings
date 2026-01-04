# MCP 서버 개발 워크플로우

## 개요

이 문서는 커스텀 MCP (Model Context Protocol) 서버를 개발하고 배포하는 전체 워크플로우를 설명합니다.

## MCP 서버 개발 프로세스

```
┌─────────────────────────────────────────────────────────────┐
│                 MCP 서버 개발 워크플로우                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐     │
│  │ 1. 설계 │──▶│ 2. 구현 │──▶│ 3. 테스트│──▶│ 4. 배포 │     │
│  │         │   │         │   │         │   │         │     │
│  └─────────┘   └─────────┘   └─────────┘   └────┬────┘     │
│       │                                         │          │
│       │         ┌───────────────────────────────┘          │
│       │         ▼                                          │
│       │    ┌─────────┐                                     │
│       └───▶│ 5. 유지 │                                     │
│            │  보수   │                                     │
│            └─────────┘                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1: 설계

### 1.1 요구사항 분석

```markdown
## MCP 서버 요구사항

### 연결할 시스템
- [ ] 어떤 외부 시스템과 연결하는가?
- [ ] API가 있는가? 문서는?
- [ ] 인증 방식은?

### 제공할 기능
- [ ] 어떤 데이터를 노출할 것인가? (Resources)
- [ ] 어떤 작업을 수행할 것인가? (Tools)
- [ ] 미리 정의된 워크플로우가 필요한가? (Prompts)

### 제약사항
- [ ] 읽기 전용인가, 쓰기도 허용하는가?
- [ ] 인증이 필요한가?
- [ ] 속도/성능 요구사항은?
```

### 1.2 아키텍처 결정

```markdown
## 아키텍처 결정

### Transport 유형
- [ ] HTTP: 원격 서비스용
- [ ] Stdio: 로컬 실행용

### 언어/프레임워크
- [ ] TypeScript/Node.js: npm 생태계 활용
- [ ] Python: 데이터/ML 작업에 적합

### 인증
- [ ] OAuth 2.0: 사용자별 인증
- [ ] API Key: 단순 인증
- [ ] 없음: 공개 데이터
```

### 1.3 설계 문서 작성

```markdown
# MCP 서버 설계: Internal Task Manager

## 목적
내부 태스크 관리 시스템과 연동

## 연결 대상
- REST API: https://tasks.internal.com/api/v1
- 인증: Bearer Token (환경 변수)

## 제공 기능

### Resources
- `tasks://active` - 활성 태스크 목록
- `tasks://my` - 내 태스크 목록
- `tasks://[id]` - 특정 태스크 상세

### Tools
- `list_tasks` - 태스크 목록 조회
- `get_task` - 태스크 상세 조회
- `create_task` - 새 태스크 생성
- `update_task` - 태스크 업데이트

### Prompts
- `weekly_report` - 주간 리포트 생성
- `task_summary` - 태스크 요약

## 기술 스택
- TypeScript
- Stdio transport
- 환경 변수로 API 토큰 전달
```

---

## Phase 2: 구현

### 2.1 프로젝트 초기화

```bash
# 프로젝트 생성
mkdir task-mcp-server
cd task-mcp-server
npm init -y

# 의존성 설치
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node ts-node

# TypeScript 설정
npx tsc --init --target ES2022 --module NodeNext \
  --moduleResolution NodeNext --outDir ./dist \
  --rootDir ./src --strict --esModuleInterop
```

### 2.2 프로젝트 구조

```
task-mcp-server/
├── src/
│   ├── index.ts          # 진입점
│   ├── server.ts         # MCP 서버 설정
│   ├── tools/
│   │   ├── index.ts      # 도구 내보내기
│   │   └── tasks.ts      # 태스크 도구 구현
│   ├── resources/
│   │   └── tasks.ts      # 리소스 구현
│   ├── prompts/
│   │   └── reports.ts    # 프롬프트 구현
│   └── api/
│       └── client.ts     # API 클라이언트
├── package.json
├── tsconfig.json
└── README.md
```

### 2.3 API 클라이언트 구현

```typescript
// src/api/client.ts
const API_BASE = process.env.TASK_API_URL || 'https://tasks.internal.com/api/v1';
const API_TOKEN = process.env.TASK_API_TOKEN;

if (!API_TOKEN) {
  console.error('TASK_API_TOKEN environment variable is required');
  process.exit(1);
}

export async function apiRequest(endpoint: string, options: RequestInit = {}) {
  const response = await fetch(`${API_BASE}${endpoint}`, {
    ...options,
    headers: {
      'Authorization': `Bearer ${API_TOKEN}`,
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });

  if (!response.ok) {
    throw new Error(`API error: ${response.status} ${response.statusText}`);
  }

  return response.json();
}

export async function listTasks(filter?: { status?: string; assignee?: string }) {
  const params = new URLSearchParams();
  if (filter?.status) params.set('status', filter.status);
  if (filter?.assignee) params.set('assignee', filter.assignee);
  return apiRequest(`/tasks?${params}`);
}

export async function getTask(id: string) {
  return apiRequest(`/tasks/${id}`);
}

export async function createTask(data: { title: string; description?: string; assignee?: string }) {
  return apiRequest('/tasks', {
    method: 'POST',
    body: JSON.stringify(data),
  });
}

export async function updateTask(id: string, data: { status?: string; title?: string }) {
  return apiRequest(`/tasks/${id}`, {
    method: 'PATCH',
    body: JSON.stringify(data),
  });
}
```

### 2.4 도구 구현

```typescript
// src/tools/tasks.ts
import { z } from 'zod';
import { listTasks, getTask, createTask, updateTask } from '../api/client.js';

export const taskTools = [
  {
    name: 'list_tasks',
    description: '태스크 목록을 조회합니다',
    inputSchema: {
      type: 'object',
      properties: {
        status: { type: 'string', enum: ['open', 'in_progress', 'done'] },
        assignee: { type: 'string' },
      },
    },
  },
  {
    name: 'get_task',
    description: '특정 태스크의 상세 정보를 조회합니다',
    inputSchema: {
      type: 'object',
      properties: {
        id: { type: 'string', description: '태스크 ID' },
      },
      required: ['id'],
    },
  },
  {
    name: 'create_task',
    description: '새로운 태스크를 생성합니다',
    inputSchema: {
      type: 'object',
      properties: {
        title: { type: 'string', description: '태스크 제목' },
        description: { type: 'string', description: '상세 설명' },
        assignee: { type: 'string', description: '담당자' },
      },
      required: ['title'],
    },
  },
  {
    name: 'update_task',
    description: '태스크를 업데이트합니다',
    inputSchema: {
      type: 'object',
      properties: {
        id: { type: 'string', description: '태스크 ID' },
        status: { type: 'string', enum: ['open', 'in_progress', 'done'] },
        title: { type: 'string' },
      },
      required: ['id'],
    },
  },
];

export async function handleToolCall(name: string, args: Record<string, any>) {
  switch (name) {
    case 'list_tasks':
      return await listTasks(args);

    case 'get_task':
      return await getTask(args.id);

    case 'create_task':
      return await createTask(args);

    case 'update_task':
      const { id, ...data } = args;
      return await updateTask(id, data);

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
}
```

### 2.5 서버 조립

```typescript
// src/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  ListToolsRequestSchema,
  CallToolRequestSchema,
  ListResourcesRequestSchema,
  ReadResourceRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';
import { taskTools, handleToolCall } from './tools/tasks.js';
import { listTasks, getTask } from './api/client.js';

const server = new Server(
  { name: 'task-mcp-server', version: '1.0.0' },
  { capabilities: { tools: {}, resources: {} } }
);

// 도구 목록
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: taskTools,
}));

// 도구 실행
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  try {
    const result = await handleToolCall(name, args || {});
    return {
      content: [{ type: 'text', text: JSON.stringify(result, null, 2) }],
    };
  } catch (error) {
    return {
      content: [{ type: 'text', text: `Error: ${error.message}` }],
      isError: true,
    };
  }
});

// 리소스 목록
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: 'tasks://active',
      name: 'Active Tasks',
      description: '진행 중인 태스크 목록',
      mimeType: 'application/json',
    },
    {
      uri: 'tasks://my',
      name: 'My Tasks',
      description: '내게 할당된 태스크',
      mimeType: 'application/json',
    },
  ],
}));

// 리소스 읽기
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;

  if (uri === 'tasks://active') {
    const tasks = await listTasks({ status: 'in_progress' });
    return {
      contents: [{ uri, mimeType: 'application/json', text: JSON.stringify(tasks, null, 2) }],
    };
  }

  if (uri === 'tasks://my') {
    const tasks = await listTasks({ assignee: 'me' });
    return {
      contents: [{ uri, mimeType: 'application/json', text: JSON.stringify(tasks, null, 2) }],
    };
  }

  // tasks://[id] 패턴
  const idMatch = uri.match(/^tasks:\/\/(.+)$/);
  if (idMatch) {
    const task = await getTask(idMatch[1]);
    return {
      contents: [{ uri, mimeType: 'application/json', text: JSON.stringify(task, null, 2) }],
    };
  }

  throw new Error(`Unknown resource: ${uri}`);
});

// 서버 시작
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('Task MCP Server running on stdio');
}

main().catch(console.error);
```

### 2.6 빌드

```bash
# 빌드
npm run build
# 또는
npx tsc

# 실행 테스트
TASK_API_TOKEN="test" node dist/index.js
```

---

## Phase 3: 테스트

### 3.1 단위 테스트

```typescript
// tests/tools.test.ts
import { describe, it, expect, vi } from 'vitest';
import { handleToolCall } from '../src/tools/tasks.js';

vi.mock('../src/api/client.js', () => ({
  listTasks: vi.fn().mockResolvedValue([
    { id: '1', title: 'Task 1', status: 'open' },
  ]),
  getTask: vi.fn().mockResolvedValue({
    id: '1', title: 'Task 1', status: 'open',
  }),
}));

describe('Task Tools', () => {
  it('list_tasks returns tasks', async () => {
    const result = await handleToolCall('list_tasks', {});
    expect(result).toHaveLength(1);
  });

  it('get_task returns task by id', async () => {
    const result = await handleToolCall('get_task', { id: '1' });
    expect(result.id).toBe('1');
  });
});
```

### 3.2 통합 테스트 (MCP Inspector)

```bash
# MCP Inspector 사용
npx @modelcontextprotocol/inspector dist/index.js

# 브라우저에서 도구/리소스 테스트
```

### 3.3 Claude Code 통합 테스트

```bash
# 서버 등록
claude mcp add --transport stdio tasks \
  --env TASK_API_TOKEN="your-token" \
  -- node /path/to/task-mcp-server/dist/index.js

# 테스트
> "내 태스크 목록을 보여줘"
> "새 태스크 만들어줘: 'MCP 서버 문서화'"
> "@tasks://123 상세 정보"
```

### 3.4 테스트 체크리스트

```markdown
## 테스트 체크리스트

### 도구
- [ ] list_tasks: 필터 없이 호출
- [ ] list_tasks: status 필터로 호출
- [ ] get_task: 존재하는 ID
- [ ] get_task: 존재하지 않는 ID (에러 처리)
- [ ] create_task: 필수 필드만
- [ ] create_task: 모든 필드
- [ ] update_task: status 변경

### 리소스
- [ ] tasks://active 로드
- [ ] tasks://my 로드
- [ ] tasks://[id] 로드

### 에러 처리
- [ ] API 에러 처리
- [ ] 인증 실패 처리
- [ ] 타임아웃 처리
```

---

## Phase 4: 배포

### 4.1 패키지 설정

```json
// package.json
{
  "name": "@myorg/task-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "bin": {
    "task-mcp-server": "dist/index.js"
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  }
}
```

### 4.2 npm 배포

```bash
# 로그인
npm login

# 배포
npm publish --access public

# 사용
claude mcp add --transport stdio tasks \
  --env TASK_API_TOKEN="token" \
  -- npx -y @myorg/task-mcp-server
```

### 4.3 프로젝트 배포 (.mcp.json)

```json
// .mcp.json
{
  "mcpServers": {
    "tasks": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@myorg/task-mcp-server"],
      "env": {
        "TASK_API_TOKEN": "${TASK_API_TOKEN}"
      }
    }
  }
}
```

### 4.4 문서화

```markdown
# Task MCP Server

## 설치

\`\`\`bash
claude mcp add --transport stdio tasks \
  --env TASK_API_TOKEN="your-token" \
  -- npx -y @myorg/task-mcp-server
\`\`\`

## 환경 변수

| 변수 | 필수 | 설명 |
|------|------|------|
| TASK_API_TOKEN | Yes | API 인증 토큰 |
| TASK_API_URL | No | API 기본 URL |

## 제공 도구

| 도구 | 설명 |
|------|------|
| list_tasks | 태스크 목록 조회 |
| get_task | 태스크 상세 조회 |
| create_task | 새 태스크 생성 |
| update_task | 태스크 업데이트 |

## 리소스

| URI | 설명 |
|-----|------|
| tasks://active | 진행 중인 태스크 |
| tasks://my | 내 태스크 |
| tasks://[id] | 특정 태스크 상세 |

## 사용 예시

\`\`\`
> "진행 중인 태스크 목록"
> "새 태스크 만들어줘: API 문서화"
> "@tasks://123 상세 정보"
\`\`\`
```

---

## Phase 5: 유지보수

### 5.1 버전 관리

```bash
# 변경 후 버전 업데이트
npm version patch  # 0.0.x
npm version minor  # 0.x.0
npm version major  # x.0.0

# 배포
npm publish
```

### 5.2 변경 로그

```markdown
# CHANGELOG.md

## [1.1.0] - 2024-01-20
### Added
- 태스크 삭제 도구 (delete_task)
- 태스크 코멘트 기능

### Fixed
- 빈 결과 처리 개선

## [1.0.0] - 2024-01-15
### Added
- 초기 릴리스
- 기본 CRUD 도구
- 리소스 지원
```

### 5.3 모니터링

```typescript
// 로깅 추가
const DEBUG = process.env.DEBUG === 'true';

function log(message: string, data?: any) {
  if (DEBUG) {
    console.error(`[${new Date().toISOString()}] ${message}`, data || '');
  }
}

// 사용
log('Tool called', { name, args });
log('API response', response);
```

### 5.4 에러 보고

```typescript
// 에러 추적
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  try {
    // ...
  } catch (error) {
    // 에러 로깅
    console.error('[ERROR]', {
      tool: request.params.name,
      args: request.params.arguments,
      error: error.message,
    });

    return {
      content: [{ type: 'text', text: `Error: ${error.message}` }],
      isError: true,
    };
  }
});
```

---

## 체크리스트 요약

### 설계 시

- [ ] 연결할 시스템 API 문서 확인
- [ ] 필요한 기능 (Tools, Resources, Prompts) 정의
- [ ] Transport 유형 결정
- [ ] 인증 방식 결정

### 구현 시

- [ ] 프로젝트 구조 설정
- [ ] API 클라이언트 구현
- [ ] 도구 구현 및 스키마 정의
- [ ] 리소스 구현
- [ ] 에러 처리

### 테스트 시

- [ ] 단위 테스트
- [ ] MCP Inspector로 통합 테스트
- [ ] Claude Code에서 실제 테스트
- [ ] 에러 케이스 테스트

### 배포 시

- [ ] package.json 설정
- [ ] npm 배포 또는 프로젝트 .mcp.json
- [ ] 문서화 완료
- [ ] 팀 공유

### 유지보수 시

- [ ] 버전 관리
- [ ] 변경 로그 유지
- [ ] 모니터링/로깅
- [ ] 에러 추적
