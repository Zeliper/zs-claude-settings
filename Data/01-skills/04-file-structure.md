# Skills 파일 구조

## 개요

Skill은 단일 파일로 간단하게 구성하거나, 복잡한 도메인 지식을 위해 다중 파일로 구성할 수 있습니다.

## 단일 파일 구조

가장 간단한 형태로, 대부분의 Skill에 적합합니다.

### 디렉토리 구조

```
skill-name/
└── SKILL.md
```

### 예제: Commit Helper

```yaml
---
name: generating-commit-messages
description: Generates clear commit messages from git diffs. Use when writing commit messages, creating commits, or reviewing staged changes.
---

# Generating Commit Messages

## Quick Start

```bash
git diff --staged
```

## 커밋 메시지 포맷

**Type**: feat, fix, chore, refactor, test, docs

**Summary**: 50자 이하, 현재 시제

**Body**: 무엇이 변경되었고 왜인지 설명 (어떻게가 아님)

## 예제

### Feature 추가
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware.
Allows users to authenticate with tokens instead of sessions.
```

### 버그 수정
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
to prevent timezone-related display issues.
```

## 베스트 프랙티스

- 동사로 시작: Add, Fix, Refactor, Update
- 가능하면 이슈 번호 참조
- why를 설명, how가 아님
- summary는 간결하게
```

---

## 다중 파일 구조

복잡한 도메인이나 대량의 참조 자료가 필요한 경우 사용합니다.

### 디렉토리 구조

```
pdf-processing/
├── SKILL.md          # 개요 및 네비게이션
├── FORMS.md          # 폼 작성 가이드
├── REFERENCE.md      # API 상세
└── scripts/
    ├── fill_form.py       # 유틸리티 스크립트
    ├── validate.py        # 검증 스크립트
    └── analyze_form.py    # 분석 스크립트
```

### 메인 SKILL.md 구성

````yaml
---
name: pdf-processing
description: Extract text, fill forms, merge PDFs. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
allowed-tools: Read, Bash(python:*)
---

# PDF Processing

## Quick Start

pdfplumber를 사용한 텍스트 추출:

```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## 추가 자료

- **폼 작성 가이드**: [FORMS.md](FORMS.md) 참조
- **API 레퍼런스**: [REFERENCE.md](REFERENCE.md) 참조
- **유틸리티 스크립트**: `python scripts/analyze_form.py input.pdf` 실행

## 요구사항

```bash
pip install pypdf pdfplumber
```
````

### 보조 파일: FORMS.md

```markdown
# 폼 작성 가이드

## fields.json 구조

```json
{
  "customer_name": "John Doe",
  "order_total": "$1,500.00",
  "date_signed": "2024-01-15"
}
```

## 일반적인 필드 유형

| 유형 | 설명 | 값 형식 |
|------|------|---------|
| Text | 단순 텍스트 입력 | 문자열 |
| Checkbox | True/False 값 | boolean |
| Date | 날짜 | YYYY-MM-DD |
| Signature | 특수 처리 필요 | 별도 처리 |

## 워크플로우

1. `python scripts/analyze_form.py input.pdf`로 폼 분석
2. `fields.json` 편집
3. `python scripts/validate.py fields.json`으로 검증
4. `python scripts/fill_form.py input.pdf fields.json output.pdf`로 채우기
```

---

## 점진적 공개(Progressive Disclosure) 패턴

### 패턴 1: 개요 + 상세 참조

메인 파일에는 핵심 지침만, 상세 내용은 별도 파일로:

```
explaining-code/
├── SKILL.md         # 개요와 기본 지침
├── DIAGRAMS.md      # ASCII 다이어그램 가이드
└── ANALOGIES.md     # 비유 예시 모음
```

**SKILL.md:**
```markdown
---
name: explaining-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works.
---

# 코드 설명하기

## 기본 원칙

1. **비유로 시작**: 일상적인 것에 비유
2. **다이어그램 그리기**: ASCII 아트로 흐름 표현
3. **단계별 설명**: 무슨 일이 일어나는지 차근차근
4. **주의점 강조**: 흔한 실수나 오해 언급

## 추가 자료

- 복잡한 다이어그램이 필요하면: [DIAGRAMS.md](DIAGRAMS.md)
- 비유 예시가 필요하면: [ANALOGIES.md](ANALOGIES.md)
```

### 패턴 2: 도메인별 구성

비즈니스 도메인별로 파일 분리:

```
bigquery-analysis/
├── SKILL.md
└── schemas/
    ├── finance.md     # 매출, ARR, 청구
    ├── sales.md       # 파이프라인, 기회
    ├── product.md     # API 사용량, 채택
    └── marketing.md   # 캠페인, 어트리뷰션
```

**SKILL.md:**
````yaml
---
name: bigquery-analysis
description: Query BigQuery datasets for finance, sales, and product metrics. Use when analyzing business data, creating reports, or querying BigQuery tables.
---

# BigQuery Analysis

## 사용 가능한 데이터셋

### Finance 메트릭
매출, ARR, 청구, 이탈 분석
[schemas/finance.md](schemas/finance.md) 참조

### Sales 메트릭
기회, 파이프라인, 전환율
[schemas/sales.md](schemas/sales.md) 참조

### Product 메트릭
API 사용량, 기능 채택, 성능
[schemas/product.md](schemas/product.md) 참조

### Marketing 메트릭
캠페인, 어트리뷰션
[schemas/marketing.md](schemas/marketing.md) 참조

## 공통 쿼리 패턴

테스트 데이터 필터링:

```sql
WHERE account_type != 'test'
  AND created_at >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)
```

## 빠른 검색

```bash
grep -i "revenue" schemas/finance.md
grep -i "pipeline" schemas/sales.md
```
````

### 패턴 3: 조건부 상세

특정 조건에서만 필요한 정보 분리:

```markdown
## 기본 문서 생성

docx-js를 사용한 문서 생성. [DOCX-JS.md](DOCX-JS.md) 참조.

## 변경 추적이 필요한 경우

법적 문서나 계약서의 변경 추적이 필요하면 [REDLINING.md](REDLINING.md) 참조.
```

---

## 스크립트 포함 구조

### 디렉토리 구조

```
data-processing/
├── SKILL.md
├── REFERENCE.md
└── scripts/
    ├── extract.py
    ├── transform.py
    ├── validate.py
    └── requirements.txt
```

### SKILL.md에서 스크립트 참조

````markdown
---
name: data-processing
description: Process and transform data files. Use when working with CSV, JSON, or Excel files.
allowed-tools: Read, Bash(python:*)
---

# 데이터 처리

## 설치

```bash
pip install -r scripts/requirements.txt
```

## 워크플로우

### 1. 데이터 추출

```bash
python scripts/extract.py input.csv --format json
```

### 2. 데이터 변환

```bash
python scripts/transform.py data.json --normalize
```

### 3. 검증

```bash
python scripts/validate.py output.json --schema schema.json
```

## 스크립트 옵션

| 스크립트 | 주요 옵션 |
|----------|----------|
| extract.py | `--format`, `--encoding` |
| transform.py | `--normalize`, `--filter` |
| validate.py | `--schema`, `--strict` |
````

### 스크립트 경로 규칙

```markdown
# 올바른 경로 (Unix 스타일)
python scripts/helper.py
reference/guide.md

# 잘못된 경로 (Windows 스타일 - 다른 OS에서 실패)
python scripts\helper.py
reference\guide.md
```

---

## 파일 구조 선택 가이드

### 단일 파일을 선택해야 할 때

- Skill 내용이 **300줄 이하**
- **단일 도메인**에 집중
- 참조 자료가 **필요 없거나 적음**
- **빠른 프로토타이핑** 단계

### 다중 파일을 선택해야 할 때

- 여러 **관련 기능**을 포함
- **도메인별 상세 정보**가 필요
- **스크립트나 도구**가 포함됨
- 팀에서 **다른 부분을 별도로 관리**해야 함
- **점진적 공개**로 컨텍스트 효율성을 높여야 함

### 구조 결정 플로우차트

```
Skill 내용이 300줄 이상인가?
├─ 아니오 → 단일 파일 구조
└─ 예 → 참조 자료가 여러 도메인인가?
         ├─ 아니오 → 메인 + 1-2개 참조 파일
         └─ 예 → 도메인별 하위 폴더 구조
```

---

## 파일 네이밍 컨벤션

### 디렉토리명

```
# 좋은 예
pdf-processing/
code-reviewing/
data-analysis/

# 나쁜 예
PDFProcessing/    # 대문자 사용
pdf_processing/   # 언더스코어 사용
pdf/              # 너무 모호
```

### 파일명

```
# 메인 파일 (필수)
SKILL.md          # 반드시 대문자

# 보조 파일
FORMS.md          # 대문자 권장
REFERENCE.md
README.md         # 추가 설명용

# 또는 소문자
forms.md
reference.md
```

### 스크립트 파일

```
scripts/
├── extract.py        # 동사형
├── transform.py
├── validate.py
└── requirements.txt  # 의존성 명시
```

---

## 실전 예제: 완전한 다중 파일 Skill

### 구조

```
excel-automation/
├── SKILL.md
├── FORMATTING.md
├── FORMULAS.md
├── CHARTS.md
└── scripts/
    ├── read_excel.py
    ├── write_excel.py
    ├── create_chart.py
    └── requirements.txt
```

### SKILL.md

````yaml
---
name: excel-automation
description: Automate Excel file operations including reading, writing, formatting, and charting. Use when working with Excel files, spreadsheets, or .xlsx documents.
allowed-tools: Read, Bash(python:*)
---

# Excel 자동화

## 설치

```bash
pip install -r scripts/requirements.txt
```

## Quick Start

### 읽기
```bash
python scripts/read_excel.py input.xlsx --sheet "Sheet1"
```

### 쓰기
```bash
python scripts/write_excel.py data.json output.xlsx
```

### 차트 생성
```bash
python scripts/create_chart.py input.xlsx --type bar --output chart.xlsx
```

## 상세 가이드

| 작업 | 참조 파일 |
|------|----------|
| 셀 서식 지정 | [FORMATTING.md](FORMATTING.md) |
| 수식 작성 | [FORMULAS.md](FORMULAS.md) |
| 차트 생성 | [CHARTS.md](CHARTS.md) |

## 주의사항

- `.xlsx` 형식만 지원 (`.xls` 미지원)
- 대용량 파일(100MB+)은 청크 처리 권장
- 매크로가 포함된 파일은 `.xlsm` 확장자 필요
````

## 다음 단계

- [실전 예제](05-examples.md) - 전체 구현 예제 확인
