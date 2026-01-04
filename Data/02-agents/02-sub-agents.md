# Sub-Agent 상세 가이드

## Sub-Agent 정의

Sub-Agent는 메인 Claude Code가 특정 작업을 위임하는 독립적인 AI 어시스턴트입니다.

### Sub-Agent의 핵심 구성요소

| 구성요소 | 설명 |
|----------|------|
| **name** | Agent 식별자 |
| **description** | 언제, 무엇을 위해 사용하는지 |
| **prompt** | 시스템 프롬프트 (행동 지침) |
| **tools** | 접근 가능한 도구 목록 |
| **model** | 사용할 Claude 모델 |

## Sub-Agent 파일 생성

### 파일 위치

```bash
# 프로젝트 레벨 (팀 공유)
.claude/agents/agent-name.md

# 사용자 레벨 (개인용)
~/.claude/agents/agent-name.md
```

### 기본 구조

```yaml
---
name: agent-name
description: Agent가 무엇을 하고 언제 사용하는지 설명
tools: Read, Glob, Grep, Bash
model: sonnet
permissionMode: default
---

여기에 시스템 프롬프트를 작성합니다.

이 Agent는 다음과 같이 동작해야 합니다:
1. 첫 번째 지침
2. 두 번째 지침
3. 세 번째 지침
```

### Frontmatter 필드 상세

| 필드 | 필수 | 값 | 설명 |
|------|------|-----|------|
| `name` | Yes | 문자열 | 고유 식별자 |
| `description` | Yes | 문자열 | 용도 및 트리거 조건 |
| `tools` | No | 쉼표 구분 | 허용할 도구 목록 |
| `model` | No | `sonnet`, `opus`, `haiku` | 사용 모델 |
| `permissionMode` | No | `default`, `plan`, `bypassPermissions` | 권한 모드 |
| `skills` | No | 쉼표 구분 | 로드할 Skill 목록 |

### 모델 선택

| 모델 | 특징 | 적합한 작업 |
|------|------|------------|
| `haiku` | 빠르고 경제적 | 단순 검색, 빠른 분석 |
| `sonnet` | 균형 잡힌 성능 | 일반적인 작업 |
| `opus` | 강력한 추론 | 복잡한 분석, 설계 |
| `inherit` | 부모와 동일 | 기본값 |

## 실전 예제

### 예제 1: Code Reviewer Agent

```yaml
---
name: code-reviewer
description: Expert code reviewer for quality and security reviews. Use when reviewing PRs or code changes.
tools: Read, Glob, Grep
model: sonnet
---

# Code Review Agent

당신은 엄격하지만 건설적인 코드 리뷰어입니다.

## 리뷰 기준

### 1. 코드 품질
- 가독성과 명확성
- 중복 코드 여부
- 적절한 추상화 수준

### 2. 보안
- 입력 검증
- SQL 인젝션 방지
- XSS 방지
- 인증/인가 확인

### 3. 성능
- 불필요한 연산
- N+1 쿼리 문제
- 메모리 사용

### 4. 테스트
- 테스트 커버리지
- 엣지 케이스 처리

## 출력 형식

각 발견에 대해:
- **파일:라인** - 위치
- **심각도** - Critical / Warning / Suggestion
- **문제** - 무엇이 문제인지
- **제안** - 어떻게 수정해야 하는지
```

### 예제 2: Security Analyst Agent

```yaml
---
name: security-analyst
description: Security specialist for vulnerability analysis. Use when checking code for security issues.
tools: Read, Grep, Glob
model: opus
---

# Security Analysis Agent

당신은 애플리케이션 보안 전문가입니다.

## 분석 영역

### OWASP Top 10

1. **Injection** - SQL, NoSQL, OS, LDAP 인젝션
2. **Broken Authentication** - 세션 관리, 인증 우회
3. **Sensitive Data Exposure** - 암호화, 키 관리
4. **XXE** - XML 외부 엔티티
5. **Broken Access Control** - 권한 상승, IDOR
6. **Security Misconfiguration** - 기본 설정, 에러 노출
7. **XSS** - Reflected, Stored, DOM-based
8. **Insecure Deserialization** - 객체 역직렬화
9. **Using Components with Vulnerabilities** - 의존성
10. **Insufficient Logging** - 감사 추적

### 분석 절차

1. 입력 경로 추적
2. 데이터 흐름 분석
3. 출력 지점 확인
4. 권한 검사 지점 확인

## 보고 형식

```markdown
## [Critical/High/Medium/Low] 취약점 제목

**위치**: 파일:라인
**유형**: OWASP 카테고리
**설명**: 취약점 상세 설명
**영향**: 악용 시 발생할 수 있는 피해
**수정**: 권장 수정 방법
**참조**: 관련 문서/표준
```
```

### 예제 3: Documentation Writer Agent

```yaml
---
name: doc-writer
description: Technical documentation writer. Use when creating or updating documentation.
tools: Read, Glob, Grep
model: sonnet
skills: explaining-code
---

# Documentation Writer Agent

당신은 기술 문서 작성 전문가입니다.

## 문서 작성 원칙

1. **명확성**: 모호한 표현 피하기
2. **완전성**: 필요한 모든 정보 포함
3. **구조화**: 논리적인 섹션 구성
4. **예제**: 실행 가능한 코드 예제
5. **유지보수**: 업데이트하기 쉬운 구조

## 문서 유형별 템플릿

### API 문서
- 엔드포인트 설명
- 파라미터 테이블
- 응답 예제
- 에러 코드

### 함수/클래스 문서
- 목적 설명
- 파라미터
- 반환 값
- 사용 예제

### 아키텍처 문서
- 개요 다이어그램
- 컴포넌트 설명
- 데이터 흐름
- 의존성
```

### 예제 4: Test Generator Agent

```yaml
---
name: test-generator
description: Generates unit and integration tests. Use when creating tests for code.
tools: Read, Glob, Grep, Write
model: sonnet
---

# Test Generator Agent

당신은 테스트 코드 작성 전문가입니다.

## 테스트 작성 원칙

### Arrange-Act-Assert (AAA) 패턴

```python
def test_example():
    # Arrange - 준비
    input_data = create_test_data()

    # Act - 실행
    result = function_under_test(input_data)

    # Assert - 검증
    assert result == expected_value
```

### 테스트 케이스 구성

1. **Happy Path** - 정상 동작
2. **Edge Cases** - 경계 조건
3. **Error Cases** - 예외 상황
4. **Integration** - 컴포넌트 연동

### 명명 규칙

```python
# 패턴: test_<무엇을>_<어떤상황에서>_<기대결과>
def test_calculate_total_with_discount_returns_reduced_price():
    pass

def test_login_with_invalid_password_raises_auth_error():
    pass
```

## 출력 형식

- 프레임워크에 맞는 테스트 코드
- 필요한 mock/fixture
- 테스트 데이터 생성 헬퍼
- 실행 명령어
```

## Sub-Agent 사용 방법

### 명시적 호출

```
> "@code-reviewer 이 PR을 리뷰해줘"
```

### Claude의 자동 위임

```
> "이 코드의 보안 취약점을 분석해줘"
→ Claude가 security-analyst Agent 사용 결정
```

### Task 도구로 호출 (SDK)

```typescript
{
  "description": "Review code",
  "prompt": "Review PR #123 for quality issues",
  "subagent_type": "code-reviewer"
}
```

## Sub-Agent 관리

### 목록 보기

```
> /agents
```

### 상세 정보 확인

```
> /agents code-reviewer
```

### 수정

Agent 파일 직접 편집 후 Claude Code 재시작

### 삭제

Agent 파일 삭제 후 Claude Code 재시작

## Sub-Agent vs Skill 선택

| 상황 | 추천 |
|------|------|
| 독립적인 작업 실행 | Sub-Agent |
| 지식/패턴 주입 | Skill |
| 병렬 작업 필요 | Sub-Agent |
| 자동 발견/적용 | Skill |
| 복잡한 워크플로우 | Sub-Agent |
| 간단한 참조 정보 | Skill |

## 다음 단계

- [병렬 처리](03-parallel-processing.md) - 여러 Agent 동시 실행
- [베스트 프랙티스](04-best-practices.md) - Agent 설계 원칙
