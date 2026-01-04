# Skills 생성 방법

## 기본 Skill 생성 프로세스

### Step 1: Skill 디렉토리 생성

```bash
# Personal Skill (본인만 사용)
mkdir -p ~/.claude/skills/my-skill-name

# Project Skill (팀 공유)
mkdir -p .claude/skills/my-skill-name
```

### Step 2: SKILL.md 파일 작성

```bash
cat > ~/.claude/skills/my-skill-name/SKILL.md << 'EOF'
---
name: my-skill-name
description: Skill이 무엇을 하고 언제 사용하는지 설명. 사용자가 자연스럽게 말할 트리거 키워드를 포함.
---

# Skill 제목

## 지침

1. 첫 번째 단계
2. 두 번째 단계
3. 세 번째 단계

## 예제

```python
# 코드 예제
```

EOF
```

### Step 3: Claude Code 재시작

```bash
# Claude Code 종료 후 재시작
claude
```

### Step 4: Skill 확인

```
Claude Code 내에서:
> "사용 가능한 Skill이 뭐가 있어?"
```

## SKILL.md Frontmatter 상세

### 필수 필드

```yaml
---
name: skill-name              # 필수: 소문자, 숫자, 하이픈만
description: 설명 텍스트       # 필수: 최대 1024자
---
```

### 선택 필드

```yaml
---
name: skill-name
description: 설명 텍스트
allowed-tools: Read, Grep, Glob    # 선택: 허용할 도구 목록
model: claude-sonnet-4-20250514    # 선택: 특정 모델 지정
---
```

### 필드별 상세 설명

| 필드 | 필수 | 최대 길이 | 규칙 |
|------|------|----------|------|
| `name` | Yes | 64자 | 소문자, 숫자, 하이픈만. 디렉토리명과 일치 권장 |
| `description` | Yes | 1024자 | **3인칭**으로 작성. 무엇을, 언제 사용하는지 포함 |
| `allowed-tools` | No | - | 쉼표로 구분된 도구 목록 |
| `model` | No | - | 모델 ID (예: `claude-sonnet-4-20250514`) |

## Description 작성법

### 올바른 Description 패턴

```yaml
# 좋은 예: 무엇을 + 언제를 명확히
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# 좋은 예: 트리거 키워드 포함
description: Generates clear commit messages from git diffs. Use when writing commit messages, creating commits, or reviewing staged changes.

# 좋은 예: 도메인 특화 명시
description: Query BigQuery datasets for finance, sales, and product metrics. Use when analyzing business data, creating reports, or querying BigQuery tables.
```

### 나쁜 Description 패턴

```yaml
# 나쁜 예: 너무 모호함
description: Helps with documents

# 나쁜 예: 트리거 조건 없음
description: Processes data

# 나쁜 예: 1인칭 사용 (발견 어려움)
description: I help you work with PDFs
```

## allowed-tools 사용법

특정 도구만 허용하여 Skill의 동작을 제한할 수 있습니다.

```yaml
---
name: read-only-analyzer
description: Safely analyze files without making changes. Use for read-only file exploration.
allowed-tools: Read, Grep, Glob
---

# Read-Only Analyzer

이 Skill은 파일을 읽기만 하고 수정하지 않습니다.

## 사용 가능한 도구
- **Read**: 파일 내용 읽기
- **Grep**: 파일 내용 검색
- **Glob**: 패턴으로 파일 찾기
```

## Skill 본문 작성 구조

### 권장 구조

```markdown
---
name: my-skill
description: ...
---

# Skill 제목

## Quick Start
가장 일반적인 사용법을 간결하게

## 상세 지침
1. 단계별 지침
2. 각 단계 설명
3. 주의사항 포함

## 예제
실제 사용 예제 코드

## 참고 자료
- [관련 파일 링크](./REFERENCE.md)
- [추가 가이드](./FORMS.md)

## 요구사항
```bash
pip install required-package
```
```

### 실제 예제: Commit Helper Skill

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

## Skill 테스트 방법

### 1. 로드 확인

```
> "사용 가능한 Skill 목록을 보여줘"
```

Claude가 새로 만든 Skill을 목록에 표시해야 합니다.

### 2. 트리거 테스트

Description에 포함된 키워드로 질문:

```
> "커밋 메시지 작성해줘"
```

Claude가 해당 Skill 사용을 요청해야 합니다.

### 3. 동작 검증

Skill이 의도한 대로 지침을 따르는지 확인:

```
> "staged 변경사항에 대한 커밋 메시지 만들어줘"
```

## Skill 수정 및 삭제

### 수정

```bash
# SKILL.md 파일 직접 편집
vim ~/.claude/skills/my-skill/SKILL.md

# Claude Code 재시작
claude
```

### 삭제

```bash
# Skill 디렉토리 삭제
rm -rf ~/.claude/skills/my-skill

# Claude Code 재시작
claude
```

## 트러블슈팅

### Skill이 로드되지 않음

1. **경로 확인**: `~/.claude/skills/skill-name/SKILL.md` 구조인지
2. **파일명 확인**: 정확히 `SKILL.md` (대소문자 구분)
3. **YAML 구문**: frontmatter가 첫 줄에서 시작하는지
4. **재시작**: Claude Code를 재시작했는지

### YAML 구문 오류

```yaml
# 잘못됨: 콜론 뒤 공백 없음
name:skill-name

# 잘못됨: 탭 사용
name: skill-name
	description: ...

# 올바름
name: skill-name
description: This is the description
```

### Skill이 트리거되지 않음

1. Description이 충분히 구체적인지 확인
2. 사용자가 사용할 키워드가 포함되어 있는지 확인
3. 3인칭으로 작성되었는지 확인

## 다음 단계

- [베스트 프랙티스](03-best-practices.md) - 효과적인 Skill 작성 요령
- [파일 구조](04-file-structure.md) - 복잡한 Skill 구성 방법
