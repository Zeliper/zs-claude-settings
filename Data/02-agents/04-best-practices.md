# Agent 베스트 프랙티스

## 설계 원칙

### 1. 단일 책임 원칙

각 Agent는 명확하고 집중된 하나의 책임을 가져야 합니다.

#### 좋은 예: 집중된 책임

```yaml
---
name: sql-optimizer
description: Optimizes SQL queries for performance. Use when analyzing or improving SQL query performance.
tools: Read, Grep
---

# SQL Query Optimizer

당신은 SQL 쿼리 최적화 전문가입니다.

## 책임 범위
- 쿼리 실행 계획 분석
- 인덱스 사용 최적화
- 조인 순서 개선
- 서브쿼리 최적화
```

#### 나쁜 예: 너무 광범위한 책임

```yaml
---
name: database-helper
description: Helps with everything database related.
tools: Read, Write, Bash, Grep
---

# Database Helper

당신은 데이터베이스 관련 모든 것을 도와줍니다:
- 스키마 설계
- 쿼리 작성
- 마이그레이션
- 백업/복구
- 성능 튜닝
- 보안 감사
...
```

### 2. 최소 권한 원칙

Agent에게는 필요한 도구만 부여합니다.

| Agent 유형 | 권장 도구 |
|-----------|----------|
| 분석/리뷰 | Read, Grep, Glob |
| 문서 작성 | Read, Grep, Write |
| 코드 수정 | Read, Grep, Glob, Write |
| 시스템 작업 | Read, Grep, Bash |

```yaml
# 읽기 전용 분석 Agent
---
name: code-analyzer
tools: Read, Grep, Glob  # Write, Bash 제외
---

# 수정 권한 있는 Agent
---
name: code-fixer
tools: Read, Grep, Glob, Write
---
```

### 3. 명확한 Description

Description은 Claude가 Agent를 선택하는 데 사용하는 핵심 정보입니다.

#### Description 구성 요소

```yaml
description: [무엇을 하는가] + [언제 사용하는가] + [트리거 키워드]
```

#### 좋은 Description 예시

```yaml
# 명확한 용도 + 트리거 조건
description: Reviews code for quality issues and suggests improvements. Use when reviewing pull requests, code changes, or when the user asks for code review.

# 구체적인 도메인 + 키워드
description: Analyzes React components for performance issues. Use when optimizing React apps or when the user mentions React performance, re-renders, or memo.
```

#### 나쁜 Description 예시

```yaml
# 너무 모호
description: Helps with code

# 트리거 조건 없음
description: Reviews code quality

# 너무 장황
description: This agent is designed to help developers who need assistance with their code by providing reviews and suggestions...
```

## 시스템 프롬프트 작성

### 구조화된 프롬프트

```yaml
---
name: my-agent
---

# Agent 역할 정의

당신은 [역할]입니다.

## 주요 책임

1. [책임 1]
2. [책임 2]
3. [책임 3]

## 작업 절차

### Step 1: [단계명]
상세 지침...

### Step 2: [단계명]
상세 지침...

## 출력 형식

결과는 다음 형식으로 제공:

```
[출력 템플릿]
```

## 주의사항

- [주의점 1]
- [주의점 2]

## 예제

### 입력 예시
[예시 입력]

### 출력 예시
[예시 출력]
```

### 프롬프트 길이 가이드라인

| 복잡도 | 권장 길이 | 예시 |
|--------|----------|------|
| 단순 | 50-100줄 | 파일 검색 |
| 중간 | 100-200줄 | 코드 리뷰 |
| 복잡 | 200-400줄 | 아키텍처 분석 |

## 모델 선택 가이드

### 작업별 권장 모델

| 작업 유형 | 권장 모델 | 이유 |
|----------|----------|------|
| 빠른 검색/탐색 | haiku | 속도, 비용 |
| 일반 분석/리뷰 | sonnet | 균형 |
| 복잡한 추론/설계 | opus | 정확도 |
| 코드 생성 | sonnet | 품질/비용 균형 |
| 보안 분석 | opus | 정밀함 필요 |

### 모델 설정 예시

```yaml
# 빠른 탐색용
---
name: quick-explorer
model: haiku
tools: Read, Grep, Glob
---

# 심층 분석용
---
name: deep-analyzer
model: opus
tools: Read, Grep
---
```

## 도구 접근 제어

### 읽기 전용 Agent

분석/리뷰 목적의 Agent:

```yaml
---
name: security-auditor
tools: Read, Grep, Glob
permissionMode: default
---

# Security Auditor

이 Agent는 코드를 읽고 분석만 합니다.
어떤 파일도 수정하지 않습니다.
```

### 수정 권한 Agent

코드 변경이 필요한 Agent:

```yaml
---
name: code-migrator
tools: Read, Grep, Glob, Write, Bash
permissionMode: default
---

# Code Migrator

이 Agent는 코드 마이그레이션을 수행합니다.
파일을 읽고, 분석하고, 수정할 수 있습니다.
```

### 권한 모드

| 모드 | 설명 | 사용 시나리오 |
|------|------|-------------|
| `default` | 표준 권한 확인 | 일반적인 사용 |
| `plan` | 계획 모드 (읽기만) | 설계/분석 |
| `bypassPermissions` | 권한 우회 | 자동화 (주의 필요) |

## 버전 관리와 협업

### 프로젝트 Agent 관리

```bash
.claude/
└── agents/
    ├── code-reviewer.md      # 팀 코드 리뷰어
    ├── test-generator.md     # 테스트 생성기
    └── doc-writer.md         # 문서 작성기
```

### Git 관리

```bash
# Agent 파일 버전 관리
git add .claude/agents/
git commit -m "feat(agents): add code-reviewer agent"

# .gitignore에서 제외하지 않기
# Agent는 팀과 공유해야 함
```

### 팀 협업 패턴

1. **PR 리뷰**: Agent 변경도 코드 리뷰
2. **테스트**: 새 Agent 테스트 케이스 포함
3. **문서화**: Agent 사용법 문서화
4. **피드백 루프**: 사용 결과 기반 개선

## 디버깅과 개선

### Agent가 예상대로 동작하지 않을 때

1. **Description 확인**
   - 충분히 구체적인가?
   - 트리거 키워드가 포함되었는가?

2. **프롬프트 검토**
   - 지침이 명확한가?
   - 모호한 부분은 없는가?

3. **도구 확인**
   - 필요한 도구가 모두 있는가?
   - 불필요한 도구가 포함되지 않았는가?

4. **모델 적합성**
   - 작업 복잡도에 맞는 모델인가?

### 반복적 개선 프로세스

```
1. 초기 Agent 생성
   └─> 기본 동작 확인

2. 실제 작업 수행
   └─> 문제점 파악

3. 프롬프트 조정
   └─> 지침 명확화, 예제 추가

4. 테스트
   └─> 다양한 시나리오 확인

5. 피드백 반영
   └─> 팀 피드백 적용

6. 반복
```

## 흔한 실수와 해결책

### 실수 1: 너무 광범위한 Agent

**문제**: 하나의 Agent가 너무 많은 일을 함

**해결**: 책임 분리
```
before: all-in-one-helper
after:
  - code-reviewer
  - security-analyzer
  - performance-checker
```

### 실수 2: 불명확한 출력 형식

**문제**: Agent 출력이 일관되지 않음

**해결**: 명시적 출력 템플릿
```markdown
## 출력 형식

항상 다음 형식으로 응답:

### 발견 사항
- [ ] 항목 1
- [ ] 항목 2

### 권장 조치
1. 첫 번째 조치
2. 두 번째 조치
```

### 실수 3: 과도한 도구 권한

**문제**: 읽기만 필요한데 Write, Bash 권한 부여

**해결**: 최소 권한 원칙 적용
```yaml
# Before
tools: Read, Write, Bash, Grep, Glob

# After
tools: Read, Grep, Glob
```

### 실수 4: 모델 불일치

**문제**: 간단한 작업에 opus 사용

**해결**: 작업에 맞는 모델 선택
```yaml
# 파일 검색 - haiku로 충분
model: haiku

# 복잡한 보안 분석 - opus 권장
model: opus
```

## 체크리스트

### Agent 생성 전

- [ ] 단일 책임이 명확히 정의되었는가?
- [ ] 유사한 기존 Agent가 없는가?
- [ ] 팀과 공유할 것인가, 개인용인가?

### Agent 작성 시

- [ ] Description이 구체적인가?
- [ ] 시스템 프롬프트가 명확한가?
- [ ] 필요한 도구만 포함했는가?
- [ ] 적절한 모델을 선택했는가?
- [ ] 출력 형식이 정의되었는가?

### Agent 테스트 시

- [ ] 다양한 입력으로 테스트했는가?
- [ ] 엣지 케이스를 확인했는가?
- [ ] 다른 모델에서도 테스트했는가?

### Agent 배포 후

- [ ] 팀에 사용법을 공유했는가?
- [ ] 피드백 채널이 있는가?
- [ ] 개선 계획이 있는가?
