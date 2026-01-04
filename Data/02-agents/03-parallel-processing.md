# 병렬 처리와 Sub-Agent

## 개요

Claude Code는 Task 도구를 사용하여 여러 Sub-Agent를 동시에 실행할 수 있습니다. 이를 통해 복잡한 작업을 효율적으로 분할 처리할 수 있습니다.

## 병렬 처리 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                    메인 Claude Code                          │
│                                                             │
│  "코드 품질과 보안을 동시에 분석해줘"                          │
└────────────────────────────┬────────────────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
       ┌──────────┐   ┌──────────┐   ┌──────────┐
       │ Quality  │   │ Security │   │ Perf     │
       │ Analyzer │   │ Analyst  │   │ Analyzer │
       │          │   │          │   │          │
       │ 작업 중...│   │ 작업 중...│   │ 작업 중...│
       └────┬─────┘   └────┬─────┘   └────┬─────┘
            │              │              │
            └──────────────┼──────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    결과 통합                                 │
│  - 품질 이슈 5건                                             │
│  - 보안 취약점 2건                                           │
│  - 성능 개선점 3건                                           │
└─────────────────────────────────────────────────────────────┘
```

## Task 도구 사용법

### 기본 구조

```typescript
{
  "description": "작업 설명 (3-5 단어)",
  "prompt": "상세 작업 지시",
  "subagent_type": "agent-name",
  "model": "sonnet",              // 선택: sonnet, opus, haiku
  "run_in_background": false      // 선택: 백그라운드 실행
}
```

### 병렬 실행 패턴

여러 Task를 동시에 호출하려면 단일 메시지에서 여러 Task 도구를 사용합니다:

```
메인 Agent:
"세 가지 분석을 병렬로 실행하겠습니다"

[Task 1: code-quality]
[Task 2: security-analysis]
[Task 3: performance-check]

→ 세 Agent가 동시에 실행됨
```

## 실전 병렬 처리 패턴

### 패턴 1: 다각도 분석

하나의 코드를 여러 관점에서 동시 분석:

```
사용자: "이 코드를 전체적으로 분석해줘"

메인 Agent:
├─ Task 1: code-reviewer (품질 분석)
├─ Task 2: security-analyst (보안 분석)
└─ Task 3: performance-analyzer (성능 분석)

결과 통합:
"분석 완료:
- 품질: 리팩토링 필요 3곳
- 보안: 취약점 1건 발견
- 성능: 최적화 가능 2곳"
```

### 패턴 2: 분할 정복

큰 작업을 여러 부분으로 나누어 처리:

```
사용자: "모든 API 엔드포인트의 문서를 업데이트해줘"

메인 Agent:
├─ Task 1: doc-writer → /api/users/**
├─ Task 2: doc-writer → /api/orders/**
└─ Task 3: doc-writer → /api/products/**

결과 통합:
"문서 업데이트 완료:
- Users API: 5개 엔드포인트
- Orders API: 8개 엔드포인트
- Products API: 6개 엔드포인트"
```

### 패턴 3: 탐색 + 실행

먼저 탐색하고 결과를 바탕으로 실행:

```
단계 1: 병렬 탐색
├─ Explore: 인증 관련 파일 찾기
├─ Explore: 권한 관련 파일 찾기
└─ Explore: 세션 관련 파일 찾기

단계 2: 결과 기반 실행
└─ General Agent: 발견된 파일들을 분석하고 개선
```

## 백그라운드 실행

### 언제 사용하는가?

- 오래 걸리는 작업
- 결과를 즉시 필요로 하지 않는 작업
- 다른 작업과 동시에 진행해야 하는 작업

### 사용 방법

```typescript
// 백그라운드로 실행
{
  "description": "Large codebase analysis",
  "prompt": "전체 코드베이스의 테스트 커버리지를 분석해줘",
  "subagent_type": "test-analyzer",
  "run_in_background": true
}

// 결과 확인
TaskOutput({ task_id: "agent-id", block: true })
```

### TaskOutput 옵션

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `task_id` | Agent ID | 필수 |
| `block` | 완료까지 대기 | true |
| `timeout` | 최대 대기 시간 (ms) | 30000 |

## Agent 체이닝

### 순차 체이닝

한 Agent의 결과를 다음 Agent의 입력으로:

```
단계 1: Explore Agent
└─ "취약한 코드 패턴을 찾아줘"
   → 결과: vulnerable_files.txt

단계 2: Security Analyst Agent (resume)
└─ "발견된 파일들을 상세 분석해줘"
   → 결과: detailed_analysis.md

단계 3: Code Fixer Agent
└─ "분석 결과를 바탕으로 수정해줘"
   → 결과: 코드 수정 완료
```

### Resume 기능

이전 Agent의 컨텍스트를 유지하며 계속:

```typescript
// 첫 번째 실행
Task({
  description: "Initial analysis",
  prompt: "파일 구조를 분석해줘",
  subagent_type: "explorer"
})
// 결과: agentId: "abc123"

// 이어서 실행 (컨텍스트 유지)
Task({
  description: "Continue analysis",
  prompt: "이제 의존성을 분석해줘",
  subagent_type: "explorer",
  resume: "abc123"  // 이전 실행 이어서
})
```

## 병렬 처리 베스트 프랙티스

### 1. 독립성 확인

병렬로 실행할 작업들이 서로 의존성이 없는지 확인:

```
✓ 독립적인 작업들 - 병렬 가능
├─ src/auth/ 분석
├─ src/api/ 분석
└─ src/utils/ 분석

✗ 의존적인 작업들 - 순차 실행 필요
├─ 1. 스키마 분석 (먼저)
└─ 2. 마이그레이션 생성 (스키마 결과 필요)
```

### 2. 적절한 Agent 선택

작업 특성에 맞는 Agent와 모델 선택:

| 작업 유형 | 권장 Agent | 모델 |
|----------|-----------|------|
| 빠른 파일 검색 | Explore | haiku |
| 코드 분석 | Custom Analyzer | sonnet |
| 복잡한 설계 | Plan | opus |
| 간단한 수정 | General | haiku |

### 3. 결과 통합 전략

```
병렬 실행 결과들:
├─ Agent 1: { issues: [...], severity: "high" }
├─ Agent 2: { issues: [...], severity: "medium" }
└─ Agent 3: { issues: [...], severity: "low" }

통합 방식:
1. 심각도별 정렬
2. 중복 제거
3. 우선순위 결정
4. 사용자에게 요약 제공
```

### 4. 에러 처리

```
병렬 실행 중 한 Agent 실패 시:
├─ Agent 1: ✓ 성공
├─ Agent 2: ✗ 실패 (타임아웃)
└─ Agent 3: ✓ 성공

대응:
1. 성공한 결과 먼저 보고
2. 실패한 작업 재시도 옵션 제공
3. 부분 결과로 진행 가능 여부 확인
```

## Claude Agent SDK에서의 병렬 처리

### TypeScript 예제

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "코드 품질과 보안을 동시에 분석해줘",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Task"],
    agents: {
      "code-reviewer": {
        description: "코드 품질 리뷰어",
        prompt: "코드 품질을 분석하고 개선점을 제안하세요",
        tools: ["Read", "Glob", "Grep"]
      },
      "security-analyst": {
        description: "보안 분석가",
        prompt: "보안 취약점을 찾고 수정 방법을 제안하세요",
        tools: ["Read", "Grep"]
      }
    }
  }
})) {
  if ("result" in message) {
    console.log(message.result);
  }
}
```

### Python 예제

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition

async def parallel_analysis():
    async for message in query(
        prompt="코드를 여러 관점에서 분석해줘",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep", "Task"],
            agents={
                "quality-checker": AgentDefinition(
                    description="코드 품질 검사기",
                    prompt="코드 품질을 검사하세요",
                    tools=["Read", "Glob", "Grep"]
                ),
                "test-coverage": AgentDefinition(
                    description="테스트 커버리지 분석기",
                    prompt="테스트 커버리지를 분석하세요",
                    tools=["Read", "Grep"]
                )
            }
        )
    ):
        if hasattr(message, "result"):
            print(message.result)

asyncio.run(parallel_analysis())
```

## 성능 고려사항

### 병렬 처리의 장점

- **시간 단축**: 독립 작업 동시 실행
- **컨텍스트 보존**: 메인 대화 컨텍스트 유지
- **전문화**: 각 Agent가 특화된 작업 수행

### 오버헤드

- **시작 비용**: 각 Sub-Agent 시작에 지연
- **컨텍스트 생성**: 새 컨텍스트 윈도우 생성
- **결과 통합**: 여러 결과 병합 필요

### 최적화 팁

1. **적절한 분할**: 너무 작은 작업으로 분할하지 않기
2. **모델 선택**: 간단한 작업에는 haiku 사용
3. **캐싱 활용**: 반복적인 탐색 결과 재사용
4. **타임아웃 설정**: 적절한 타임아웃으로 리소스 관리

## 다음 단계

- [베스트 프랙티스](04-best-practices.md) - Agent 설계 원칙
