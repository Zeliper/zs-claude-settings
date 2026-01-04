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

> **중요: 모든 설정 생성은 반드시 Plan 모드에서 실행되어야 합니다.**

Claude Code에게 이 저장소의 `Data/README.md`를 읽고 설정을 생성하도록 요청합니다.

### 프로젝트 전체 분석 및 설정 (권장)

프로젝트를 분석하여 필요한 모든 설정을 한 번에 생성하려면:

```
https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 현재 프로젝트를 분석해서
필요한 Skills, Agents, MCP를 생성하고 CLAUDE.md를 생성해줘.
그다음 Workflow를 설명한 파일을 ./Workflow.md로 생성하고싶어.
```

이 프롬프트를 사용하면 Claude Code가:
1. 프로젝트 구조와 기술 스택을 분석합니다
2. **Plan 모드에서** 각 설정에 필요한 정보를 질문합니다
3. 사용자 응답을 바탕으로 최적의 설정을 계획합니다
4. 승인 후 모든 설정 파일을 생성합니다

### 개별 설정 생성

특정 설정만 생성하려면:

```
https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 [원하는 설정 유형]을 만들어줘
```

**예시 프롬프트:**
- "https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 로그 분석 MCP 서버를 만들어줘"
- "https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 코드 리뷰 Agent를 만들어줘"
- "https://github.com/Zeliper/zs-claude-settings 의 Data/README.md를 읽고 이 프로젝트의 CLAUDE.md를 만들어줘"

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
