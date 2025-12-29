# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Build & Type Checking
- **Type check**: `bun run typecheck`
- **Full build**: `bun run build` - Builds ESM output with `bun build`, generates TypeScript declarations with `tsc --emitDeclarationOnly`, and builds the JSON schema
- **Schema only**: `bun run build:schema` - Rebuilds JSON schema after modifying `src/config/schema.ts`

### Testing
- **Run tests**: `bun test`
- **Single test file**: `bun test <path-to-test-file.test.ts>`

### Local Development Testing
To test your local build in OpenCode, update your config (`~/.config/opencode/opencode.json` or `opencode.jsonc`):
```json
{
  "plugin": [
    "file:///absolute/path/to/oh-my-opencode/dist/index.js"
  ]
}
```

## Architecture Overview

**Oh-My-OpenCode** is an OpenCode plugin that provides Claude Code/AmpCode-style features with multi-model agent orchestration, enhanced tools, and comprehensive compatibility layers.

### Core Architecture Pattern

The plugin follows a **feature-based modular architecture** where each major subsystem is self-contained:

1. **Main Plugin Entry** (`src/index.ts`): Orchestrates all components, handles config loading (user + project with deep merge), and registers tools/agents/hooks/events
2. **Agent System** (`src/agents/`): 7 built-in AI agents (Sisyphus/oracle/librarian/explore/frontend-ui-ux-engineer/document-writer/multimodal-looker) plus loader for user-defined agents
3. **Hook System** (`src/hooks/`): 21 lifecycle hooks that intercept events (`session.created`, `tool.execute.before`, etc.) for cross-cutting concerns
4. **Tool System** (`src/tools/`): Custom tools including LSP integration (11 tools), AST-Grep, Grep, Glob, session management, background tasks, and interactive bash
5. **MCP Integration** (`src/mcp/`): Built-in MCP servers (context7, websearch_exa, grep_app) plus loader for user-defined MCPs
6. **Claude Code Compatibility** (`src/features/`): Loaders for Claude Code's commands/skills/agents/MCPs/hooks from standard locations (`~/.claude/`, `.claude/`, etc.)

### Config Loading Priority

Config files are loaded with cascading precedence:
1. **User config**: `~/.config/opencode/oh-my-opencode.jsonc` (preferred) or `.json`
2. **Project config**: `.opencode/oh-my-opencode.jsonc` (preferred) or `.json`

Project config **deep merges** with user config (arrays are concatenated for `disabled_*` lists). Both support JSONC syntax (comments, trailing commas).

### Agent Name Migration

The project underwent a renaming: `OmO` → `Sisyphus`. The main plugin handles automatic migration of legacy config files and maintains backward compatibility through `AGENT_NAME_MAP` in `src/index.ts`.

### Hook Event Flow

Hooks are called in a specific order. Key event types:
- **config**: Initialize agents, tools, MCPs, commands
- **event**: Session lifecycle (`session.created`, `session.error`, `session.deleted`)
- **tool.execute.before**: Pre-tool validation/injection (comment checker, directory AGENTS.md/README.md injector, rules injector)
- **tool.execute.after**: Post-tool processing (truncation, context monitoring)

### Sisyphus Agent Orchestration

When enabled (default), `Sisyphus` becomes the default agent and `Planner-Sisyphus` replaces the default `plan` agent. The original `build` and `plan` agents are demoted to "subagent" mode. This is controlled by the `sisyphus_agent` config object.

### Background Agent System

The `BackgroundManager` (`src/features/background-agent/`) manages async agent execution:
- Agents run via `call_omo_agent` tool with `run_in_background: true`
- Main agent gets notified via background notification hook on completion
- Results retrieved via `TaskOutput` tool

## Key Conventions

### Package Manager & Build
- **Bun only**: Never use npm/yarn
- **Types**: Use `bun-types`, not `@types/node`
- **Dual build**: `bun build` for EM output + `tsc --emitDeclarationOnly` for `.d.ts`
- **No local publishing**: Version bumps and npm publishes are GitHub Actions only

### Code Organization
- **Directory naming**: kebab-case (`ast-grep/`, `claude-code-hooks/`)
- **Tool structure**: Each tool directory contains `index.ts`, `types.ts`, `constants.ts`, `tools.ts`, `utils.ts`
- **Hook pattern**: `createXXXHook(input: PluginInput)` function returning event handlers
- **Barrel exports**: `export * from "./module"` in index.ts files

### Anti-Patterns to Avoid
- Using bash commands (mkdir/touch/rm/cp/mv) for file operations in code
- Suppressing TypeScript errors with `as any`, `@ts-ignore`, `@ts-expect-error`
- Modifying `package.json` version locally
- Running `bun publish` directly (OIDC provenance requires GitHub Actions)
- Using year 2024 in prompts/code (use current year)

## Where Things Are Located

| Task | Location |
|------|----------|
| Add new agent | `src/agents/` - Create `.ts` file, add to `builtinAgents` in `index.ts` |
| Add new hook | `src/hooks/` - Create directory with `createXXXHook()`, export from `index.ts` |
| Add new tool | `src/tools/` - Create directory with standard structure, add to `builtinTools` |
| Add new MCP | `src/mcp/` - Create config, add to `index.ts` |
| LSP tools | `src/tools/lsp/` - `client.ts` (connection), `tools.ts` (11 LSP handlers) |
| AST-Grep | `src/tools/ast-grep/napi.ts` - `@ast-grep/napi` binding |
| Google OAuth | `src/auth/antigravity/` - Antigravity OAuth plugin |
| Config schema | `src/config/schema.ts` - Zod schema (run `bun run build:schema` after changes) |
| Claude Code loaders | `src/features/claude-code-*-loader/` - Command/skill/agent/MCP loaders |
| Session state | `src/features/claude-code-session-state/` - Main session tracking |
| CLI installer | `src/cli/install.ts` - Interactive installer script |

## Publishing

**GitHub Actions workflow_dispatch only.**

1. Never modify `package.json` version locally
2. Commit and push changes
3. Trigger publish workflow: `gh workflow run publish -f bump=patch` (or `minor`/`major`)

Direct `bun publish` is prohibited due to OIDC provenance requirements.
