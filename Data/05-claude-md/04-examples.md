# CLAUDE.md 실전 예제

## 프로젝트 유형별 예제

### 웹 애플리케이션 (React + TypeScript)

```markdown
# Project: E-Commerce Dashboard

## 프로젝트 개요
어드민용 이커머스 대시보드. 주문 관리, 상품 관리, 분석 기능 제공.

## 기술 스택
- Frontend: React 18 + TypeScript 5.3
- State: Zustand
- Styling: Tailwind CSS 3.4
- UI: Radix UI + shadcn/ui
- API: React Query + Axios
- Testing: Vitest + Testing Library

## 디렉토리 구조
src/
├── components/     # 재사용 가능한 UI 컴포넌트
├── features/       # 기능별 모듈 (orders, products, analytics)
├── hooks/          # 커스텀 훅
├── lib/            # 유틸리티 함수
├── services/       # API 호출
└── types/          # 타입 정의

## 코딩 컨벤션
- 컴포넌트: 함수형 + PascalCase (예: `OrderList.tsx`)
- 훅: camelCase + use 접두사 (예: `useOrders.ts`)
- 타입: PascalCase + 접미사 (예: `OrderDto`, `ProductResponse`)
- CSS: Tailwind 클래스 사용, 인라인 스타일 금지

## 금지 사항
- any 타입 사용 금지
- useEffect 내 데이터 페칭 금지 (React Query 사용)
- props drilling 금지 (3단계 이상 시 Context 사용)
```

---

### CLI 도구 (Node.js)

```markdown
# Project: deploy-cli

## 프로젝트 개요
AWS 인프라 배포 자동화 CLI 도구.

## 기술 스택
- Runtime: Node.js 20
- Language: TypeScript 5.3
- CLI: Commander.js
- Config: cosmiconfig
- Testing: Vitest

## 디렉토리 구조
src/
├── commands/       # CLI 명령어 정의
├── services/       # AWS SDK 래퍼
├── utils/          # 헬퍼 함수
└── types/          # 타입 정의

## 코딩 컨벤션
- 모든 명령어는 commands/ 폴더에 독립 파일로
- AWS 호출은 반드시 services/ 통해서
- 에러 메시지는 사용자 친화적으로 (AWS 에러 번역)

## 실행 방법
# 개발
npm run dev -- deploy --env staging

# 빌드
npm run build

# 테스트
npm test

## 금지 사항
- console.log 금지 → ora 스피너 또는 chalk 컬러 로그 사용
- 동기 파일 I/O 금지 → fs/promises 사용
- 하드코딩된 AWS 리전 금지 → 설정 파일에서 읽기
```

---

### Python 백엔드 (FastAPI)

```markdown
# Project: User Service API

## 프로젝트 개요
마이크로서비스 아키텍처의 사용자 인증/관리 서비스.

## 기술 스택
- Python: 3.12
- Framework: FastAPI
- ORM: SQLAlchemy 2.0
- Database: PostgreSQL 16
- Migration: Alembic
- Testing: pytest + httpx

## 디렉토리 구조
app/
├── api/            # 라우터 정의
├── core/           # 설정, 보안
├── models/         # SQLAlchemy 모델
├── schemas/        # Pydantic 스키마
├── services/       # 비즈니스 로직
└── utils/          # 유틸리티

## 코딩 컨벤션
- 타입 힌트 필수 (모든 함수 인자와 반환값)
- Pydantic v2 스타일 사용
- 비동기 함수 선호 (async def)
- docstring은 Google 스타일

## API 응답 형식
{
    "success": true,
    "data": {...},
    "message": "optional message"
}

## 금지 사항
- 직접 SQL 쿼리 금지 → SQLAlchemy ORM 사용
- 평문 패스워드 저장 금지 → bcrypt 해시
- 민감 정보 로깅 금지
```

---

### 모노레포 (Turborepo)

```markdown
# Project: Platform Monorepo

## 프로젝트 개요
웹앱, 모바일앱, 공유 라이브러리를 포함한 모노레포.

## 기술 스택
- Build: Turborepo
- Package Manager: pnpm
- Language: TypeScript 5.3

## 패키지 구조
apps/
├── web/            # Next.js 웹앱
├── mobile/         # React Native 앱
└── admin/          # 어드민 대시보드

packages/
├── ui/             # 공유 UI 컴포넌트
├── config/         # 공유 설정 (ESLint, TypeScript)
├── utils/          # 공유 유틸리티
└── types/          # 공유 타입 정의

## 작업 규칙
- 패키지 간 의존성: packages/* → apps/* (역방향 금지)
- 새 패키지 추가 시 turbo.json 업데이트 필수
- 공유 컴포넌트는 packages/ui에 추가

## 명령어
# 전체 빌드
pnpm build

# 특정 앱만 개발
pnpm dev --filter web

# 테스트
pnpm test

## 금지 사항
- apps/ 간 직접 import 금지
- 패키지 버전 수동 관리 금지 → changeset 사용
```

---

## 팀 협업용 CLAUDE.md

```markdown
# Project: Team Collaboration Platform

## 팀 정보
- 팀: Backend Squad
- 담당자: @lead-developer
- 슬랙: #backend-team

## 기술 스택
- 언어: Go 1.21
- 프레임워크: Gin
- DB: PostgreSQL + Redis
- 메시지 큐: RabbitMQ

## 코드 리뷰 규칙
- PR은 최소 2명 승인 필요
- 100줄 이상 변경 시 아키텍처 리뷰 요청
- 보안 관련 변경은 @security-team 태그

## 브랜치 전략
- main: 프로덕션 (직접 푸시 금지)
- develop: 개발 통합
- feature/JIRA-XXX: 기능 개발
- hotfix/JIRA-XXX: 긴급 수정

## 커밋 메시지
- 형식: [JIRA-XXX] type: description
- 예: [BACKEND-123] feat: add user authentication

## 온콜 규칙
- 프로덕션 배포는 온콜 담당자 승인 필요
- 금요일 오후 배포 금지
- 롤백 절차: docs/rollback.md 참조

## 금지 사항
- main 직접 푸시 금지
- 테스트 없이 PR 생성 금지
- 시크릿 하드코딩 금지 (Vault 사용)
```

---

## 개인 프로젝트용 CLAUDE.md

```markdown
# Project: Personal Blog

## 개요
Astro 기반 개인 블로그. 마크다운으로 글 작성.

## 기술 스택
- Framework: Astro 4
- Styling: Tailwind CSS
- Hosting: Vercel

## 디렉토리
src/
├── content/blog/   # 블로그 글 (*.md)
├── pages/          # 페이지
├── components/     # 컴포넌트
└── layouts/        # 레이아웃

## 글 작성 규칙
- 파일명: YYYY-MM-DD-title.md
- 프론트매터: title, date, tags 필수
- 이미지: public/images/posts/ 에 저장

## 배포
git push origin main → Vercel 자동 배포
```

---

## 템플릿 모음

### 최소 템플릿

프로젝트 시작 시 바로 사용할 수 있는 최소 구성:

```markdown
# Project: [프로젝트명]

## 개요
[한 줄 설명]

## 기술 스택
- [기술 1]
- [기술 2]

## 규칙
- [핵심 규칙 1]
- [핵심 규칙 2]
```

---

### 표준 템플릿

대부분의 프로젝트에 적합한 구성:

```markdown
# Project: [프로젝트명]

## 프로젝트 개요
[2-3문장으로 프로젝트 설명]

## 기술 스택
- Language: [언어 + 버전]
- Framework: [프레임워크 + 버전]
- Database: [DB + 버전]
- Testing: [테스트 도구]

## 디렉토리 구조
src/
├── [폴더1]/    # [설명]
├── [폴더2]/    # [설명]
└── [폴더3]/    # [설명]

## 코딩 컨벤션
- 네이밍: [규칙]
- 파일 구조: [규칙]
- 주석: [규칙]

## 워크플로우
- 브랜치: [전략]
- 커밋: [형식]
- PR: [규칙]

## 금지 사항
- [금지 1]
- [금지 2]
```

---

### 상세 템플릿

대규모 팀 프로젝트용:

```markdown
# Project: [프로젝트명]

## 팀 정보
- 팀: [팀명]
- 담당자: [이름]
- 채널: [커뮤니케이션 채널]

## 프로젝트 개요
[프로젝트 상세 설명]

## 기술 스택
### 백엔드
- [기술들]

### 프론트엔드
- [기술들]

### 인프라
- [기술들]

## 아키텍처 개요
[아키텍처 다이어그램 또는 설명]

## 디렉토리 구조
[상세 구조]

## 코딩 컨벤션
### 네이밍 규칙
### 파일 구조
### 타입 정의
### 에러 처리

## 워크플로우
### 개발 프로세스
### 코드 리뷰
### 배포

## API 규칙
### 엔드포인트 네이밍
### 응답 형식
### 에러 코드

## 테스트 규칙
### 단위 테스트
### 통합 테스트
### E2E 테스트

## 금지 사항
### 절대 금지
### 권장하지 않음

## 참고 자료
- [문서 링크]
```

---

## 다음 단계

1. [CLAUDE.md 개요](01-overview.md) - 기본 개념 복습
2. [CLAUDE.md 생성 방법](02-creating-claude-md.md) - 처음부터 시작하기
3. [베스트 프랙티스](03-best-practices.md) - 효과적인 작성 요령
