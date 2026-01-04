# CLAUDE.md 생성 방법

## 기본 CLAUDE.md 생성 프로세스

### Step 1: 파일 생성

프로젝트 루트 디렉토리에 `CLAUDE.md` 파일을 생성합니다.

```bash
# 프로젝트 디렉토리로 이동
cd your-project

# CLAUDE.md 파일 생성
touch CLAUDE.md
```

Windows의 경우:
```powershell
# 프로젝트 디렉토리에서
New-Item -Path "CLAUDE.md" -ItemType File
```

### Step 2: 기본 구조 작성

다음 기본 구조로 시작합니다:

```markdown
# Project: [프로젝트명]

## 프로젝트 개요
[프로젝트가 무엇인지 1-2문장으로 설명]

## 기술 스택
- [기술 1]
- [기술 2]

## 코딩 컨벤션
- [규칙 1]
- [규칙 2]

## 워크플로우
- [작업 방식 1]
- [작업 방식 2]
```

### Step 3: 프로젝트 맥락 추가

프로젝트에 맞는 구체적인 내용을 채웁니다:

```markdown
# Project: Task Manager API

## 프로젝트 개요
RESTful API 기반의 태스크 관리 백엔드 서비스입니다.

## 기술 스택
- Runtime: Node.js 20
- Framework: Express.js
- Database: PostgreSQL 15
- ORM: Prisma
- Testing: Jest + Supertest

## 코딩 컨벤션
- 모든 함수는 JSDoc 주석 필수
- 에러는 커스텀 Error 클래스 사용
- API 응답 형식: { success: boolean, data?: T, error?: string }

## 워크플로우
- 새 기능: feature 브랜치 생성 → 구현 → 테스트 → PR
- 버그 수정: hotfix 브랜치 사용
```

### Step 4: 검증

Claude Code를 시작하여 CLAUDE.md가 제대로 로드되는지 확인합니다:

```bash
# Claude Code 시작
claude

# Claude에게 프로젝트에 대해 질문
> 이 프로젝트의 기술 스택이 뭐야?
```

Claude가 CLAUDE.md에 명시된 정보를 바탕으로 응답하면 성공입니다.

---

## CLAUDE.md 구조 상세

### 권장 섹션 순서

효과적인 CLAUDE.md는 다음 순서로 구성합니다:

```
┌─────────────────────────────────────┐
│ 1. 프로젝트 제목과 개요              │ ← 가장 먼저
├─────────────────────────────────────┤
│ 2. 기술 스택                         │
├─────────────────────────────────────┤
│ 3. 디렉토리 구조                     │
├─────────────────────────────────────┤
│ 4. 코딩 컨벤션                       │
├─────────────────────────────────────┤
│ 5. 워크플로우                        │
├─────────────────────────────────────┤
│ 6. 금지 사항                         │ ← 명확히 구분
├─────────────────────────────────────┤
│ 7. 참고 자료 (선택)                  │
└─────────────────────────────────────┘
```

### 각 섹션별 역할

| 섹션 | 역할 | 필수 여부 |
|------|------|----------|
| 프로젝트 개요 | 프로젝트의 목적과 범위 설명 | 필수 |
| 기술 스택 | 사용 중인 언어, 프레임워크, 도구 나열 | 필수 |
| 디렉토리 구조 | 주요 폴더와 파일 위치 설명 | 권장 |
| 코딩 컨벤션 | 코드 스타일, 네이밍 규칙 등 | 권장 |
| 워크플로우 | 작업 프로세스, 브랜치 전략 등 | 선택 |
| 금지 사항 | 하지 말아야 할 것들 명시 | 권장 |
| 참고 자료 | 관련 문서, 링크 등 | 선택 |

---

## 섹션별 작성법

### 프로젝트 설명

프로젝트의 핵심을 1-3문장으로 요약합니다.

#### 좋은 예
```markdown
## 프로젝트 개요
실시간 협업 문서 편집 플랫폼입니다. WebSocket을 통한 동시 편집과
버전 히스토리 관리를 지원합니다.
```

#### 나쁜 예
```markdown
## 프로젝트 개요
이 프로젝트는 우리 회사에서 사용하는 문서 편집 도구인데,
처음에는 간단한 메모 앱으로 시작했다가 점점 기능이 추가되어서...
(장황한 히스토리 설명)
```

### 기술 스택

구체적인 버전과 함께 나열합니다.

#### 좋은 예
```markdown
## 기술 스택
- Language: TypeScript 5.3
- Frontend: React 18 + Vite
- State: Zustand
- Styling: Tailwind CSS 3.4
- Testing: Vitest + Testing Library
```

#### 나쁜 예
```markdown
## 기술 스택
- JavaScript
- React
- CSS
```

### 코딩 컨벤션

Claude가 따라야 할 구체적인 규칙을 명시합니다.

#### 좋은 예
```markdown
## 코딩 컨벤션

### 네이밍
- 컴포넌트: PascalCase (예: `UserProfile.tsx`)
- 함수: camelCase (예: `getUserById`)
- 상수: SCREAMING_SNAKE_CASE (예: `MAX_RETRY_COUNT`)
- 타입: PascalCase + 접미사 (예: `UserDto`, `AuthService`)

### 파일 구조
- 컴포넌트당 하나의 파일
- index.ts로 re-export 금지 (직접 import 사용)

### 주석
- 복잡한 로직에만 주석 추가
- TODO는 이슈 번호와 함께 (예: `// TODO(#123): 리팩토링 필요`)
```

#### 나쁜 예
```markdown
## 코딩 컨벤션
- 좋은 코드를 작성하세요
- 읽기 쉽게 작성하세요
```

### 워크플로우 규칙

작업 프로세스와 브랜치 전략을 설명합니다.

#### 좋은 예
```markdown
## 워크플로우

### 브랜치 전략
- `main`: 프로덕션 브랜치
- `develop`: 개발 브랜치
- `feature/*`: 기능 개발
- `hotfix/*`: 긴급 수정

### 커밋 메시지
- 형식: `type(scope): description`
- 예: `feat(auth): add OAuth2 login`
- 타입: feat, fix, docs, style, refactor, test, chore

### PR 규칙
- 최소 1명 리뷰 필수
- 모든 테스트 통과 필수
- 컨플릭트 해결 후 머지
```

### 금지 사항

Claude가 하지 말아야 할 것을 명확히 명시합니다.

#### 좋은 예
```markdown
## 금지 사항

### 절대 하지 말 것
- `any` 타입 사용 금지 (대신 `unknown` 사용)
- `console.log` 커밋 금지 (logger 사용)
- 직접 DOM 조작 금지 (React ref 사용)

### 주의할 것
- API 키나 시크릿을 코드에 하드코딩하지 말 것
- 테스트 없이 비즈니스 로직 추가하지 말 것
```

---

## 트러블슈팅

| 문제 | 원인 | 해결책 |
|------|------|--------|
| CLAUDE.md가 로드되지 않음 | 파일 위치가 잘못됨 | 프로젝트 루트에 `CLAUDE.md` 배치 확인 |
| 일부 내용만 반영됨 | 파일이 너무 김 | 핵심 내용만 남기고 축약 |
| 사용자 설정과 충돌 | 중복 규칙 존재 | 프로젝트 CLAUDE.md가 우선 적용됨 확인 |
| 마크다운 형식 오류 | 문법 오류 | 마크다운 프리뷰어로 검증 |

### CLAUDE.md가 반영되지 않을 때

1. 파일 이름 확인: `CLAUDE.md` (대문자)
2. 파일 위치 확인: 프로젝트 루트 디렉토리
3. 파일 인코딩 확인: UTF-8
4. Claude Code 재시작

---

## 다음 단계

1. [베스트 프랙티스](03-best-practices.md) - 효과적인 CLAUDE.md 작성 요령
2. [실전 예제](04-examples.md) - 프로젝트 유형별 템플릿
3. [개요로 돌아가기](01-overview.md) - CLAUDE.md 기본 개념 복습
