# Codex 协作技能

> 为 Claude Code 提供 AI 驱动的代码审查、实现规划和调试功能，通过 MCP 工具与 Codex 进行深度协作。

[![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)](https://github.com/GuDaStudio/codexmcp)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## ✨ 功能特性

| 功能 | 描述 |
|------|------|
| **代码审查** | 分析代码中的 Bug、安全漏洞、性能问题和可维护性问题 |
| **实现规划** | 拆解需求，生成包含优先级和依赖关系的实现计划 |
| **Debug 辅助** | 定位根因，提供排查步骤，建议最小修复方案 |
| **会话管理** | 使用 SESSION_ID 跨多轮对话保持上下文 |

## 📋 前置条件

使用此技能前，需要先安装 CodexMCP 服务器：

```bash
claude mcp add codex -s user --transport stdio -- uvx --from git+https://github.com/GuDaStudio/codexmcp.git codexmcp
```

## 🚀 安装方式

### 方式一：软链接（推荐）

适合需要自动同步更新的开发者。

```bash
# 克隆仓库
git clone https://github.com/GuDaStudio/codexmcp.git
cd codexmcp

# 创建软链接（使用绝对路径）
ln -s "$(pwd)/skill" ~/.claude/skills/codex

# 验证安装
ls -la ~/.claude/skills/codex/SKILL.md
```

### 方式二：直接复制

适合需要稳定离线副本的用户。

```bash
# 克隆仓库
git clone https://github.com/GuDaStudio/codexmcp.git

# 复制到 Claude skills 目录
cp -r codexmcp/skill ~/.claude/skills/codex
```

### 权限配置（可选）

在 `~/.claude/settings.json` 中添加以下内容以允许 Codex 工具：

```json
{
  "mcpServers": {
    "codex": {
      "allowed": ["mcp__codex__codex"]
    }
  }
}
```

## 📖 使用方法

### 自动触发

当你提到 "codex" 或相关场景时，技能会自动激活：

- "帮我用 codex 审查一下这个文件"
- "让 codex 分析一下这个 bug"
- "和 codex 一起规划这个功能"

### 命令参考

| 场景 | 触发示例 | 参考文档 |
|------|---------|---------|
| 代码审查 | "审查这个文件"、"codex review src/app.py" | `commands/codex-review.md` |
| 实现规划 | "规划登录功能"、"和 codex 一起设计这个方案" | `commands/codex-plan.md` |
| Debug 定位 | "分析这个错误"、"帮我定位这个 bug" | `commands/codex-debug.md` |

**注意**：在纯 Skill 模式下，命令不会注册为 slash commands。请使用自然语言触发协作流程。

## 💡 使用示例

### 代码审查

```
你：帮我审查一下 src/server.py，重点关注安全问题

Claude：（Codex 协作技能激活）
我先来看看这个文件...
[调用 Codex 进行代码审查]
发现了 3 个安全问题...
```

### 实现规划

```
你：规划一下用户登录功能，需要支持 OAuth

Claude：（Codex 协作技能激活）
让我先分析一下需求...
[调用 Codex 生成实现计划]
这个功能可以分为 3 个阶段...
```

### Debug 定位

```
你：帮我分析这个错误："TypeError: Cannot read property 'id' of undefined"

Claude：（Codex 协作技能激活）
让我分析一下这个错误...
[调用 Codex 定位问题根因]
根因是...
```

### 继续会话

```
你：继续上次的会话 session=019ba7a9-8b52-7723-971c-90454123c715

Claude：（继续之前的 Codex 对话）
好的，我们继续讨论...
```

## 🔧 核心约束

- **只读沙箱**：所有 Codex 调用必须使用 `sandbox="read-only"`
- **仅提供建议**：Codex 仅提供问题清单或 unified diff patch，不做真实修改
- **先分析后调用**：Claude 必须先给出初步分析，再调用 Codex
- **验证结论**：Claude 需交叉验证 Codex 结果，指出不同意或不确定的点

## 📚 文档资源

### 核心文档

| 文档 | 描述 |
|------|------|
| [共享模式](reference/shared-patterns.md) | 完整的协作规则、工作流程和 Codex 调用模板 |
| [使用示例](reference/examples.md) | 真实使用案例，包含完整的输入输出流程 |
| [故障排除](reference/troubleshooting.md) | 常见问题 FAQ、错误排查和已知限制 |

### 命令详情

| 命令 | 描述 |
|------|------|
| [codex-review](commands/codex-review.md) | 代码审查命令规范 |
| [codex-plan](commands/codex-plan.md) | 实现规划命令规范 |
| [codex-debug](commands/codex-debug.md) | Debug 定位命令规范 |

## 📄 许可证

MIT License - Copyright (c) 2025 GuDaStudio

---

🌐 **Language / 语言**: [English](README.md) | 中文
