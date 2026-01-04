# zs-claude-settings

Claude Code 확장 기능(Skills, Agents, MCP, Workflows, CLAUDE.md) 설정 가이드 문서 모음입니다.

## 프로젝트 소개

이 저장소는 Claude Code의 확장 기능을 프로젝트에 통합하기 위한 종합 문서를 제공합니다. Claude Code가 이 문서를 읽고 Plan 모드에서 사용자와 상호작용하며 필요한 설정을 자동으로 생성합니다.

### 지원하는 설정 유형

| 유형 | 설명 | 문서 위치 |
|------|------|-----------|
| **Skills** | Claude에게 전문 지식을 주입하는 마크다운 파일 | `Data/01-skills/` |
| **Agents** | 독립적인 컨텍스트에서 작업을 수행하는 특화된 AI 어시스턴트 | `Data/02-agents/` |
| **MCP** | 외부 시스템과 연결하는 Model Context Protocol 서버 | `Data/03-mcp/` |
| **Workflows** | 개발 프로세스 가이드 | `Data/04-workflows/` |
| **CLAUDE.md** | 프로젝트 레벨 컨텍스트 및 가이드라인 | `Data/05-claude-md/` |

---

## 사용 방법

Claude Code에게 이 저장소의 `Data/README.md`를 읽고 설정을 생성하도록 요청합니다:

```
https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 [원하는 설정 유형]을 만들어줘
```

**예시 프롬프트:**
- "https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 로그 분석 MCP 서버를 만들어줘"
- "https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 코드 리뷰 Agent를 만들어줘"
- "https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 이 프로젝트의 CLAUDE.md를 만들어줘"

---

## Claude Code를 위한 지시사항

> **이 섹션은 Claude Code가 읽고 따라야 하는 지침입니다.**

### 설정 생성 워크플로우

사용자가 이 저장소의 문서를 참조하여 설정 생성을 요청하면 다음 단계를 따르세요:

1. **Data/README.md 읽기**: 먼저 `https://raw.githubusercontent.com/Zeliper/zs-claude-settings/main/Data/README.md`를 읽어 전체 가이드를 파악합니다
2. **관련 문서 읽기**: 요청된 설정 유형에 맞는 세부 문서를 GitHub에서 읽습니다
3. **Plan 모드 진입**: 설정 생성 계획을 수립합니다
4. **필수 입력값 수집**: 사용자에게 설정에 필요한 정보를 질문합니다
5. **설정 생성**: 수집한 정보를 바탕으로 설정 파일을 생성합니다

### GitHub 문서 URL 형식

```
https://raw.githubusercontent.com/Zeliper/zs-claude-settings/main/Data/[폴더]/[파일명]
```

**예시:**
- Skills 개요: `https://raw.githubusercontent.com/Zeliper/zs-claude-settings/main/Data/01-skills/01-overview.md`
- MCP 생성: `https://raw.githubusercontent.com/Zeliper/zs-claude-settings/main/Data/03-mcp/02-creating-servers.md`

### 설정 유형별 필수 질문

#### Skills 생성 시
- 스킬의 이름과 목적은 무엇인가요?
- 어떤 파일 유형이나 도메인을 대상으로 하나요?
- 스킬이 활성화되어야 하는 조건(트리거)은 무엇인가요?
- 포함할 지식 소스나 참조 문서가 있나요?

#### Agents 생성 시
- Agent의 역할과 전문 분야는 무엇인가요?
- 어떤 도구에 접근해야 하나요? (Read, Write, Bash 등)
- 시스템 프롬프트에 포함할 특별한 지침이 있나요?
- 병렬 처리가 필요한가요?

#### MCP 서버 생성 시
- 연결할 외부 시스템은 무엇인가요?
- 전송 방식은 무엇인가요? (HTTP, SSE, Stdio)
- 인증 방식은 무엇인가요? (OAuth, API Key, 없음)
- 노출할 리소스/도구는 무엇인가요?
- **구체적인 데이터 입력값**:
  - 로그 분석: 로그 폴더 경로, 로그 형식
  - 데이터베이스: 연결 문자열, 테이블 목록
  - API 연동: 엔드포인트 URL, 헤더 정보

#### Workflows 생성 시
- 어떤 유형의 워크플로우인가요? (Skill, Agent, MCP 개발)
- 프로젝트 특화 단계가 있나요?
- 기존 프로세스와의 통합이 필요한가요?

#### CLAUDE.md 생성 시
- 프로젝트의 기술 스택은 무엇인가요?
- 코딩 컨벤션이나 스타일 가이드가 있나요?
- 팀 가이드라인이나 규칙이 있나요?
- 프로젝트 아키텍처 개요를 알려주세요

### 중요 원칙

1. **항상 질문하기**: 설정에 필요한 정보가 불명확하면 추측하지 말고 사용자에게 질문하세요
2. **문서 참조하기**: `Data/` 폴더의 문서를 읽고 베스트 프랙티스를 따르세요
3. **점진적 생성**: 한 번에 모든 것을 만들지 말고, 핵심 기능부터 시작하세요
4. **사용자 확인**: 생성 전에 계획을 사용자에게 확인받으세요

---

## 문서 구조

```
zs-claude-settings/
├── README.md                  # 이 파일 (한국어)
├── index.html                 # 대화형 문서 포털
└── Data/
    ├── README.md              # 기술 가이드 (영어)
    ├── 01-skills/             # Skills 설정 가이드
    │   ├── 01-overview.md
    │   ├── 02-creating-skills.md
    │   ├── 03-best-practices.md
    │   ├── 04-file-structure.md
    │   └── 05-examples.md
    ├── 02-agents/             # Agents 설정 가이드
    │   ├── 01-overview.md
    │   ├── 02-sub-agents.md
    │   ├── 03-parallel-processing.md
    │   └── 04-best-practices.md
    ├── 03-mcp/                # MCP 서버 가이드
    │   ├── 01-overview.md
    │   ├── 02-creating-servers.md
    │   ├── 03-configuration.md
    │   └── 04-examples.md
    ├── 04-workflows/          # 개발 워크플로우 가이드
    │   ├── 01-skill-workflow.md
    │   ├── 02-agent-workflow.md
    │   └── 03-mcp-workflow.md
    └── 05-claude-md/          # CLAUDE.md 가이드
        ├── 01-overview.md
        ├── 02-creating-claude-md.md
        ├── 03-best-practices.md
        └── 04-examples.md
```

## 대화형 문서 포털

`index.html` 파일을 브라우저에서 열면 대화형 문서 포털을 사용할 수 있습니다:
- 다크 테마 UI
- 사이드바 네비게이션
- 전체 텍스트 검색
- 코드 하이라이팅

---

## 라이선스

MIT License
