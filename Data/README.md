# Claude Code Configuration Guide

This folder contains comprehensive documentation for Claude Code's extensibility features. Claude Code should read these documents when helping users create or merge configurations into their projects.

## Purpose

When a user requests to create Skills, Agents, MCP servers, Workflows, or CLAUDE.md files, Claude Code should:

1. **Read** the relevant documentation from this folder
2. **Enter Plan mode** to design the configuration
3. **Gather required inputs** from the user through questions
4. **Generate** the configuration based on user responses

---

## Plan Mode Integration Workflow

```
User Request → Read Documentation → Enter Plan Mode → Ask Questions → Generate Config
```

### Step-by-Step Process

1. **Identify Configuration Type**: Determine which type the user wants (Skills, Agents, MCP, etc.)
2. **Read Relevant Docs**: Navigate to the appropriate subfolder and read the documentation
3. **Enter Plan Mode**: Design the implementation approach
4. **Gather User Inputs**: Ask the required questions listed below for each type
5. **Confirm Plan**: Present the plan to the user for approval
6. **Generate Configuration**: Create the files based on collected information

---

## Configuration Types & Required User Inputs

### Skills (`01-skills/`)

Skills are Markdown files that inject specialized knowledge into Claude for specific domains or tasks.

**Required Questions to Ask:**

| Question | Purpose |
|----------|---------|
| What is the skill's name and purpose? | Define the skill's identity |
| What file types or domains does it target? | Scope the skill's applicability |
| When should this skill activate? (triggers) | Define activation conditions |
| What knowledge sources should be included? | Gather reference materials |
| Should it be project-level or user-level? | Determine file location |

**Output Location:**
- Project: `.claude/skills/SKILL_NAME.md`
- User: `~/.claude/skills/SKILL_NAME.md`

**Documentation:** [01-skills/01-overview.md](01-skills/01-overview.md)

---

### Agents (`02-agents/`)

Agents are specialized AI assistants that operate in isolated context windows for independent tasks.

**Required Questions to Ask:**

| Question | Purpose |
|----------|---------|
| What is the agent's role and specialization? | Define agent identity |
| What tools should the agent access? | Configure tool permissions |
| What specific instructions should be in the system prompt? | Customize behavior |
| Should it support parallel processing? | Configure concurrency |
| What naming convention for the agent file? | File naming |

**Output Location:**
- Project: `.claude/agents/AGENT_NAME.md`
- User: `~/.claude/agents/AGENT_NAME.md`

**Documentation:** [02-agents/01-overview.md](02-agents/01-overview.md)

---

### MCP Servers (`03-mcp/`)

MCP (Model Context Protocol) servers connect Claude to external systems through a standardized protocol.

**Required Questions to Ask:**

| Question | Purpose |
|----------|---------|
| What external system should be connected? | Identify integration target |
| What transport type? (HTTP/SSE/Stdio) | Configure connection method |
| What authentication method? (OAuth/API Key/None) | Security configuration |
| What resources/tools should be exposed? | Define capabilities |

**Specific Data Inputs by Use Case:**

| Use Case | Required Inputs |
|----------|-----------------|
| Log Analysis | Log folder path, log format, analysis features needed |
| Database | Connection string, database type, table/collection list |
| API Integration | Endpoint URLs, headers, request/response formats |
| File System | Root directory, allowed operations, file patterns |
| Issue Tracker | Project ID, authentication token, workflow states |

**Output Location:**
- Project config: `.mcp.json`
- Server code: Custom location based on project structure

**Documentation:** [03-mcp/01-overview.md](03-mcp/01-overview.md)

---

### Workflows (`04-workflows/`)

Workflows are development process guides for creating each configuration type.

**Required Questions to Ask:**

| Question | Purpose |
|----------|---------|
| What type of workflow? (Skill/Agent/MCP development) | Identify workflow category |
| Are there project-specific phases to include? | Customize workflow steps |
| How should it integrate with existing processes? | Ensure compatibility |
| What testing requirements exist? | Include validation steps |

**Documentation:** [04-workflows/01-skill-workflow.md](04-workflows/01-skill-workflow.md)

---

### CLAUDE.md (`05-claude-md/`)

CLAUDE.md provides project-level context and guidelines that Claude reads at session start.

**Required Questions to Ask:**

| Question | Purpose |
|----------|---------|
| What is the project's tech stack? | Document technologies |
| What coding conventions/style guides exist? | Capture standards |
| What team guidelines or rules apply? | Document processes |
| What is the project architecture overview? | Explain structure |
| What common tasks does the team perform? | Add helpful commands |

**Output Location:**
- Project root: `./CLAUDE.md`
- User level: `~/.claude/CLAUDE.md`

**Documentation:** [05-claude-md/01-overview.md](05-claude-md/01-overview.md)

---

## Example Prompts for Users

Users can use these prompts to request configuration creation:

### Skills
```
Read the documentation in .claude-settings/Data/01-skills and help me create
a skill for [DOMAIN/PURPOSE].
```

### Agents
```
Using .claude-settings/Data/02-agents as reference, create an agent
specialized for [ROLE/TASK].
```

### MCP Servers
```
Reference .claude-settings/Data/03-mcp to build an MCP server that
connects to [EXTERNAL SYSTEM] for [PURPOSE].
```

### CLAUDE.md
```
Based on .claude-settings/Data/05-claude-md, create a CLAUDE.md file
for this project that covers [ASPECTS].
```

---

## Documentation Reference

| Folder | Description | Key Documents |
|--------|-------------|---------------|
| `01-skills/` | Skill creation guides | overview, creating, best-practices, examples |
| `02-agents/` | Agent configuration | overview, sub-agents, parallel-processing |
| `03-mcp/` | MCP server development | overview, creating-servers, configuration |
| `04-workflows/` | Development processes | skill-workflow, agent-workflow, mcp-workflow |
| `05-claude-md/` | Project context files | overview, creating, best-practices, examples |

---

## Critical Principles for Claude Code

1. **Always Ask First**: Never assume values for required inputs. Ask the user explicitly.

2. **Read Before Creating**: Always read the relevant documentation before generating configurations.

3. **Confirm Plans**: Present the configuration plan to the user before creating files.

4. **Start Simple**: Create minimal viable configurations first, then iterate.

5. **Follow Best Practices**: Apply the best practices documented in each section.

6. **Respect Project Structure**: Place files in appropriate locations based on project-level vs user-level scope.

---

## File Structure Quick Reference

```
Project Root/
├── .claude/
│   ├── skills/           # Project-level skills
│   │   └── *.md
│   └── agents/           # Project-level agents
│       └── *.md
├── .mcp.json             # MCP server configurations
└── CLAUDE.md             # Project context file

User Home/
└── ~/.claude/
    ├── skills/           # User-level skills
    ├── agents/           # User-level agents
    └── CLAUDE.md         # User-level context
```
