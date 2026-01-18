---
name: codex-review
description: Review code changes, files, or git diffs for bugs, security issues, and quality problems with Codex assistance. Focus on correctness, security, performance, and maintainability.
allowed-tools: mcp__codex__codex, Read, Grep, Glob, Bash(git:*)
argument-hint: [path|diff] [focus=security|perf|style] [plan=<path>] [session=<id>]
---

# Codex 代码审查

你正在执行 `/codex-review`，与 Codex 协作进行代码审查。

**用户参数**：`$ARGUMENTS`

**完整的协作规则和工作流程详见**：`reference/shared-patterns.md`

## 核心规则（简要）

遵循 Codex 协作的 6 条核心规则：计划检测优先、先自检、提问、安全调用、只读输出、质疑验证。

*详见 reference/shared-patterns.md 的"核心规则"章节*

## 任务特定说明

### 输入解析

从 `$ARGUMENTS` 中提取：
- **审查目标**：文件路径 | `git diff` | `git diff HEAD~N`
- **可选参数**：
  - `focus=security|perf|style` - 重点关注维度
  - `plan=<path>` - 指定计划文件（自动检测项目级 `.claude/plans/*.md` 和全局级 `~/.claude/plans/*.md`，`plan=none` 禁用）
  - `session=<id>` - 继续之前的会话

**计划文件用途**：审查时对照计划中的验收标准，检查实现是否符合预期。

**若信息不足，向用户提问后停止。**

### 审查维度

根据 `focus` 参数确定重点：
- **security**：注入、越权、敏感信息泄露
- **perf**：复杂度、资源泄漏、阻塞操作
- **style**：命名、结构、可读性
- **无 focus**：全面审查（正确性、安全性、性能、可维护性）

### Codex PROMPT 模板

使用 `reference/shared-patterns.md` 中的"代码审查场景" PROMPT 模板。

## 工作流程

遵循标准的6步 Codex 协作工作流程：

1. **确认输入** - 解析参数，验证审查目标
2. **收集上下文** - 读取代码、查看 git diff
3. **初步分析**（必须）- 输出变更范围、潜在风险点
4. **调用 Codex** - 使用上述 PROMPT 模板
5. **质疑与验证** - 审视 Codex 的问题清单
6. **输出结论** - 包含问题清单、修复建议、测试步骤、session ID

**详细的工作流程和输出格式详见**：`reference/shared-patterns.md`
