# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code Skill** (pure documentation, no code) that enables Claude-Codex collaboration for code review, implementation planning, and debugging via MCP tools.

**Skill Name**: `codex`
**Version**: 0.2.0
**Prerequisite**: CodexMCP server must be installed

## Architecture

```
codex-skill/
├── SKILL.md                    # Skill definition (YAML frontmatter with hooks + quick start)
├── commands/                   # Command specifications
│   ├── codex-review.md         # Code review workflow
│   ├── codex-plan.md           # Implementation planning workflow
│   └── codex-debug.md          # Debug/root cause analysis workflow
├── reference/                  # Core documentation
│   ├── shared-patterns.md      # 6 core rules (including rule 0), 6-step workflow, PROMPT templates
│   ├── examples.md             # Real-world usage examples
│   └── troubleshooting.md      # FAQ and known limitations
└── .github/workflows/
    └── release.yml             # Tag-triggered release with zip packaging
```

## SKILL.md Frontmatter

The skill uses these key configurations:
- `user-invocable: true` - Can be explicitly invoked
- `allowed-tools` - Whitelisted tools: `mcp__codex__codex`, `Read`, `Grep`, `Glob`, `Bash(git:*)`
- `hooks.PreToolUse` - Reminder prompt before calling Codex

## Core Collaboration Rules (Must Follow)

0. **Plan Detection First** - Check `.claude/plans/*.md` and `~/.claude/plans/*.md` before calling Codex
1. **Analyze First** - Output preliminary analysis BEFORE calling Codex
2. **Ask When Unclear** - Request clarification if information is insufficient
3. **Read-Only Sandbox** - Always use `sandbox="read-only"` for `mcp__codex__codex`
4. **Diff-Based Output** - Codex provides unified diff patches or issue lists only
5. **Verify Conclusions** - Cross-validate Codex results, note disagreements

## 6-Step Standard Workflow

All commands follow this workflow (details in `reference/shared-patterns.md`):

1. Parse input parameters
2. Gather context (read files, git diff)
3. **Output preliminary analysis** (mandatory)
4. Call `mcp__codex__codex` with appropriate PROMPT template
5. Critically evaluate Codex conclusions
6. Output final results with SESSION_ID

## Key MCP Tool Usage

```python
mcp__codex__codex(
    cd="project_root_path",
    sandbox="read-only",              # REQUIRED
    SESSION_ID="<id>",                # Optional, for continuing sessions
    PROMPT="<see templates>",
    return_all_messages=False         # True for debug scenarios
)
```

PROMPT templates are in `reference/shared-patterns.md` under "Codex 调用模板".

## Release Process

Push a tag (e.g., `git tag v0.1.0 && git push origin v0.1.0`) to trigger GitHub Actions workflow that creates a release with packaged zip file.
