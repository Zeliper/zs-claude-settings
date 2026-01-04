# Agent 개발 워크플로우

## 개요

이 문서는 커스텀 Claude Code Agent를 설계하고 구현하는 전체 워크플로우를 설명합니다.

## Agent 개발 프로세스

```
┌─────────────────────────────────────────────────────────────┐
│                   Agent 개발 워크플로우                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐     │
│  │ 1. 설계 │──▶│ 2. 구현 │──▶│ 3. 테스트│──▶│ 4. 배포 │     │
│  │         │   │         │   │         │   │         │     │
│  └─────────┘   └─────────┘   └─────────┘   └────┬────┘     │
│       │                                         │          │
│       │                                         ▼          │
│       │                                   ┌─────────┐      │
│       └──────────────────────────────────▶│ 5. 개선 │      │
│                 피드백 루프                └─────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Agent 설계

### 1.1 요구사항 분석

```markdown
## 요구사항 체크리스트

### 목적
- [ ] Agent가 해결할 문제는 무엇인가?
- [ ] 기존 내장 Agent로 충분하지 않은 이유는?
- [ ] 어떤 전문성이 필요한가?

### 범위
- [ ] 어떤 작업을 수행해야 하는가?
- [ ] 어떤 작업은 수행하면 안 되는가?
- [ ] 예상 입력과 출력은?

### 제약
- [ ] 어떤 도구가 필요한가?
- [ ] 어떤 도구는 제한해야 하는가?
- [ ] 보안 고려사항은?
```

### 1.2 Agent 유형 결정

| 유형 | 특징 | 적합한 사용처 |
|------|------|-------------|
| 읽기 전용 | Read, Grep, Glob만 사용 | 분석, 리뷰, 탐색 |
| 수정 가능 | Write 포함 | 코드 생성, 문서 작성 |
| 시스템 접근 | Bash 포함 | 빌드, 테스트, 배포 |

### 1.3 설계 문서 작성

```markdown
# Agent 설계: Security Auditor

## 목적
코드의 보안 취약점을 분석하고 수정 방안을 제시

## 책임
- OWASP Top 10 취약점 검사
- 민감 데이터 노출 확인
- 인증/인가 로직 검토

## 제외 범위
- 코드 직접 수정 (권장 사항만 제시)
- 인프라 보안 (코드 레벨만)

## 필요 도구
- Read: 코드 읽기
- Grep: 패턴 검색
- Glob: 파일 탐색

## 출력 형식
- 심각도별 분류
- 파일:라인 위치
- 문제 설명 + 수정 방안
```

---

## Phase 2: Agent 구현

### 2.1 기본 구조 작성

```yaml
---
name: security-auditor
description: Analyzes code for security vulnerabilities and suggests fixes. Use when checking code security or performing security reviews.
tools: Read, Grep, Glob
model: sonnet
---

# Security Auditor

당신은 애플리케이션 보안 전문가입니다.
```

### 2.2 시스템 프롬프트 상세화

```markdown
# Security Auditor

당신은 애플리케이션 보안 전문가입니다.

## 역할

코드베이스의 보안 취약점을 분석하고 수정 방안을 제시합니다.

## 분석 영역

### OWASP Top 10
1. **Injection** - SQL, NoSQL, OS, LDAP 인젝션
2. **Broken Authentication** - 세션 관리, 인증 우회
3. **Sensitive Data Exposure** - 암호화, 키 관리
4. **XXE** - XML 외부 엔티티
5. **Broken Access Control** - 권한 상승, IDOR

### 추가 검사
- 하드코딩된 자격 증명
- 안전하지 않은 의존성
- 로깅에 민감 정보 노출

## 분석 절차

1. **파일 탐색**: 관련 파일 식별
2. **입력 추적**: 사용자 입력 경로 분석
3. **데이터 흐름**: 데이터가 어떻게 처리되는지
4. **출력 확인**: 외부 노출 지점

## 출력 형식

각 발견에 대해:

```
## [Critical/High/Medium/Low] 취약점 제목

**위치**: 파일경로:라인번호
**유형**: OWASP 카테고리
**설명**: 취약점 상세 설명
**영향**: 악용 시 발생할 수 있는 피해
**수정**: 권장 수정 방법
```

## 제약사항

- 코드를 직접 수정하지 않습니다
- 권장 사항만 제시합니다
- 불확실한 경우 추가 분석을 제안합니다
```

### 2.3 파일 생성

```bash
# 프로젝트 Agent (팀 공유)
mkdir -p .claude/agents
cat > .claude/agents/security-auditor.md << 'EOF'
---
name: security-auditor
description: Analyzes code for security vulnerabilities and suggests fixes. Use when checking code security or performing security reviews.
tools: Read, Grep, Glob
model: sonnet
---

[위에서 작성한 시스템 프롬프트]
EOF

# 개인 Agent
mkdir -p ~/.claude/agents
# 같은 방식으로 파일 생성
```

### 2.4 Claude에게 생성 요청

```
더 쉬운 방법:

> "보안 분석을 위한 Agent를 만들어줘.
   OWASP Top 10을 검사하고, 발견된 취약점을
   심각도별로 분류해서 보고해야 해.
   코드 수정은 하지 않고 권장 사항만 제시해야 해."

Claude가 Agent 파일을 생성해줍니다.
```

---

## Phase 3: 테스트

### 3.1 단위 테스트

```markdown
## 테스트 케이스

### TC1: SQL 인젝션 탐지
입력: SQL 인젝션 취약점이 있는 코드
기대: Critical/High로 보고, 파라미터화된 쿼리 권장
파일: test_cases/sql_injection.py

### TC2: XSS 탐지
입력: XSS 취약점이 있는 코드
기대: High로 보고, 이스케이프 처리 권장
파일: test_cases/xss_vulnerable.js

### TC3: 안전한 코드
입력: 취약점 없는 코드
기대: "주요 취약점 없음" 보고
파일: test_cases/secure_code.py
```

### 3.2 테스트 실행

```bash
# Claude Code 재시작
claude

# Agent 목록 확인
> /agents

# 테스트 실행
> "@security-auditor test_cases/sql_injection.py 를 분석해줘"

# 또는
> "이 코드의 보안 취약점을 분석해줘"
# → Claude가 security-auditor 자동 선택
```

### 3.3 결과 평가

```markdown
## 평가 기준

### 정확성
- [ ] 취약점을 정확히 식별했는가?
- [ ] 오탐(false positive)이 없는가?
- [ ] 미탐(false negative)이 없는가?

### 유용성
- [ ] 수정 권장 사항이 실행 가능한가?
- [ ] 설명이 충분히 상세한가?
- [ ] 심각도 분류가 적절한가?

### 형식
- [ ] 출력 형식이 일관적인가?
- [ ] 파일:라인 정보가 정확한가?
```

### 3.4 모델별 테스트

```bash
# Haiku로 테스트 (빠른 분석)
# Agent 파일에서 model: haiku로 변경 후 테스트

# Opus로 테스트 (심층 분석)
# Agent 파일에서 model: opus로 변경 후 테스트
```

---

## Phase 4: 배포

### 4.1 프로젝트 배포

```bash
# Agent 파일을 프로젝트에 포함
cp ~/.claude/agents/security-auditor.md .claude/agents/

# 버전 관리에 추가
git add .claude/agents/security-auditor.md
git commit -m "feat(agents): add security auditor agent"
git push
```

### 4.2 팀 공유

```markdown
# Security Auditor Agent 사용 가이드

## 설치
이 저장소를 클론하면 자동으로 Agent가 로드됩니다.

## 사용법

### 자동 호출
"보안 분석해줘", "취약점 찾아줘" 등의 질문 시 자동 활성화

### 명시적 호출
> "@security-auditor src/ 디렉토리를 분석해줘"

## 출력 예시
[예시 출력]

## 제한사항
- 코드 수정은 직접 하지 않음
- 인프라 보안은 범위 외

## 피드백
#security-team 채널 또는 GitHub Issue
```

### 4.3 문서화

```bash
# README 또는 CONTRIBUTING.md에 Agent 정보 추가

## Custom Agents

### Security Auditor
보안 취약점 분석을 위한 Agent입니다.
상세 사용법: [링크]
```

---

## Phase 5: 지속적 개선

### 5.1 피드백 수집

```markdown
## 피드백 로그

### 2024-01-15 - @developer1
문제: Java 코드에서 SQL 인젝션 미탐지
원인: 패턴이 Python 중심으로 작성됨
조치: Java JDBC 패턴 추가

### 2024-01-18 - @developer2
문제: 오탐이 너무 많음
원인: 단순 문자열 매칭으로 인한 오탐
조치: 컨텍스트 분석 지침 추가
```

### 5.2 반복 개선

```
개선 루프:

1. 피드백 수집 (1주)
   └─> 문제점 정리

2. 분석 (1일)
   └─> 근본 원인 파악

3. 수정 (1일)
   └─> 시스템 프롬프트 개선

4. 테스트 (1일)
   └─> 기존 + 새 테스트 케이스

5. 배포 (1일)
   └─> 팀에 공지
```

### 5.3 버전 관리

```bash
# 버전 태깅
git tag -a v1.1.0 -m "Add Java SQL injection patterns"

# 변경 로그 유지
cat >> CHANGELOG.md << 'EOF'
## v1.1.0 - 2024-01-20
### Added
- Java JDBC SQL injection 패턴
- PreparedStatement 권장 사항

### Fixed
- 오탐 감소를 위한 컨텍스트 분석 개선
EOF
```

---

## 실전 예제: Test Generator Agent

### 요구사항 분석

```markdown
목적: 코드에 대한 단위 테스트 자동 생성
범위: Python, TypeScript 코드
도구: Read, Grep, Write
출력: pytest/jest 형식의 테스트 코드
```

### 구현

```yaml
---
name: test-generator
description: Generates unit tests for Python and TypeScript code. Use when creating tests or when the user asks for test cases.
tools: Read, Grep, Glob, Write
model: sonnet
---

# Test Generator

당신은 테스트 코드 전문가입니다.

## 지원 언어 및 프레임워크

| 언어 | 프레임워크 | 파일 패턴 |
|------|-----------|----------|
| Python | pytest | test_*.py |
| TypeScript | jest | *.test.ts |

## 테스트 작성 원칙

### AAA 패턴 (Arrange-Act-Assert)

```python
def test_example():
    # Arrange - 준비
    input_data = create_data()

    # Act - 실행
    result = function_under_test(input_data)

    # Assert - 검증
    assert result == expected
```

### 테스트 케이스 구성

1. **Happy Path** - 정상 동작
2. **Edge Cases** - 경계 조건 (빈 값, 최대값)
3. **Error Cases** - 예외 상황

### 명명 규칙

```python
# test_<무엇을>_<상황>_<기대결과>
def test_calculate_total_with_discount_returns_reduced_price():
    pass
```

## 워크플로우

1. 대상 함수/클래스 분석
2. 입력/출력 타입 파악
3. 테스트 케이스 설계
4. 테스트 코드 생성
5. 실행 명령어 제공

## 출력 형식

```python
# tests/test_<module>.py

import pytest
from <module> import <function>

class Test<ClassName>:
    """<클래스/함수> 테스트"""

    def test_<scenario>(self):
        """<테스트 설명>"""
        # Arrange
        ...
        # Act
        ...
        # Assert
        ...
```
```

### 테스트 실행

```
> "@test-generator src/utils/calculator.py에 대한 테스트 생성해줘"

# 결과: tests/test_calculator.py 생성
```

---

## 병렬 Agent 개발 워크플로우

### 여러 Agent 동시 개발

```markdown
## 프로젝트: Code Quality Suite

### Phase 1: 설계 (1일)
동시에 3개 Agent 설계:
- code-reviewer
- security-auditor
- performance-analyzer

### Phase 2: 구현 (2일)
각 Agent 독립적으로 구현
공통 출력 형식 통일

### Phase 3: 통합 테스트 (1일)
병렬 실행 테스트:
> "이 PR을 품질, 보안, 성능 관점에서 분석해줘"

### Phase 4: 배포 (1일)
모든 Agent 동시 배포
통합 문서화
```

---

## 체크리스트 요약

### 설계 시

- [ ] 목적이 명확히 정의됨
- [ ] 책임 범위가 한정됨
- [ ] 필요한 도구만 선정됨

### 구현 시

- [ ] name이 네이밍 규칙 준수
- [ ] description이 구체적
- [ ] 시스템 프롬프트가 상세함
- [ ] 출력 형식이 정의됨

### 테스트 시

- [ ] 다양한 케이스 테스트
- [ ] 여러 모델에서 테스트
- [ ] 오탐/미탐 확인

### 배포 시

- [ ] 버전 관리에 포함
- [ ] 팀 문서화 완료
- [ ] 피드백 채널 설정
