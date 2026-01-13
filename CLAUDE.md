# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code Skill** (pure documentation, no code) that enables Claude-Codex collaboration for code review, implementation planning, and debugging via MCP tools.

**Skill Name**: `codex`
**Version**: 0.1.0
**Prerequisite**: CodexMCP server must be installed

## Architecture

```
codex/
├── SKILL.md                    # Skill definition (YAML frontmatter + quick start)
├── commands/                   # Command specifications
│   ├── codex-review.md         # Code review workflow
│   ├── codex-plan.md           # Implementation planning workflow
│   └── codex-debug.md          # Debug/root cause analysis workflow
└── reference/                  # Core documentation
    ├── shared-patterns.md      # 5 core rules, 6-step workflow, PROMPT templates
    ├── examples.md             # Real-world usage examples
    └── troubleshooting.md      # FAQ and known limitations
```

## Core Collaboration Rules (Must Follow)

When executing any Codex collaboration:

1. **Analyze First** - Output your own preliminary analysis BEFORE calling Codex
2. **Ask When Unclear** - Request clarification from user if information is insufficient
3. **Read-Only Sandbox** - Always use `sandbox="read-only"` for `mcp__codex__codex`
4. **Diff-Based Output** - Codex provides unified diff patches or issue lists only
5. **Verify Conclusions** - Cross-validate Codex results, note disagreements

## 6-Step Standard Workflow

All commands follow this workflow defined in `reference/shared-patterns.md`:

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

PROMPT templates for each scenario are in `reference/shared-patterns.md` under "Codex 调用模板".

## Command Quick Reference

| Scenario | Command | Key Parameters |
|----------|---------|----------------|
| Code Review | `/codex-review` | path, `focus=security\|perf\|style`, `session=<id>` |
| Planning | `/codex-plan` | goal, constraints, `session=<id>` |
| Debug | `/codex-debug` | symptom/error, `session=<id>`, uses `return_all_messages=true` |

## SESSION_ID Management

- New session: omit SESSION_ID
- Continue session: pass previous SESSION_ID
- Always output session ID at end: `Codex session: <id>`
