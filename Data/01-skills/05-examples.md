# Skills 실전 예제

## 예제 1: Commit Message Generator

### 목적
Git diff를 분석하여 팀 컨벤션에 맞는 커밋 메시지 생성

### 파일 구조

```
generating-commit-messages/
└── SKILL.md
```

### SKILL.md 전체 코드

```yaml
---
name: generating-commit-messages
description: Generates clear commit messages from git diffs. Use when writing commit messages, creating commits, or reviewing staged changes.
---

# 커밋 메시지 생성

## Quick Start

staged 변경사항 확인:

```bash
git diff --staged
```

## 커밋 메시지 포맷

### 구조

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 종류

| Type | 설명 | 예시 |
|------|------|------|
| feat | 새로운 기능 | feat(auth): add JWT authentication |
| fix | 버그 수정 | fix(api): resolve null pointer exception |
| refactor | 코드 리팩토링 | refactor(user): simplify validation logic |
| docs | 문서 변경 | docs(readme): update installation guide |
| test | 테스트 추가/수정 | test(auth): add login unit tests |
| chore | 빌드/설정 변경 | chore(deps): update dependencies |

### Subject 규칙

- **50자 이하**로 작성
- **현재 시제** 사용 (add, not added)
- **첫 글자 소문자**
- **마침표 없음**

### Body 규칙

- **무엇**이 변경되었는지 설명
- **왜** 변경했는지 설명
- 어떻게는 코드가 설명함

## 실전 예제

### Feature 추가

```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware.
Allows users to authenticate with tokens instead of sessions.

Closes #123
```

### 버그 수정

```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
to prevent timezone-related display issues.

Fixes #456
```

### 리팩토링

```
refactor(user): extract validation logic to separate module

Move email and password validation to shared validators.
Improves testability and reduces code duplication.
```

## 워크플로우

1. `git diff --staged`로 변경사항 확인
2. 변경의 **주요 목적** 파악
3. 적절한 **type** 선택
4. **scope** 결정 (변경된 모듈/기능)
5. **subject** 작성 (50자 이하)
6. 필요시 **body** 추가

## 체크리스트

- [ ] Type이 변경 내용과 일치하는가?
- [ ] Subject가 50자 이하인가?
- [ ] 현재 시제를 사용했는가?
- [ ] Body가 왜(why)를 설명하는가?
- [ ] 관련 이슈가 있다면 참조했는가?
```

---

## 예제 2: PDF Form Processor

### 목적
PDF 폼 분석, 필드 추출, 폼 채우기 자동화

### 파일 구조

```
pdf-form-processing/
├── SKILL.md
├── FORMS.md
├── REFERENCE.md
└── scripts/
    ├── analyze_form.py
    ├── validate_fields.py
    ├── fill_form.py
    └── requirements.txt
```

### SKILL.md

````yaml
---
name: pdf-form-processing
description: Extract form fields, fill PDF forms, validate entries. Use when working with PDF forms or form filling tasks.
allowed-tools: Read, Bash(python:*)
---

# PDF 폼 처리

## 설치

```bash
pip install -r scripts/requirements.txt
```

## Quick Start

### Step 1: 폼 분석

```bash
python scripts/analyze_form.py input.pdf
```

결과: `fields.json` 파일 생성 (모든 폼 필드 정보)

### Step 2: 필드 편집

`fields.json`을 열어 값 입력:

```json
{
  "customer_name": "홍길동",
  "order_date": "2024-01-15",
  "total_amount": "150,000"
}
```

### Step 3: 검증

```bash
python scripts/validate_fields.py fields.json
```

### Step 4: 폼 채우기

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```

## 검증 워크플로우

1. 폼 구조 분석
2. 필드 매핑 생성 (`fields.json` 편집)
3. **검증**: `python scripts/validate_fields.py fields.json`
4. 폼 채우기
5. 출력 확인

## 상세 가이드

- 폼 채우기 상세: [FORMS.md](FORMS.md)
- API 레퍼런스: [REFERENCE.md](REFERENCE.md)

## 주의사항

- PDF 버전 1.4 이상 권장
- 암호화된 PDF는 비밀번호 필요
- 플래튼된(flattened) 폼은 편집 불가
````

### FORMS.md

```markdown
# 폼 채우기 상세 가이드

## fields.json 구조

### 기본 형식

```json
{
  "field_name": "value",
  "another_field": "another_value"
}
```

### 필드 유형별 값 형식

| 유형 | 형식 | 예시 |
|------|------|------|
| 텍스트 | 문자열 | `"홍길동"` |
| 숫자 | 문자열(숫자) | `"150000"` |
| 날짜 | YYYY-MM-DD | `"2024-01-15"` |
| 체크박스 | "Yes"/"Off" | `"Yes"` |
| 라디오 | 선택 값 | `"option1"` |
| 드롭다운 | 선택 값 | `"Korea"` |

### 복잡한 폼 예시

```json
{
  "personal_info": {
    "name": "홍길동",
    "email": "hong@example.com",
    "phone": "010-1234-5678"
  },
  "address": {
    "street": "강남대로 123",
    "city": "서울",
    "zip": "06000"
  },
  "agreement": "Yes",
  "payment_method": "credit_card"
}
```

## 일반적인 문제 해결

### 필드명을 모를 때

```bash
python scripts/analyze_form.py form.pdf --verbose
```

### 필드 값이 적용되지 않을 때

1. 필드명 대소문자 확인
2. 폼이 플래튼되지 않았는지 확인
3. PDF 버전 확인

### 한글이 깨질 때

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf --font "NanumGothic"
```
```

### scripts/requirements.txt

```
pypdf>=3.0.0
pdfplumber>=0.9.0
reportlab>=4.0.0
```

---

## 예제 3: BigQuery Data Analyst

### 목적
비즈니스 데이터 분석을 위한 BigQuery 스키마 레퍼런스 제공

### 파일 구조

```
bigquery-analysis/
├── SKILL.md
└── schemas/
    ├── finance.md
    ├── sales.md
    ├── product.md
    └── marketing.md
```

### SKILL.md

````yaml
---
name: bigquery-analysis
description: Query BigQuery datasets for finance, sales, and product metrics. Use when analyzing business data, creating reports, or querying BigQuery tables.
---

# BigQuery 데이터 분석

## 데이터셋 개요

| 도메인 | 주요 테이블 | 참조 |
|--------|------------|------|
| Finance | revenue, billing, churn | [schemas/finance.md](schemas/finance.md) |
| Sales | opportunities, pipeline | [schemas/sales.md](schemas/sales.md) |
| Product | api_usage, feature_adoption | [schemas/product.md](schemas/product.md) |
| Marketing | campaigns, attribution | [schemas/marketing.md](schemas/marketing.md) |

## 공통 쿼리 패턴

### 테스트 데이터 필터링 (필수)

```sql
WHERE account_type != 'test'
  AND created_at >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)
```

### 날짜 범위 설정

```sql
-- 이번 달
WHERE DATE_TRUNC(created_at, MONTH) = DATE_TRUNC(CURRENT_DATE(), MONTH)

-- 최근 7일
WHERE created_at >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)

-- 분기별
WHERE DATE_TRUNC(created_at, QUARTER) = DATE_TRUNC(CURRENT_DATE(), QUARTER)
```

### 집계 패턴

```sql
SELECT
  DATE_TRUNC(created_at, MONTH) as month,
  COUNT(DISTINCT user_id) as unique_users,
  SUM(amount) as total_amount
FROM dataset.table
GROUP BY 1
ORDER BY 1 DESC
```

## 빠른 스키마 검색

```bash
# 특정 컬럼 찾기
grep -i "revenue" schemas/finance.md

# 테이블 찾기
grep -i "pipeline" schemas/sales.md
```

## 주의사항

- 항상 테스트 계정 필터링
- 대용량 쿼리 시 LIMIT 사용
- 비용이 많이 드는 쿼리는 DRY RUN 먼저
````

### schemas/finance.md

```markdown
# Finance 스키마

## revenue 테이블

```sql
dataset.revenue
```

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | STRING | 고유 식별자 |
| account_id | STRING | 계정 ID |
| amount | FLOAT64 | 금액 (USD) |
| currency | STRING | 원본 통화 |
| created_at | TIMESTAMP | 생성 시간 |
| account_type | STRING | 'enterprise', 'startup', 'test' |

### 일반적인 쿼리

#### 월별 매출
```sql
SELECT
  DATE_TRUNC(created_at, MONTH) as month,
  SUM(amount) as monthly_revenue
FROM dataset.revenue
WHERE account_type != 'test'
GROUP BY 1
ORDER BY 1 DESC
```

#### ARR 계산
```sql
SELECT
  SUM(amount) * 12 as arr
FROM dataset.revenue
WHERE account_type != 'test'
  AND DATE_TRUNC(created_at, MONTH) = DATE_TRUNC(CURRENT_DATE(), MONTH)
```

## billing 테이블

```sql
dataset.billing
```

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | STRING | 청구 ID |
| account_id | STRING | 계정 ID |
| invoice_amount | FLOAT64 | 청구 금액 |
| status | STRING | 'paid', 'pending', 'failed' |
| due_date | DATE | 납부 기한 |

## churn 테이블

```sql
dataset.churn
```

| 컬럼 | 타입 | 설명 |
|------|------|------|
| account_id | STRING | 계정 ID |
| churn_date | DATE | 이탈 일자 |
| reason | STRING | 이탈 사유 |
| mrr_lost | FLOAT64 | 손실 MRR |
```

---

## 예제 4: Code Explainer

### 목적
코드를 시각적 다이어그램과 비유를 사용하여 설명

### 파일 구조

```
explaining-code/
├── SKILL.md
├── DIAGRAMS.md
└── ANALOGIES.md
```

### SKILL.md

```yaml
---
name: explaining-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
---

# 코드 설명하기

## 설명 원칙

코드를 설명할 때 항상 다음을 포함:

1. **비유로 시작**: 일상적인 것에 비유
2. **다이어그램 그리기**: ASCII 아트로 흐름/구조 표현
3. **단계별 진행**: 무슨 일이 일어나는지 차근차근
4. **주의점 강조**: 흔한 실수나 오해 언급

## 설명 예시

### Promise 설명

**비유**: 음식점 진동벨

> Promise는 음식점의 진동벨과 같습니다. 주문하면(Promise 생성)
> 벨을 받고(pending), 음식이 준비되면 벨이 울리거나(resolved)
> 재료가 떨어지면 취소됩니다(rejected).

**다이어그램**:
```
┌─────────────┐
│   주문      │ ← new Promise()
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  대기 중    │ ← pending
└──────┬──────┘
       │
   ┌───┴───┐
   ▼       ▼
┌─────┐ ┌─────┐
│완료 │ │거부 │
│.then│ │.catch│
└─────┘ └─────┘
```

## 추가 자료

- 복잡한 다이어그램: [DIAGRAMS.md](DIAGRAMS.md)
- 비유 모음: [ANALOGIES.md](ANALOGIES.md)
```

---

## 예제 5: API Integration Helper

### 목적
외부 API 연동 코드 작성 지원

### 파일 구조

```
api-integration/
├── SKILL.md
└── templates/
    ├── rest_client.py
    ├── graphql_client.py
    └── webhook_handler.py
```

### SKILL.md

````yaml
---
name: api-integration
description: Generate API integration code for REST, GraphQL, and webhooks. Use when integrating external APIs or building API clients.
---

# API 통합 가이드

## REST API 클라이언트

### 기본 패턴

```python
import requests
from typing import Optional, Dict, Any

class APIClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        })

    def get(self, endpoint: str, params: Optional[Dict] = None) -> Dict[Any, Any]:
        response = self.session.get(
            f'{self.base_url}/{endpoint}',
            params=params
        )
        response.raise_for_status()
        return response.json()

    def post(self, endpoint: str, data: Dict) -> Dict[Any, Any]:
        response = self.session.post(
            f'{self.base_url}/{endpoint}',
            json=data
        )
        response.raise_for_status()
        return response.json()
```

### 에러 처리

```python
from requests.exceptions import HTTPError, Timeout, ConnectionError

try:
    result = client.get('users/123')
except HTTPError as e:
    if e.response.status_code == 404:
        print("User not found")
    elif e.response.status_code == 401:
        print("Authentication failed")
    else:
        raise
except Timeout:
    print("Request timed out")
except ConnectionError:
    print("Connection failed")
```

### 재시도 로직

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
def fetch_with_retry(client, endpoint):
    return client.get(endpoint)
```

## 템플릿 파일

| 용도 | 파일 |
|------|------|
| REST 클라이언트 | [templates/rest_client.py](templates/rest_client.py) |
| GraphQL 클라이언트 | [templates/graphql_client.py](templates/graphql_client.py) |
| Webhook 핸들러 | [templates/webhook_handler.py](templates/webhook_handler.py) |

## 베스트 프랙티스

1. **환경 변수**로 API 키 관리
2. **타임아웃** 항상 설정
3. **재시도** 로직으로 일시적 오류 처리
4. **로깅**으로 디버깅 용이하게
5. **Rate limiting** 준수
````

---

## Skill 개발 체크리스트

각 예제를 만들 때 확인:

- [ ] **Description**이 명확하고 트리거 키워드 포함
- [ ] **Quick Start** 섹션으로 빠른 시작 가능
- [ ] **예제**가 복사-붙여넣기로 바로 실행 가능
- [ ] **주의사항**이 명시됨
- [ ] 필요시 **보조 파일**로 상세 정보 분리
- [ ] **모든 모델**에서 테스트 완료
