# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Autonoe Skill is an autonomous agent orchestrator skill for Claude Code, packaged as a Claude plugin.

## Plugin Structure

```
.claude-plugin/
└── plugin.json          # Plugin manifest (required)
commands/                # Markdown skill files (slash commands)
agents/                  # Custom agent definitions
skills/                  # Skills with SKILL.md structure
hooks/                   # Event handlers (hooks.json)
.mcp.json               # MCP server configurations
.lsp.json               # LSP server configurations
```

**Important**: Only `plugin.json` goes inside `.claude-plugin/`. All other components must be at the plugin root level.

## Plugin Development

### Testing Locally

```bash
claude --plugin-dir ./
```

### plugin.json Schema

Required field: `name` (kebab-case, used as namespace prefix for skills)

Optional but recommended: `version`, `description`, `author`, `license`, `homepage`, `repository`, `keywords`

Component paths (all relative with `./` prefix): `commands`, `agents`, `skills`, `hooks`, `mcpServers`, `lspServers`

### Skill Structure

Each skill is a directory containing `SKILL.md`:

```
skills/
└── my-skill/
    ├── SKILL.md         # Required: instructions with frontmatter
    ├── reference.md     # Optional: supporting documentation
    └── scripts/         # Optional: executable scripts
```

### SKILL.md Frontmatter

```yaml
---
name: skill-name
description: When to use this skill
argument-hint: "[optional-hint]"
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep
context: fork
agent: Explore
---
```

Key fields:
- `disable-model-invocation: true` - Prevent automatic invocation (manual-only)
- `user-invocable: false` - Hide from users (Claude-only)
- `context: fork` - Run in isolated subagent
- `allowed-tools` - Restrict available tools

### Plugin Namespacing

Plugin skills use `/autonoe:skill-name` format. Use `${CLAUDE_PLUGIN_ROOT}` for absolute paths in hooks and MCP configs.

## Commit Message Guidelines

Plugin Markdown files (`commands/`, `skills/`, `agents/`) are **functional definitions**, not documentation.

| Change Type | Prefix | Example |
|------------|--------|---------|
| New skill/command | `feat:` | `feat: add git-commit command` |
| Skill behavior change | `fix:` | `fix: improve error handling in skill` |
| Skill prompt refinement | `refactor:` | `refactor: clarify instructions` |
| Actual documentation (README, etc.) | `docs:` | `docs: update installation guide` |

**Important**: Avoid using `docs:` for changes to skill/command Markdown files. These files define behavior and should use `feat:`, `fix:`, or `refactor:` based on the nature of the change.
