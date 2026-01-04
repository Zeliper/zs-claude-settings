# Skills 베스트 프랙티스

## 핵심 원칙

### 1. 간결하게 유지

Claude는 이미 매우 똑똑합니다. Skill에는 Claude가 **아직 모르는 정보**만 포함하세요.

#### 간결한 예시 (~50 토큰)

```markdown
## PDF 텍스트 추출

pdfplumber를 사용한 텍스트 추출:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
```

#### 장황한 예시 (~150 토큰) - 피해야 함

```markdown
## PDF 텍스트 추출

PDF (Portable Document Format) 파일은 널리 사용되는
문서 형식입니다. PDF 처리를 위한 다양한 라이브러리가
있지만, pdfplumber를 권장합니다. 사용하기 쉽고
대부분의 경우를 잘 처리하기 때문입니다. 먼저 pip으로
설치해야 합니다...
```

### 권장 사항

- SKILL.md는 **500줄 이하**로 유지
- Claude가 이미 알고 있는 일반적인 설명 제외
- 핵심 지침과 도메인 특화 정보에 집중

---

## 2. 효과적인 Description 작성

Description은 Claude가 Skill을 발견하는 데 사용하는 가장 중요한 요소입니다.

### Description 작성 규칙

| 규칙 | 설명 | 예시 |
|------|------|------|
| **3인칭 사용** | 발견 가능성 향상 | "Extracts text from PDFs" |
| **무엇 + 언제** | 기능과 트리거 조건 | "Use when working with PDF files" |
| **트리거 키워드** | 사용자가 자연스럽게 말할 단어 | "PDFs, forms, document extraction" |

### 좋은 Description 예시

```yaml
# PDF 처리
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# 커밋 메시지
description: Generates clear commit messages from git diffs. Use when writing commit messages, creating commits, or reviewing staged changes.

# 데이터 분석
description: Query BigQuery datasets for finance, sales, and product metrics. Use when analyzing business data, creating reports, or querying BigQuery tables.
```

### 나쁜 Description 예시

```yaml
# 너무 모호
description: Helps with documents

# 트리거 조건 없음
description: Processes data

# 1인칭 사용 (발견 어려움)
description: I help you work with PDFs
```

---

## 3. 네이밍 컨벤션

### 이름 규칙

| 규칙 | 설명 |
|------|------|
| **동명사 형태** | verb + -ing 형태 사용 |
| **소문자만** | 대문자 사용 금지 |
| **하이픈 사용** | 단어 구분에 하이픈 |
| **최대 64자** | 이름 길이 제한 |
| **디렉토리명 일치** | 폴더명과 name 필드 동일 |

### 좋은 이름 예시

```
processing-pdfs
analyzing-spreadsheets
managing-databases
testing-code
generating-reports
reviewing-code
```

### 피해야 할 이름

```
# 너무 모호
helper
utils
tools

# 일반적
documents
data
files

# 예약어
anthropic-helper
claude-tools
```

---

## 4. 자유도(Degrees of Freedom) 설정

작업의 민감도와 변동성에 따라 지침의 구체성 수준을 조절하세요.

### 높은 자유도 (텍스트 기반 지침)

여러 접근 방식이 유효하고 상황에 따라 최선이 달라질 때:

```markdown
## 코드 리뷰 프로세스

1. 코드 구조와 조직 분석
2. 잠재적 버그나 엣지 케이스 확인
3. 가독성 개선 제안
4. 프로젝트 컨벤션 준수 여부 검증
```

### 중간 자유도 (의사 코드 + 파라미터)

선호하는 패턴은 있지만 변형이 허용될 때:

```python
def generate_report(data, format="markdown", include_charts=True):
    # 데이터 처리
    # 지정된 형식으로 출력 생성
    # 선택적으로 시각화 포함
```

### 낮은 자유도 (구체적인 스크립트)

작업이 민감하고 특정 순서가 중요할 때:

```bash
python scripts/migrate.py --verify --backup
# 명령을 수정하거나 추가 플래그를 넣지 마세요
```

---

## 5. 모든 모델에서 테스트

Skill의 효과는 모델에 따라 달라집니다. 다양한 모델에서 테스트하세요:

| 모델 | 특징 | 테스트 포인트 |
|------|------|--------------|
| **Claude Haiku** | 경제적이지만 명확성 필요 | 지침이 충분히 명확한가? |
| **Claude Sonnet** | 균형 잡힌 성능 | 일반적인 사용에서 잘 작동하는가? |
| **Claude Opus** | 강력한 추론 능력 | 복잡한 작업을 잘 처리하는가? |

---

## 6. 점진적 공개(Progressive Disclosure)

대용량 정보를 효율적으로 관리하는 패턴입니다.

### 패턴 1: 개요 + 참조

```markdown
## Quick Start

기본 사용법 여기에...

## 상세 정보

- **폼 작성 가이드**: [FORMS.md](FORMS.md) 참조
- **API 레퍼런스**: [REFERENCE.md](REFERENCE.md) 참조
```

### 패턴 2: 도메인별 구성

```
bigquery-skill/
├── SKILL.md
└── reference/
    ├── finance.md (매출, 청구)
    ├── sales.md (파이프라인, 기회)
    ├── product.md (API 사용량, 채택)
    └── marketing.md (캠페인, 어트리뷰션)
```

### 패턴 3: 조건부 상세

```markdown
## 문서 생성

docx-js를 사용한 새 문서 생성. [DOCX-JS.md](DOCX-JS.md) 참조.

## 변경 추적이 필요한 경우

[REDLINING.md](REDLINING.md) 참조 (필요 시에만 로드)
```

---

## 7. 흔한 실수와 해결책

### 실수 1: Skill이 트리거되지 않음

**문제**: Skill을 만들었지만 Claude가 사용하지 않음

**해결책**:
```yaml
# Before: 너무 모호
description: Helps with documents

# After: 구체적 + 트리거 키워드
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

### 실수 2: 다중 Skill 충돌

**문제**: Claude가 잘못된 Skill을 선택함

**해결책**:
```yaml
# Before: 구분 어려움
# Skill 1
description: Analyzes data
# Skill 2
description: Processes data

# After: 명확한 구분
# Skill 1
description: Analyzes sales data in Excel files and CRM exports. Use for sales reports and metrics.
# Skill 2
description: Processes log files and system metrics. Use for infrastructure analysis.
```

### 실수 3: 스크립트 실행 실패

**문제**: Claude가 스크립트를 찾거나 실행할 수 없음

**해결책**:
```markdown
# Before: Windows 스타일 경로
scripts\helper.py

# After: Unix 스타일 경로 (모든 OS에서 작동)
scripts/helper.py

# 권한 확인
chmod +x scripts/*.py
```

### 실수 4: YAML 구문 오류

**문제**: Skill이 로드되지 않음

**해결책**:
```yaml
# Before: 공백 없음
name:skill-name

# Before: 탭 사용
name: skill-name
	description: ...

# After: 올바른 형식
---
name: skill-name
description: This is the description
---
```

---

## 8. Skill 공유 전 체크리스트

### 콘텐츠 품질

- [ ] Description이 구체적이고 트리거 키워드 포함
- [ ] SKILL.md가 500줄 이하
- [ ] 추가 상세 정보는 별도 파일에 (필요 시)
- [ ] 시간에 민감한 정보 없음
- [ ] 일관된 용어 사용
- [ ] 예제가 구체적이고 추상적이지 않음
- [ ] 파일 참조가 1단계 깊이
- [ ] 점진적 공개 적절히 사용

### 코드 및 스크립트 (해당 시)

- [ ] 스크립트가 오류를 명시적으로 처리
- [ ] 필요한 패키지가 지침에 명시됨
- [ ] Windows 스타일 경로 없음 (모두 forward slash)
- [ ] 중요 작업에 검증 단계 포함

### 테스트

- [ ] 최소 3개의 평가 케이스 생성
- [ ] Haiku, Sonnet, Opus에서 테스트
- [ ] 실제 사용 시나리오로 테스트
- [ ] 팀 피드백 반영

---

## 9. Quick Reference: Description 템플릿

```yaml
# 기본 템플릿
description: [동작 설명]. Use when [트리거 조건] or when the user mentions [키워드들].

# PDF 처리
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# 데이터베이스 쿼리
description: Query [데이터베이스] for [데이터 유형]. Use when [분석 유형] or querying [테이블명].

# 코드 작업
description: [작업 설명] in [언어/프레임워크]. Use when [트리거 조건] or when the user asks about [키워드들].
```

## 다음 단계

- [파일 구조](04-file-structure.md) - 복잡한 Skill 구성 방법
- [실전 예제](05-examples.md) - 전체 구현 예제 확인
