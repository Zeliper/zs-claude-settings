# MCP 서버 제작 가이드

## 개요

MCP 서버는 Claude에게 외부 시스템과 상호작용할 수 있는 기능을 제공합니다. 이 가이드에서는 처음부터 MCP 서버를 만드는 방법을 설명합니다.

## MCP 서버의 핵심 구성요소

| 구성요소 | 설명 | 예시 |
|----------|------|------|
| **Resources** | 읽을 수 있는 데이터 | 파일, DB 레코드, API 응답 |
| **Tools** | 실행할 수 있는 함수 | 검색, 쿼리, 전송 |
| **Prompts** | 미리 정의된 워크플로우 | 리뷰 템플릿, 보고서 생성 |

## 개발 환경 설정

### TypeScript/Node.js

```bash
# 프로젝트 초기화
mkdir my-mcp-server
cd my-mcp-server
npm init -y

# 의존성 설치
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node ts-node

# TypeScript 설정
npx tsc --init
```

### Python

```bash
# 프로젝트 초기화
mkdir my-mcp-server
cd my-mcp-server
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install mcp pydantic
```

## 기본 MCP 서버 구현

### TypeScript 예제

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  ListResourcesRequestSchema,
  ReadResourceRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import { z } from "zod";

// 서버 인스턴스 생성
const server = new Server(
  {
    name: "my-mcp-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
      resources: {},
    },
  }
);

// 도구 스키마 정의
const SearchSchema = z.object({
  query: z.string().describe("검색할 키워드"),
  limit: z.number().optional().default(10).describe("결과 개수"),
});

// 도구 목록 핸들러
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "search",
        description: "데이터베이스에서 항목을 검색합니다",
        inputSchema: {
          type: "object",
          properties: {
            query: { type: "string", description: "검색 키워드" },
            limit: { type: "number", description: "결과 개수" },
          },
          required: ["query"],
        },
      },
      {
        name: "get_item",
        description: "ID로 특정 항목을 조회합니다",
        inputSchema: {
          type: "object",
          properties: {
            id: { type: "string", description: "항목 ID" },
          },
          required: ["id"],
        },
      },
    ],
  };
});

// 도구 실행 핸들러
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "search": {
      const { query, limit } = SearchSchema.parse(args);
      // 실제 검색 로직 구현
      const results = await performSearch(query, limit);
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(results, null, 2),
          },
        ],
      };
    }

    case "get_item": {
      const { id } = args as { id: string };
      const item = await getItemById(id);
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(item, null, 2),
          },
        ],
      };
    }

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// 리소스 목록 핸들러
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "myapp://items/recent",
        name: "Recent Items",
        description: "최근 추가된 항목 목록",
        mimeType: "application/json",
      },
    ],
  };
});

// 리소스 읽기 핸들러
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;

  if (uri === "myapp://items/recent") {
    const items = await getRecentItems();
    return {
      contents: [
        {
          uri,
          mimeType: "application/json",
          text: JSON.stringify(items, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown resource: ${uri}`);
});

// 비즈니스 로직 (예시)
async function performSearch(query: string, limit: number) {
  // 실제 검색 구현
  return [
    { id: "1", title: `Result for "${query}"`, score: 0.95 },
    { id: "2", title: `Another result for "${query}"`, score: 0.87 },
  ].slice(0, limit);
}

async function getItemById(id: string) {
  return { id, title: "Sample Item", createdAt: new Date().toISOString() };
}

async function getRecentItems() {
  return [
    { id: "1", title: "Item 1" },
    { id: "2", title: "Item 2" },
  ];
}

// 서버 시작
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP Server running on stdio");
}

main().catch(console.error);
```

### Python 예제

```python
# server.py
import asyncio
import json
from typing import Any
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import (
    Tool,
    TextContent,
    Resource,
)

# 서버 인스턴스 생성
server = Server("my-mcp-server")

# 도구 정의
@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="search",
            description="데이터베이스에서 항목을 검색합니다",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "검색 키워드"},
                    "limit": {"type": "number", "description": "결과 개수"},
                },
                "required": ["query"],
            },
        ),
        Tool(
            name="get_item",
            description="ID로 특정 항목을 조회합니다",
            inputSchema={
                "type": "object",
                "properties": {
                    "id": {"type": "string", "description": "항목 ID"},
                },
                "required": ["id"],
            },
        ),
    ]

# 도구 실행
@server.call_tool()
async def call_tool(name: str, arguments: dict[str, Any]) -> list[TextContent]:
    if name == "search":
        query = arguments["query"]
        limit = arguments.get("limit", 10)
        results = await perform_search(query, limit)
        return [TextContent(type="text", text=json.dumps(results, indent=2))]

    elif name == "get_item":
        item_id = arguments["id"]
        item = await get_item_by_id(item_id)
        return [TextContent(type="text", text=json.dumps(item, indent=2))]

    raise ValueError(f"Unknown tool: {name}")

# 리소스 정의
@server.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="myapp://items/recent",
            name="Recent Items",
            description="최근 추가된 항목 목록",
            mimeType="application/json",
        ),
    ]

# 리소스 읽기
@server.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "myapp://items/recent":
        items = await get_recent_items()
        return json.dumps(items, indent=2)

    raise ValueError(f"Unknown resource: {uri}")

# 비즈니스 로직
async def perform_search(query: str, limit: int) -> list[dict]:
    return [
        {"id": "1", "title": f'Result for "{query}"', "score": 0.95},
        {"id": "2", "title": f'Another result for "{query}"', "score": 0.87},
    ][:limit]

async def get_item_by_id(item_id: str) -> dict:
    return {"id": item_id, "title": "Sample Item", "createdAt": "2024-01-15"}

async def get_recent_items() -> list[dict]:
    return [
        {"id": "1", "title": "Item 1"},
        {"id": "2", "title": "Item 2"},
    ]

# 서버 실행
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream)

if __name__ == "__main__":
    asyncio.run(main())
```

## 고급 기능 구현

### 데이터베이스 연결

```typescript
// db-server.ts
import { Pool } from "pg";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

// 쿼리 도구
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "query") {
    const { sql } = request.params.arguments as { sql: string };

    // 읽기 전용 쿼리만 허용 (보안)
    if (!sql.trim().toLowerCase().startsWith("select")) {
      throw new Error("Only SELECT queries are allowed");
    }

    const result = await pool.query(sql);
    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(result.rows, null, 2),
        },
      ],
    };
  }
});
```

### 외부 API 연결

```typescript
// api-server.ts
import fetch from "node-fetch";

const API_BASE = process.env.API_BASE_URL;
const API_KEY = process.env.API_KEY;

async function callExternalAPI(endpoint: string, params: Record<string, any>) {
  const response = await fetch(`${API_BASE}${endpoint}`, {
    headers: {
      "Authorization": `Bearer ${API_KEY}`,
      "Content-Type": "application/json",
    },
    method: "POST",
    body: JSON.stringify(params),
  });

  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }

  return response.json();
}
```

### 파일 시스템 접근

```typescript
// file-server.ts
import { readdir, readFile, stat } from "fs/promises";
import { join } from "path";

const ALLOWED_DIR = process.env.ALLOWED_DIR || "/data";

server.setRequestHandler(ListResourcesRequestSchema, async () => {
  const files = await readdir(ALLOWED_DIR);
  return {
    resources: files.map((file) => ({
      uri: `file://${join(ALLOWED_DIR, file)}`,
      name: file,
      mimeType: "text/plain",
    })),
  };
});

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;
  const path = uri.replace("file://", "");

  // 경로 검증 (디렉토리 순회 방지)
  if (!path.startsWith(ALLOWED_DIR)) {
    throw new Error("Access denied");
  }

  const content = await readFile(path, "utf-8");
  return {
    contents: [{ uri, mimeType: "text/plain", text: content }],
  };
});
```

## Prompts 구현

```typescript
// prompts-server.ts
import { ListPromptsRequestSchema, GetPromptRequestSchema } from "@modelcontextprotocol/sdk/types.js";

server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: [
      {
        name: "code_review",
        description: "코드 리뷰 요청을 생성합니다",
        arguments: [
          { name: "file_path", description: "리뷰할 파일 경로", required: true },
          { name: "focus", description: "집중할 영역", required: false },
        ],
      },
      {
        name: "bug_report",
        description: "버그 리포트 템플릿을 생성합니다",
        arguments: [
          { name: "title", description: "버그 제목", required: true },
          { name: "severity", description: "심각도", required: true },
        ],
      },
    ],
  };
});

server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "code_review") {
    const filePath = args?.file_path || "";
    const focus = args?.focus || "general quality";

    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `다음 파일을 리뷰해주세요: ${filePath}\n집중 영역: ${focus}\n\n코드 품질, 보안, 성능 측면에서 분석해주세요.`,
          },
        },
      ],
    };
  }

  if (name === "bug_report") {
    const title = args?.title || "";
    const severity = args?.severity || "medium";

    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `## 버그 리포트\n\n**제목**: ${title}\n**심각도**: ${severity}\n\n### 재현 단계\n1. \n\n### 예상 동작\n\n### 실제 동작\n`,
          },
        },
      ],
    };
  }

  throw new Error(`Unknown prompt: ${name}`);
});
```

## 빌드 및 패키징

### package.json

```json
{
  "name": "my-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "my-mcp-server": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^0.5.0",
    "zod": "^3.22.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.3.0",
    "ts-node": "^10.9.0"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true
  },
  "include": ["src/**/*"]
}
```

### 빌드 및 실행

```bash
# 빌드
npm run build

# 로컬 테스트
npm start

# Claude Code에 등록
claude mcp add --transport stdio my-server -- node /path/to/dist/index.js
```

## 디버깅

### 로그 출력

```typescript
// stderr로 로그 출력 (stdout은 MCP 통신에 사용)
console.error("[DEBUG] Processing request:", request);

// 환경 변수로 로그 레벨 제어
const DEBUG = process.env.DEBUG === "true";
if (DEBUG) {
  console.error("[DEBUG] Detailed info...");
}
```

### 테스트 클라이언트

```typescript
// test-client.ts
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

const client = new Client({ name: "test-client", version: "1.0.0" });

async function test() {
  const transport = new StdioClientTransport({
    command: "node",
    args: ["dist/index.js"],
  });

  await client.connect(transport);

  // 도구 목록 테스트
  const tools = await client.listTools();
  console.log("Available tools:", tools);

  // 도구 호출 테스트
  const result = await client.callTool({
    name: "search",
    arguments: { query: "test", limit: 5 },
  });
  console.log("Search result:", result);
}

test().catch(console.error);
```

## 보안 고려사항

### 1. 입력 검증

```typescript
import { z } from "zod";

const QuerySchema = z.object({
  query: z.string().min(1).max(1000),
  limit: z.number().int().min(1).max(100).default(10),
});

// 사용
const validated = QuerySchema.parse(request.params.arguments);
```

### 2. 권한 제어

```typescript
// 읽기 전용 작업만 허용
const READONLY_OPERATIONS = ["search", "get", "list"];

if (!READONLY_OPERATIONS.includes(request.params.name)) {
  throw new Error("Operation not permitted");
}
```

### 3. 경로 검증

```typescript
import { resolve, normalize } from "path";

function validatePath(inputPath: string, baseDir: string): string {
  const resolved = resolve(baseDir, inputPath);
  const normalized = normalize(resolved);

  if (!normalized.startsWith(baseDir)) {
    throw new Error("Path traversal detected");
  }

  return normalized;
}
```

### 4. 비밀 정보 관리

```typescript
// 환경 변수에서 로드
const API_KEY = process.env.API_KEY;

if (!API_KEY) {
  console.error("API_KEY environment variable is required");
  process.exit(1);
}

// 절대 로그에 출력하지 않음
console.error("[INFO] API key loaded: ***hidden***");
```

## 다음 단계

- [MCP 설정](03-configuration.md) - 서버 설정 및 배포
- [MCP 예제](04-examples.md) - 실전 서버 예제
