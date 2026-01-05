# Oh-My-OpenCode 기능 관계도

**Generated:** 2026-01-05
**Branch:** feature/analyze-features
**Commit:** 65b00c9

---

## 전체 아키텍처

```mermaid
graph TB
    subgraph UserLayer[사용자 계층]
        User[사용자]
        CLI[OpenCode CLI]
    end

    subgraph PluginLayer[플러그인 계층]
        Plugin[Oh-My-OpenCode Plugin]
        Config[Config Handler]
        Auth[Google Antigravity Auth]
        BackgrdMgr[Background Manager]
        ContextColl[Context Collector]
    end

    subgraph HooksLayer[Hooks 계층 - 22개]
        TodoCont[todo-continuation-enforcer]
        SessionRec[session-recovery]
        CtxWinReco[anthropic-context-window-limit-recovery]
        ThinkMode[think-mode]
        RalphLoop[ralph-loop]
        Keyword[keyword-detector]
        Rules[rules-injector]
        AutoSlash[auto-slash-command]
        EditReco[edit-error-recovery]
        OtherHooks[나머지 13개 훅들]
    end

    subgraph ToolsLayer[Tools 계층 - 20+개]
        LSP[LSP Tools - 11개]
        AST[AST-Grep]
        FileSearch[grep, glob]
        SessionOps[session_*]
        BackgroundTools[background_task, background_output, background_cancel]
        SkillTools[skill, skill_mcp]
        AgentTool[call_omo_agent]
        LookAt[look_at]
        Bash[interactive_bash]
        Slash[slashcommand]
    end

    subgraph FeaturesLayer[Features 계층 - 13개]
        AgentLoaders[에이전트 로더]
        CmdLoaders[명령어 로더]
        McpLoaders[MCP 로더]
        PluginLoader[플러그인 로더]
        SkillLoader[스킬 로더]
        SkillMCP[스킬 MCP 관리자]
        BackgroundAgent[백그라운드 에이전트]
        SessionState[세션 상태]
        MsgInjector[메시지 인젝터]
        CtxInjector[컨텍스트 인젝터]
        BuiltinCmds[내장 명령어]
        BuiltinSkills[내장 스킬]
    end

    subgraph AgentsLayer[Agents 계층 - 7개]
        Sisyphus[Sisyphus - 오케스트레이터]
        Oracle[oracle - 어드바이저]
        Librarian[librarian - 리서치]
        Explore[explore - 검색]
        Frontend[frontend-ui-ux-engineer]
        DocWriter[document-writer]
        MultiLooker[multimodal-looker]
    end

    subgraph SharedLayer[Shared 계층 - 21개]
        Frontmatter[frontmatter]
        ModelSan[model-sanitizer]
        DeepMerge[deep-merge]
        DynamicTrunc[dynamic-truncator]
        FileUtils[file-utils]
        JSONC[jsonc-parser]
        Logger[logger]
        PathUtils[경로 유틸리티]
        Others[나머지 12개]
    end

    subgraph ExternalLayer[외부 계층]
        ClaudeCode[Claude Code<br/>~/.claude/*]
        OpenCodeConfig[OpenCode Config<br/>~/.config/opencode/*]
        Project[프로젝트<br/>.claude/*, .opencode/*]
        LSPServers[LSP 서버들]
        MCPServers[MCP 서버들]
    end

    User --> CLI
    CLI --> Plugin

    Config --> Plugin
    Auth --> Plugin
    BackgrdMgr --> Plugin
    ContextColl --> Plugin

    Plugin --> HooksLayer
    Plugin --> ToolsLayer
    Plugin --> AgentsLayer

    HooksLayer --> FeaturesLayer
    ToolsLayer --> FeaturesLayer
    AgentsLayer --> FeaturesLayer

    FeaturesLayer --> SharedLayer
    FeaturesLayer --> ExternalLayer

    AgentsLayer --> SharedLayer
    ToolsLayer --> SharedLayer
    HooksLayer --> SharedLayer

    style Plugin fill:#00CED1
    style Sisyphus fill:#e1f5e1
    style Oracle fill:#e1f5e1
    style Librarian fill:#e1f5e1
    style Explore fill:#e1f5e1
    style LSP fill:#e1f5ff
    style AST fill:#e1f5ff
    style BackgrdMgr fill:#ffe1e1
    style ContextColl fill:#ffe1e1
    style SharedLayer fill:#fff4e1
```

---

## Hooks 기능 관계

```mermaid
graph TB
    subgraph Events[OpenCode 이벤트]
        ChatMsg[chat.message]
        MsgsTrans[experimental.chat.messages.transform]
        ConfigEv[config]
        EventEv[event]
        ToolBefore[tool.execute.before]
        ToolAfter[tool.execute.after]
    end

    subgraph Hooks[훅들]
        TodoHook[todo-continuation-enforcer]
        CtxWinHook[context-window-monitor]
        SessionRecoHook[session-recovery]
        ThinkHook[think-mode]
        RalphHook[ralph-loop]
        KeywordHook[keyword-detector]
        AutoSlashHook[auto-slash-command]
        EditRecoHook[edit-error-recovery]
    end

    subgraph Outputs[처리 결과]
        ModifiedInput[수정된 입력]
        Blocked[차단됨]
        SyntheticMsg[합성 메시지]
        RecoveryPrompt[복구 프롬프트]
        Countdown[카운트다운 토스트]
    end

    ChatMsg --> TodoHook
    ChatMsg --> RalphHook
    ChatMsg --> KeywordHook

    MsgsTrans --> CtxInjectMsg[context-injector/messages-transform]
    MsgsTrans --> ThinkBlockVal[thinking-block-validator]
    MsgsTrans --> EmptyMsgSan[empty-message-sanitizer]

    ConfigEv --> ConfigHandler[config-handler]

    EventEv --> AutoUpdateHook[auto-update-checker]
    EventEv --> BgNotifHook[background-notification]
    EventEv --> SessionNotifHook[session-notification]
    EventEv --> TodoHook
    EventEv --> CtxWinHook

    ToolBefore --> TodoHook
    ToolBefore --> SessionRecoHook
    ToolBefore --> CommentCheck[comment-checker]
    ToolBefore --> NonIntEnv[non-interactive-env]
    ToolBefore --> AutoSlashHook

    ToolAfter --> CtxWinHook
    ToolAfter --> ToolTrunc[tool-output-truncator]
    ToolAfter --> CommentCheck
    ToolAfter --> EmptyTaskResp[empty-task-response-detector]
    ToolAfter --> AgentReminder[agent-usage-reminder]
    ToolAfter --> InteractiveBash[interactive-bash-session]
    ToolAfter --> EditRecoHook

    style TodoHook fill:#ffe1e1
    style SessionRecoHook fill:#ffe1e1
    style CtxWinHook fill:#ffe1e1
    style RalphHook fill:#ffe1e1
    style ThinkHook fill:#ffe1e1
```

---

## Tools 기능 관계

```mermaid
graph TB
    subgraph ToolCategories[툴 카테고리]
        LSPCat[LSP - 11개]
        ASTCat[AST-Grep - 2개]
        SearchCat[검색 - 2개]
        SessionCat[세션 - 4개]
        AgentCat[에이전트 - 1개]
        Multimodal[멀티모달 - 1개]
        SkillCat[스킬 - 2개]
        TmuxCat[터미널 - 1개]
        BgCat[백그라운드 - 3개]
        SlashCat[슬래시 - 1개]
    end

    subgraph InternalDependencies[내부 의존성]
        LSPClient[LspClient<br/>611 lines]
        BackgroundMgr[BackgroundManager]
        SkillMCPMgr[SkillMcpManager]
        ContextCollector[ContextCollector]
    end

    LSPCat --> LSPClient
    SessionCat --> SessionFiles[세션 파일 시스템]

    AgentCat --> BackgrdMgr
    BgCat --> BackgrdMgr
    SkillCat --> SkillMCPMgr

    BackgrdMgr --> SessionState[session-state]
    SkillMCPMgr --> McpLoader[mcp-loader]
    ContextCollector --> ContextInjector[context-injector]

    style LSPCat fill:#e1f5ff
    style ASTCat fill:#e1f5ff
    style BgCat fill:#ffe1e1
    style SkillMCPMgr fill:#ffe1e1
```

---

## Agents 기능 관계

```mermaid
graph TB
    subgraph Agents[7 AI 에이전트]
        Sisyphus[Sisyphus<br/>오케스트레이터]
        Oracle[oracle<br/>어드바이저]
        Librarian[librarian<br/>리서치]
        Explore[explore<br/>검색]
        Frontend[frontend-ui-ux-engineer<br/>UI/UX]
        DocWriter[document-writer<br/>문서 작성]
        MultiLooker[multimodal-looker<br/>PDF/이미지]
    end

    subgraph PromptBuilder[Sisyphus Prompt Builder]
        KeyTriggers[buildKeyTriggersSection]
        ToolSel[buildToolSelectionTable]
        ExploreSec[buildExploreSection]
        LibrarianSec[buildLibrarianSection]
        Delegation[buildDelegationTable]
        FrontendSec[buildFrontendSection]
        OracleSec[buildOracleSection]
        AntiPatterns[buildAntiPatternsSection]
    end

    subgraph CostBased[비용 기반 분류]
        FREE[explore<br/>grok-code]
        CHEAP[librarian, frontend, doc-writer<br/>sonnet-4.5, gemini-3]
        EXPENSIVE[oracle<br/>gpt-5.2]
    end

    subgraph Categories[카테고리 기반 분류]
        Exploration[explore, librarian<br/>exploration]
        Specialist[oracle, frontend, doc-writer<br/>specialist]
        Utility[multimodal-looker<br/>utility]
    end

    Sisyphus --> PromptBuilder
    PromptBuilder --> Agents
    PromptBuilder --> CostBased
    PromptBuilder --> Categories

    style Sisyphus fill:#00CED1
    style Oracle fill:#e1f5e1
    style Librarian fill:#e1f5e1
    style Explore fill:#e1f5e1
    style PromptBuilder fill:#ffe1e1
```

---

## Features 기능 관계

```mermaid
graph TB
    subgraph Loaders[로더 계층]
        AgentLoader[claude-code-agent-loader]
        CmdLoader[claude-code-command-loader]
        McpLoader[claude-code-mcp-loader]
        PluginLoader[claude-code-plugin-loader]
        SkillLoader[opencode-skill-loader]
    end

    subgraph Managers[관리자 계층]
        BackgrdMgr[background-agent<br/>TaskLifecycle]
        SkillMCPMgr[skill-mcp-manager<br/>MCPConnections]
        SkillMerger[opencode-skill-loader/merger<br/>SkillMerge]
    end

    subgraph State[상태 관리]
        SessionState[claude-code-session-state<br/>subagentSessions]
        MsgInjector[hook-message-injector<br/>FileSystem]
        CtxColl[context-injector/collector<br/>ContextCollector]
        CtxInj[context-injector/injector<br/>MessagesTransform]
    end

    subgraph Builtins[내장 리소스]
        BuiltinCmds[builtin-commands<br/>init-deep, ralph-loop]
        BuiltinSkills[builtin-skills<br/>playwright]
    end

    subgraph Shared[공통 유틸리티]
        Frontmatter[frontmatter]
        ModelSan[model-sanitizer]
        EnvExpand[env-expander]
        FileUtil[file-utils]
        DeepMerge[deep-merge]
        Logger[logger]
    end

    AgentLoader --> Frontmatter
    AgentLoader --> FileUtil
    CmdLoader --> Frontmatter
    CmdLoader --> ModelSan
    CmdLoader --> FileUtil
    McpLoader --> EnvExpand
    PluginLoader --> Frontmatter
    PluginLoader --> ModelSan
    PluginLoader --> FileUtil
    PluginLoader --> DeepMerge
    PluginLoader --> McpLoader
    PluginLoader --> CmdLoader
    PluginLoader --> SkillLoader

    SkillLoader --> Frontmatter
    SkillLoader --> ModelSan
    SkillLoader --> FileUtil
    SkillLoader --> DeepMerge
    SkillMerger --> DeepMerge
    SkillMerger --> Frontmatter

    BuiltinCmds --> SkillMerger
    BuiltinSkills --> SkillMerger

    BackgrdMgr --> SessionState
    BackgrdMgr --> MsgInjector
    BackgrdMgr --> Logger

    SkillMCPMgr --> McpLoader
    SkillMCPMgr --> EnvCleaner[env-cleaner]

    style AgentLoader fill:#e1f5e1
    style CmdLoader fill:#e1f5e1
    style McpLoader fill:#e1f5e1
    style PluginLoader fill:#e1f5e1
    style SkillLoader fill:#e1f5e1
    style BackgrdMgr fill:#e1f5ff
    style SkillMCPMgr fill:#e1f5ff
    style SessionState fill:#ffe1e1
    style MsgInjector fill:#ffe1e1
    style CtxColl fill:#ffe1e1
    style Shared fill:#fff4e1
```

---

## 데이터 흐름

```mermaid
sequenceDiagram
    participant User as 사용자
    participant CLI as OpenCode CLI
    participant Plugin as Plugin
    participant Hook as Hooks
    participant Agent as Sisyphus
    participant Tool as Tools
    participant LSP as LSP Client
    participant Backgrd as Background Manager
    participant Explore as Explore Agent
    participant Librarian as Librarian Agent

    User->>CLI: 사용자 입력
    CLI->>Plugin: session.prompt
    Plugin->>Hook: chat.message
    Hook->>Plugin: 수정된 입력

    Plugin->>Agent: Sisyphus 호출
    Agent->>Agent: 스킬 체크
    Agent->>Tool: 툴 호출
    Tool->>Hook: tool.execute.before
    Hook->>Tool: 승인된 입력

    Tool->>LSP: lsp_hover
    LSP->>Tool: 결과
    Tool->>Hook: tool.execute.after
    Hook->>Plugin: 처리 완료

    Agent->>Backgrd: background_task(explore)
    Backgrd->>Explore: 세션 생성
    Explore->>Tool: grep, glob 병렬
    Tool->>Agent: 결과 수집
    Agent->>Hook: background_output
    Hook->>Plugin: 결과 전달

    Plugin->>CLI: 최종 응답
    CLI->>User: 결과 표시
```

---

## 설정 우선순위 흐름

```mermaid
graph TB
    subgraph Sources[설정 소스]
        ProjectConfig[opencode.json<br/>프로젝트]
        UserConfig[oh-my-opencode.json<br/>사용자]
        ClaudeCodeClaude[settings.json<br/>Claude Code]
        Defaults[기본값<br/>코드 내장]
    end

    subgraph Mergers[병합 과정]
        SkillMerge[스킬 병합]
        AgentOverride[에이전트 오버라이드]
        HookFilter[훅 활성/비활성]
    end

    subgraph Outputs[최종 설정]
        MergedSkills[병합된 스킬]
        MergedAgents[오버라이드된 에이전트]
        ActiveHooks[활성화된 훅들]
    end

    ProjectConfig --> SkillMerge
    UserConfig --> SkillMerge
    Defaults --> SkillMerge

    UserConfig --> AgentOverride
    Defaults --> AgentOverride

    UserConfig --> HookFilter
    Defaults --> HookFilter

    BuiltinSkills[내장 스킬] --> SkillMerge

    SkillMerge --> MergedSkills
    AgentOverride --> MergedAgents
    HookFilter --> ActiveHooks

    style ProjectConfig fill:#ffe1e1
    style UserConfig fill:#ffe1e1
    style Defaults fill:#fff4e1
```

---

## OpenCode Plugin 컴포넌트

### 1. Tools (도구)

| 카테고리 | 툴 | 설명 |
|-----------|------|------|
| **LSP (11개)** | `lsp_hover`, `lsp_goto_definition`, `lsp_find_references`, `lsp_document_symbols`, `lsp_workspace_symbols`, `lsp_diagnostics`, `lsp_servers`, `lsp_prepare_rename`, `lsp_rename`, `lsp_code_actions`, `lsp_code_action_resolve` | 코드 네비게이션, 정의, 참조 찾기 |
| **AST** | `ast_grep_search`, `ast_grep_replace` | 구조 기반 검색/치환 |
| **파일 검색** | `grep`, `glob` | 텍스트 검색, 파일 패턴 매칭 |
| **세션 관리** | `session_list`, `session_read`, `session_search`, `session_info` | 세션 파일 읽기/검색 |
| **에이전트** | `call_omo_agent` | explore/librarian 서브에이전트 호출 |
| **멀티모달** | `look_at` | PDF/이미지 분석 |
| **스킬** | `skill`, `skill_mcp` | 스킬 실행, 스킬 MCP 호출 |
| **터미널** | `interactive_bash` | Tmux 세션 관리 |
| **슬래시 명령** | `slashcommand` | /command 실행 |
| **백그라운드** | `background_task`, `background_output`, `background_cancel` | 비동기 태스크 관리 |

**총 20+ 툴 제공**

---

### 2. Hooks (이벤트 핸들러)

| 훅 | 타이밍 | 용도 |
|-----|---------|------|
| `chat.message` | 채팅 메시지 | ralph-loop, context-injection 처리 |
| `experimental.chat.messages.transform` | 메시지 변환 | synthetic message 주입 |
| `config` | 설정 변경 | config-handler로 처리 |
| `event` | 세션 이벤트 | session.created, session.deleted, session.error 등 |
| `tool.execute.before` | 툴 실행 전 | 유효성 검사, 입력 수정 |
| `tool.execute.after` | 툴 실행 후 | 결과 처리, 컨텍스트 추가 |

**총 6개 메인 훅 포인트 제공**

---

### 3. Auth (인증)

Google Antigravity OAuth 제공:

```typescript
auth: googleAuthHooks.auth  // Bearer token injection
```

**기능:**
- Gemini 모델용 OAuth 플로우
- 자동 토큰 갱신
- 다중 계정 지원 (최대 10개)
- Thinking block 보존

---

### 4. Config Handler

설정 변경 처리:

```typescript
config: configHandler
```

**기능:**
- `oh-my-opencode.json` 변경 감지
- 리얼타임 설정 업데이트
- hook/agent/tool 활성화 비활성화

---

### 5. Background Manager

백그라운드 태스크 관리 (OpenCode에 직접 노출되지 않지만 internal):

```typescript
const backgroundManager = new BackgroundManager(ctx)
```

**기능:**
- `background_task`로 세션 생성
- `background_output`로 결과 수집
- `background_cancel`로 취소
- TTL 정리 (30분)

---

### 6. Context Collector

컨텍스트 수집 및 주입 (internal):

```typescript
contextCollector = new ContextCollector()
contextInjector = createContextInjectorHook(contextCollector)
```

**기능:**
- `experimental.chat.messages.transform` 훅으로 컨텍스트 주입
- 우선순위 기반 정렬 (critical > high > normal > low)

---

## 사용자 관점에서 활용 가능한 기능

OpenCode CLI에서 다음 기능들을 사용할 수 있습니다:

### 1. 자동화된 스킬 실행
```
/skill playwright
/skill init-deep
```

### 2. LSP 기반 코드 네비게이션
```
lsp_hover 파일경로 라인
lsp_goto_definition 파일경로 라인
lsp_find_references 파일경로 라인
```

### 3. 백그라운드 연구
```
background_task(agent="explore", prompt="X구조 파악")
background_task(agent="librarian", prompt="라이브러리 레퍼런스")
// 나중에 수집
background_output(task_id="...")
```

### 4. AST 기반 리팩토링
```
ast_grep_search(pattern="function $NAME($$)", language=typescript)
ast_grep_replace(pattern="...", replacement="...", language=typescript)
```

### 5. 세션 이력 검색
```
session_search(query="auth implementation")
session_read(session_id="...")
```

### 6. Ralph Loop (반복 개발)
```
/ralph-loop "버그 수정"
/cancel-ralph
```

---

## 요약

| 구분 | 제공 항목 | 수량 |
|--------|------------|------|
| Tools | LSP, AST, 검색, 세션, 에이전트, 스킬, 터미널 | 20+ |
| Hooks | chat.message, messages.transform, tool.execute, config, event | 6개 포인트 |
| Auth | Google Antigravity OAuth | 1개 |
| Config | 실시간 설정 변경 감지 | 1개 |
| Internal | BackgroundManager, ContextCollector | 2개 |

OpenCode 사용자는 이 플러그인을 통해 **엔터프라이즈급 개발 도구 세트**와 **다중 모델 오케스트레이션**을 활용할 수 있습니다.

---

## 범례

| 색상 | 카테고리 |
|--------|----------|
| 🟢 #00CED1 | Sisyphus (오케스트레이터) |
| 🟢 #e1f5e1 | Agents (AI 에이전트들) |
| 🔵 #e1f5ff | Tools (도구들) |
| 🔴 #ffe1e1 | Hooks (훅들), Features (기능들) |
| 🟡 #fff4e1 | Shared (공통 유틸리티) |
