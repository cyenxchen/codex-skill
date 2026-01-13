---
description: Analyze requirements and generate implementation plans through Codex collaboration. Break down requirements into tasks, assess risks, and define acceptance criteria.
allowed-tools: mcp__codex__codex, Read, Grep, Glob, Bash(git:*)
argument-hint: [goal] [constraints] [session=<id>]
---

# Codex 需求分析与实现计划

你正在执行 `/codex-plan`，与 Codex 协作进行需求分析和实现规划。

**用户参数**：`$ARGUMENTS`

**完整的协作规则和工作流程详见**：`reference/shared-patterns.md`

## 核心规则（简要）

遵循 Codex 协作的5条核心规则：先自检、提问、安全调用、只读输出、质疑验证。

*详见 reference/shared-patterns.md 的"核心规则"章节*

## 任务特定说明

### 输入解析

从 `$ARGUMENTS` 中提取：
- **需求目标**：要实现的功能或解决的问题
- **约束条件**：技术栈、性能要求、兼容性等
- **可选参数**：`session=<id>` - 继续之前的会话

**若需求不清晰，向用户提出澄清问题**：
- 要解决什么问题？
- 期望的行为是什么？
- 有什么约束或限制？

### 规划重点

需求分析应输出：
1. **需求拆解** - 具体的功能点列表
2. **实施计划** - 阶段划分、任务列表（含优先级和依赖）
3. **风险评估** - 潜在风险和应对策略
4. **验收标准** - 如何判断需求完成

### Codex PROMPT 模板

使用 `reference/shared-patterns.md` 中的"需求分析场景" PROMPT 模板。

## 工作流程

遵循标准的6步 Codex 协作工作流程：

1. **确认输入** - 解析需求目标和约束条件
2. **收集上下文** - 了解现有架构、相关模块
3. **初步分析**（必须）- 输出需求拆解、关键假设、主要风险
4. **调用 Codex** - 使用上述 PROMPT 模板
5. **质疑与验证** - 审视任务拆分、依赖关系、风险评估
6. **输出计划** - 包含需求拆解、实施计划、风险评估、验收标准、session ID

**详细的工作流程和输出格式详见**：`reference/shared-patterns.md`
