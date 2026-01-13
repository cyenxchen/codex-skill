# Codex Collaboration Skill

> A Claude Code skill for AI-powered code review, implementation planning, and debugging through deep collaboration with Codex via MCP tools.

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/cyenxchen/codex-skill/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## ✨ Features

| Feature | Description |
|---------|-------------|
| **Code Review** | Analyze code for bugs, security vulnerabilities, performance issues, and maintainability |
| **Implementation Planning** | Break down requirements, generate implementation plans with priorities and dependencies |
| **Debug Assistance** | Locate root causes, provide debugging steps, and suggest minimal fixes |
| **Session Management** | Continue conversations across multiple turns using SESSION_ID |

## 📋 Prerequisites

Install the CodexMCP server before using this skill:

```bash
claude mcp add codex -s user --transport stdio -- uvx --from git+https://github.com/GuDaStudio/codexmcp.git codexmcp
```

## 🚀 Installation

### Option 1: Download Release (Recommended)

Download the latest release and extract to your skills directory.

```bash
# Download latest release
curl -L https://github.com/cyenxchen/codex-skill/releases/latest/download/codex-skill-v1.0.0.zip -o codex-skill.zip

# Extract to Claude skills directory
unzip codex-skill.zip -d ~/.claude/skills/codex

# Verify installation
ls ~/.claude/skills/codex/SKILL.md
```

Or manually download from [Releases](https://github.com/cyenxchen/codex-skill/releases).

### Option 2: Clone Repository

Best for developers who want to contribute or customize.

```bash
# Clone the repository
git clone https://github.com/cyenxchen/codex-skill.git

# Create symlink (using absolute path)
ln -s "$(pwd)/codex" ~/.claude/skills/codex

# Verify installation
ls -la ~/.claude/skills/codex/SKILL.md
```

### Permission Configuration (Optional)

Add the following to `~/.claude/settings.json` to allow the Codex tool:

```json
{
  "mcpServers": {
    "codex": {
      "allowed": ["mcp__codex__codex"]
    }
  }
}
```

## 📖 Usage

### Auto Trigger

The skill activates automatically when you mention "codex" or related scenarios:

- "Help me review this file with codex"
- "Let codex analyze this bug"
- "Plan this feature with codex"

### Commands Reference

| Scenario | Trigger Examples | Reference |
|----------|-----------------|-----------|
| Code Review | "Review this file", "codex review src/app.py" | `commands/codex-review.md` |
| Implementation Planning | "Plan the login feature", "Design this with codex" | `commands/codex-plan.md` |
| Debug | "Analyze this error", "Help locate this bug" | `commands/codex-debug.md` |

**Note**: In pure Skill mode, commands are not registered as slash commands. Use natural language to trigger the collaboration workflow.

## 💡 Examples

### Code Review

```
You: Review src/server.py, focus on security issues

Claude: (Codex collaboration skill activates)
Let me examine this file first...
[Calls Codex for code review]
Found 3 security issues...
```

### Implementation Planning

```
You: Plan a user login feature with OAuth support

Claude: (Codex collaboration skill activates)
Let me analyze the requirements...
[Calls Codex to generate implementation plan]
This feature can be divided into 3 phases...
```

### Debug

```
You: Analyze this error: "TypeError: Cannot read property 'id' of undefined"

Claude: (Codex collaboration skill activates)
Let me analyze this error...
[Calls Codex to locate root cause]
The root cause is...
```

### Continue Session

```
You: Continue previous session session=019ba7a9-8b52-7723-971c-90454123c715

Claude: (Continues previous Codex conversation)
OK, let's continue our discussion...
```

## 🔧 Core Constraints

- **Read-only sandbox**: All Codex calls must use `sandbox="read-only"`
- **Suggestions only**: Codex provides issue lists or unified diff patches, never makes real modifications
- **Analyze first**: Claude must provide initial analysis before calling Codex
- **Verify conclusions**: Claude cross-validates Codex results and notes disagreements

## 📚 Documentation

### Core Documents

| Document | Description |
|----------|-------------|
| [Shared Patterns](reference/shared-patterns.md) | Complete collaboration rules, workflows, and Codex call templates |
| [Examples](reference/examples.md) | Real-world use cases with full input/output flows |
| [Troubleshooting](reference/troubleshooting.md) | FAQ, error debugging, and known limitations |

### Command Details

| Command | Description |
|---------|-------------|
| [codex-review](commands/codex-review.md) | Code review command specification |
| [codex-plan](commands/codex-plan.md) | Implementation planning command specification |
| [codex-debug](commands/codex-debug.md) | Debug command specification |

## 📄 License

MIT License - Copyright (c) 2025 GuDaStudio

---

🌐 **Language / 语言**: English | [中文](README_zh.md)
