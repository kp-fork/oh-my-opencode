# src/features/ Architecture Analysis

**Generated:** 2026-01-05
**Branch:** feature/analyze-features
**Commit:** 65b00c9

## Overview

`src/features/` contains 13 feature modules that provide Claude Code compatibility layer and core functionality for OpenCode plugin. This analysis covers architecture, dependencies, and detailed breakdown of each feature.

---

## Architecture Diagram

```mermaid
graph TB
    subgraph ExternalSources[External Sources]
        CFS[Claude Code<br/>~/.claude/*]
        OFS[OpenCode<br/>~/.config/opencode/*]
        PROJ[Project<br/>.claude/*, .opencode/*]
        PLUGINS[Plugins<br/>~/.claude/plugins/*]
        ENV[Environment<br/>process.env]
    end

    subgraph Builtins[Built-in Resources]
        BUILTIN_CMD[builtin-commands<br/>init-deep, ralph-loop]
        BUILTIN_SKILL[builtin-skills<br/>playwright]
    end

    subgraph LoaderCC[Claude Code Loaders]
        AGENT_LDR[claude-code-agent-loader<br/>→ AgentConfig[]]
        CMD_LDR[claude-code-command-loader<br/>→ CommandDefinition[]]
        MCP_LDR[claude-code-mcp-loader<br/>→ McpServerConfig[]]
        PLUGIN_LDR[claude-code-plugin-loader<br/>→ All Components]
    end

    subgraph LoaderOC[OpenCode Loaders]
        SKILL_LDR[opencode-skill-loader<br/>→ LoadedSkill[]]
    end

    subgraph Managers[Feature Managers]
        BG_MGR[background-agent<br/>TaskLifecycle]
        SKILL_MCP_MGR[skill-mcp-manager<br/>MCPConnections]
        SKILL_MERGER[opencode-skill-loader/merger<br/>SkillMerge]
    end

    subgraph State[State & Injection]
        SESSION_STATE[claude-code-session-state<br/>subagentSessions]
        MSG_INJECTOR[hook-message-injector<br/>FileSystem]
        CTX_COLLECTOR[context-injector<br/>ContextCollector]
        CTX_INJECTOR[context-injector/injector<br/>MessagesTransform]
    end

    subgraph Shared[Shared Utilities]
        FRONTMATTER[frontmatter<br/>parseFrontmatter]
        MODEL_SAN[model-sanitizer<br/>sanitizeModelField]
        ENV_EXPANDER[env-expander<br/>expandEnvVarsInObject]
        ENV_CLEANER[env-cleaner<br/>createCleanMcpEnvironment]
        FILE_UTILS[file-utils<br/>isMarkdownFile, resolveSymlink]
        DEEP_MERGE[deep-merge]
        LOGGER[logger]
        OPENCODE_CONFIG[opencode-config-dir]
    end

    subgraph Config[Config Schema]
        SCHEMA[config/schema<br/>SkillsConfig, SkillDefinition]
    end

    Main[Main Plugin<br/>src/index.ts]

    ExternalSources --> LoaderCC
    ExternalSources --> LoaderOC
    PLUGINS --> PLUGIN_LDR
    ENV --> MCP_LDR
    ENV --> SKILL_MCP_MGR

    BUILTIN_CMD --> SKILL_MERGER
    BUILTIN_SKILL --> SKILL_MERGER
    SKILL_LDR --> SKILL_MERGER
    SCHEMA --> SKILL_MERGER

    AGENT_LDR --> Main
    CMD_LDR --> Main
    MCP_LDR --> Main
    PLUGIN_LDR --> Main
    SKILL_MERGER --> Main

    BG_MGR --> SESSION_STATE
    BG_MGR --> MSG_INJECTOR
    CTX_INJECTOR --> Main

    SKILL_MCP_MGR --> MSG_INJECTOR

    AGENT_LDR --> FRONTMATTER
    AGENT_LDR --> FILE_UTILS
    AGENT_LDR --> OPENCODE_CONFIG

    CMD_LDR --> FRONTMATTER
    CMD_LDR --> MODEL_SAN
    CMD_LDR --> FILE_UTILS
    CMD_LDR --> OPENCODE_CONFIG

    MCP_LDR --> OPENCODE_CONFIG
    MCP_LDR --> ENV_EXPANDER

    PLUGIN_LDR --> CMD_LDR
    PLUGIN_LDR --> MCP_LDR
    PLUGIN_LDR --> SKILL_LDR
    PLUGIN_LDR --> FRONTMATTER
    PLUGIN_LDR --> MODEL_SAN
    PLUGIN_LDR --> FILE_UTILS
    PLUGIN_LDR --> DEEP_MERGE

    SKILL_LDR --> FRONTMATTER
    SKILL_LDR --> MODEL_SAN
    SKILL_LDR --> FILE_UTILS
    SKILL_LDR --> OPENCODE_CONFIG

    SKILL_MERGER --> DEEP_MERGE
    SKILL_MERGER --> FRONTMATTER
    SKILL_MERGER --> MODEL_SAN

    BG_MGR --> LOGGER

    SKILL_MCP_MGR --> MCP_LDR
    SKILL_MCP_MGR --> ENV_EXPANDER
    SKILL_MCP_MGR --> ENV_CLEANER

    MSG_INJECTOR --> FILE_UTILS

    style AGENT_LDR fill:#e1f5e1
    style CMD_LDR fill:#e1f5e1
    style MCP_LDR fill:#e1f5e1
    style PLUGIN_LDR fill:#e1f5e1
    style SKILL_LDR fill:#e1f5e1
    style SKILL_MERGER fill:#ffe1e1
    style BG_MGR fill:#e1f5ff
    style SKILL_MCP_MGR fill:#e1f5ff
    style CTX_COLLECTOR fill:#ffe1e1
    style SESSION_STATE fill:#ffe1e1
    style MSG_INJECTOR fill:#ffe1e1
    style CTX_INJECTOR fill:#ffe1e1
    style FRONTMATTER fill:#fff4e1
    style MODEL_SAN fill:#fff4e1
    style ENV_EXPANDER fill:#fff4e1
    style ENV_CLEANER fill:#fff4e1
    style FILE_UTILS fill:#fff4e1
    style DEEP_MERGE fill:#fff4e1
```

### Legend

| Color | Category |
|-------|----------|
| 🟢 Green | Claude Code Loaders |
| 🔴 Red | OpenCode Loaders/Managers |
| 🔵 Blue | Managers |
| 🟡 Yellow | Shared Utilities |

---

## Feature Analysis

### Loaders (Claude Code Compatibility)

#### claude-code-agent-loader

| Property | Value |
|----------|-------|
| **Purpose** | Load `~/.claude/agents/*.md` → `AgentConfig[]` |
| **Files** | loader.ts (91 lines) |
| **Key Functions** | `loadUserAgents()`, `loadProjectAgents()` |
| **Scope Labels** | `(user)`, `(project)` |
| **Features** | Frontmatter parsing, tools config parsing |

---

#### claude-code-command-loader

| Property | Value |
|----------|-------|
| **Purpose** | Load `~/.claude/commands/*.md` → `CommandDefinition[]` |
| **Files** | loader.ts (260 lines) |
| **Key Functions** | `loadUserCommands()`, `loadProjectCommands()`, `loadOpencodeGlobalCommands()`, `loadOpencodeProjectCommands()` |
| **Scope Labels** | `(user)`, `(project)`, `(opencode)`, `(opencode-project)` |
| **Features** | Async/sync loaders, nested directory support, `agent`, `model`, `subtask`, `handoffs` frontmatter |

---

#### claude-code-mcp-loader

| Property | Value |
|----------|-------|
| **Purpose** | Load `.mcp.json` → `McpServerConfig[]` |
| **Files** | loader.ts (114 lines), transformer.ts (54 lines), env-expander.ts (28 lines) |
| **Key Functions** | `loadMcpConfigs()`, `getSystemMcpServerNames()` |
| **Sources** | `~/.claude/.mcp.json`, `.mcp.json`, `.claude/.mcp.json` |
| **Features** | stdio/remote transformation, `${VAR:-default}` expansion |

---

#### claude-code-plugin-loader

| Property | Value |
|----------|-------|
| **Purpose** | Load `installed_plugins.json` → all components |
| **Files** | loader.ts (487 lines) |
| **Key Functions** | `discoverInstalledPlugins()`, `loadPluginCommands()`, `loadPluginSkillsAsCommands()`, `loadPluginAgents()`, `loadPluginMcpServers()`, `loadPluginHooksConfigs()` |
| **Manifest** | `.claude-plugin/plugin.json` |
| **Features** | Namespace prefixes (`plugin:name`), enabled/disabled filtering, `${CLAUDE_PLUGIN_ROOT}` resolution |

---

#### opencode-skill-loader

| Property | Value |
|----------|-------|
| **Purpose** | Multi-source skill loading with lazy content |
| **Files** | loader.ts (458 lines) |
| **Key Functions** | `discoverAllSkills()`, `getSkillByName()` |
| **Scope Priority** | opencode-project (6) > project (5) > opencode (4) > user (3) > config (2) > builtin (1) |
| **Patterns Supported** | `skill-name/SKILL.md`, `skill-name/{skill-name}.md`, `skill-name.md` |
| **Features** | Lazy template loading, MCP config from frontmatter/mcp.json, `allowed-tools` parsing |

---

### Built-in Resources

#### builtin-commands

| Property | Value |
|----------|-------|
| **Commands** | `init-deep`, `ralph-loop`, `cancel-ralph` |
| **Files** | commands.ts (52 lines), init-deep.ts (301 lines), ralph-loop.ts (39 lines) |
| **init-deep** | Generate hierarchical AGENTS.md with complexity scoring |
| **ralph-loop** | Self-referential development loop until completion |

---

#### builtin-skills

| Property | Value |
|----------|-------|
| **Skills** | `playwright` |
| **Files** | skills.ts (20 lines) |
| **Features** | Embedded MCP config (`@playwright/mcp@latest`) |

---

### Feature Managers

#### background-agent

| Property | Value |
|----------|-------|
| **Purpose** | Background task lifecycle management |
| **Files** | manager.ts (499 lines) |
| **Key Functions** | `launch()`, `handleEvent()`, `notifyParentSession()` |
| **Lifecycle** | pending → running → completed/failed/cancelled |
| **Features** | Session polling, OS notifications, TTL pruning (30 min), `background_output` retrieval |

---

#### skill-mcp-manager

| Property | Value |
|----------|-------|
| **Purpose** | MCP server management for skills |
| **Files** | manager.ts (317 lines), env-cleaner.ts (28 lines) |
| **Key Functions** | `getOrCreateClient()`, `disconnectSession()`, `callTool()` |
| **Features** | Lazy connection, session-scoped cleanup, idle timeout (5 min), pnpm/YARN env filtering |

---

#### opencode-skill-loader/merger

| Property | Value |
|----------|-------|
| **Purpose** | Skill merging with priority resolution |
| **Files** | merger.ts (268 lines) |
| **Key Functions** | `mergeSkills()` |
| **Config Schema** | `skills.enable[]`, `skills.disable[]`, `skills.entries{}` |
| **Features** | Deep merge of metadata/tools, `{file:path}` resolution, config → file override |

---

### State Management

#### claude-code-session-state

| Property | Value |
|----------|-------|
| **Purpose** | Subagent session tracking |
| **Files** | state.ts (12 lines) |
| **Exports** | `subagentSessions: Set<string>`, `mainSessionID`, `setMainSession()`, `getMainSessionID()` |

---

#### hook-message-injector

| Property | Value |
|----------|-------|
| **Purpose** | Inject messages into conversation via filesystem |
| **Files** | injector.ts (152 lines) |
| **Key Functions** | `injectHookMessage()`, `findNearestMessageWithFields()` |
| **Storage** | `MESSAGE_STORAGE/sessionID/{messageID}.json`, `PART_STORAGE/{messageID}/{partID}.json` |
| **Features** | Fallback to nearest valid message, synthetic flag |

---

#### context-injector

| Property | Value |
|----------|-------|
| **Purpose** | Context collection and injection |
| **Files** | collector.ts (86 lines), injector.ts (157 lines) |
| **Key Functions** | `ContextCollector.register()`, `createContextInjectorMessagesTransformHook()` |
| **Priorities** | critical > high > normal > low |
| **Injection** | Prepend to last user message as synthetic message |

---

## Dependency Matrix

### Feature → Shared Dependencies

| Feature | frontmatter | model-sanitizer | env-expander | env-cleaner | file-utils | deep-merge | logger | opencode-config |
|---------|-------------|-----------------|-------------|-------------|------------|-----------|--------|-----------------|
| claude-code-agent-loader | ✅ | | | | ✅ | | | ✅ |
| claude-code-command-loader | ✅ | ✅ | | | ✅ | | | ✅ |
| claude-code-mcp-loader | | | ✅ | | | | | ✅ |
| claude-code-plugin-loader | ✅ | ✅ | ✅ | | ✅ | ✅ | | |
| opencode-skill-loader | ✅ | ✅ | | | ✅ | ✅ | | ✅ |
| opencode-skill-loader/merger | ✅ | ✅ | | | | ✅ | | |
| background-agent | | | | | | | ✅ | |
| skill-mcp-manager | | | ✅ | ✅ | | | | |

---

### Feature → Feature Dependencies

| From | To | Type | Description |
|------|-----|------|-------------|
| claude-code-plugin-loader | claude-code-command-loader/types | Import | CommandDefinition type |
| claude-code-plugin-loader | claude-code-mcp-loader/transformer | Import | transformMcpServer |
| claude-code-plugin-loader | claude-code-mcp-loader/env-expander | Import | Environment expansion |
| claude-code-plugin-loader | opencode-skill-loader/types | Import | SkillMetadata type |
| opencode-skill-loader | claude-code-command-loader/types | Import | CommandDefinition type |
| opencode-skill-loader | skill-mcp-manager/types | Import | SkillMcpConfig type |
| skill-mcp-manager | claude-code-mcp-loader/types | Import | ClaudeCodeMcpServer type |
| skill-mcp-manager | claude-code-mcp-loader/env-expander | Import | Environment expansion |
| background-agent | hook-message-injector | Import | findNearestMessageWithFields, MESSAGE_STORAGE |
| background-agent | claude-code-session-state | Import | subagentSessions |

---

### Internal Feature Dependencies

| From | To | Type | Description |
|------|-----|------|-------------|
| claude-code-mcp-loader/transformer | claude-code-mcp-loader/env-expander | Import | Environment expansion before transformation |
| skill-mcp-manager/manager | skill-mcp-manager/env-cleaner | Import | Clean environment for MCP processes |

---

## Key Findings

### 1. Central Dependencies

- **shared/frontmatter**: Most reused utility (5 features)
- **shared/model-sanitizer**: Used by all loaders (4 features)
- **shared/file-utils**: Used by all loaders (4 features)

### 2. Loader Hierarchy

```
claude-code-plugin-loader (orchestrator)
├── claude-code-command-loader (commands)
├── claude-code-mcp-loader (MCP servers)
└── opencode-skill-loader (skills)
```

### 3. State Integration

```
background-agent
├── claude-code-session-state (subagent tracking)
└── hook-message-injector (notification delivery)
```

### 4. Code Reuse

- **skill-mcp-manager** shares `ClaudeCodeMcpServer` type from `claude-code-mcp-loader` (avoid duplication)
- **opencode-skill-loader/merger** uses `claude-code-command-loader/types.CommandDefinition` for type consistency

### 5. Separation of Concerns

- **context-injector**: Collector (state) + Injector (hook) split for testability
- **opencode-skill-loader**: Loader (discovery) + Merger (logic) split

### 6. Feature Complexity

| Feature | Total Lines | Complexity |
|---------|-------------|------------|
| claude-code-plugin-loader | 487 | High (multi-component orchestration) |
| background-agent | 499 | High (lifecycle + polling + notifications) |
| opencode-skill-loader | 458 | High (multi-source + lazy loading) |
| opencode-skill-loader/merger | 268 | Medium (priority + merge logic) |
| skill-mcp-manager | 317 | Medium (connection management) |
| claude-code-command-loader | 260 | Medium (async/sync variants) |

---

## Recommendations

### 1. Extract Common Loader Logic

```typescript
// src/features/loaders/shared/
export abstract class BaseLoader<T> {
  protected parseFrontmatter<T>(content: string)
  protected resolvePath(path: string)
  // ...
}
```

### 2. Unify Skill/Command Interfaces

Both loaders use similar `CommandDefinition` structure - consider a shared type in `shared/`.

### 3. Type Safety for MCP Configs

Add runtime validation using Zod for `ClaudeCodeMcpServer` parsing (currently JSON.parse only).

### 4. Background Agent Event Driven

Replace polling (2s interval) with event-driven architecture using OpenCode event hooks.

### 5. Lazy Loading Documentation

Add JSDoc comments to `LazyContentLoader` explaining load-on-first-use pattern.

---

## File Index

```
src/features/
├── background-agent/
│   ├── manager.ts (499 lines)
│   └── manager.test.ts
├── builtin-commands/
│   ├── commands.ts (52 lines)
│   ├── templates/init-deep.ts (301 lines)
│   └── templates/ralph-loop.ts (39 lines)
├── builtin-skills/
│   └── skills.ts (20 lines)
├── claude-code-agent-loader/
│   └── loader.ts (91 lines)
├── claude-code-command-loader/
│   ├── loader.ts (260 lines)
│   └── types.ts
├── claude-code-mcp-loader/
│   ├── loader.ts (114 lines)
│   ├── transformer.ts (54 lines)
│   ├── env-expander.ts (28 lines)
│   └── types.ts
├── claude-code-plugin-loader/
│   └── loader.ts (487 lines)
├── claude-code-session-state/
│   └── state.ts (12 lines)
├── context-injector/
│   ├── collector.ts (86 lines)
│   └── injector.ts (157 lines)
├── hook-message-injector/
│   ├── injector.ts (152 lines)
│   └── constants.ts
└── opencode-skill-loader/
    ├── loader.ts (458 lines)
    ├── merger.ts (268 lines)
    ├── async-loader.ts (180 lines)
    ├── blocking.ts (62 lines)
    ├── discover-worker.ts (59 lines)
    └── types.ts
```

---

**Total Lines Analyzed:** ~3,500 lines
**Files Analyzed:** 25 files
**Dependencies Tracked:** 30+ relationships
