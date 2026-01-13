---
name: codex
description: Collaborate with Codex AI for code review, implementation planning, and debugging. Provides guidance for effective Claude-Codex collaboration workflow using read-only analysis and diff-based suggestions. Use when reviewing code, planning features, or debugging errors with Codex assistance.
version: 0.1.0
---

# Codex 协作指南

当用户提到 Codex 时，自动激活此协作流程。

**完整的协作规则和模式详见：**`reference/shared-patterns.md`

## 核心规则（简要）

1. **先自检再调用** - 必须先输出初步分析，再调用 Codex
2. **信息不足时提问** - 信息不明确时先追问用户
3. **安全调用** - 使用 `sandbox="read-only"`
4. **只读输出** - 仅给出 unified diff patch 或建议
5. **质疑验证** - 交叉验证 Codex 结论

*详细说明见 reference/shared-patterns.md*

## 场景识别

根据用户意图自动选择合适的协作模式：

| 场景关键词 | 推荐命令 | 说明 |
|-----------|---------|------|
| 审查、review、检查代码 | `/codex-review` | 代码审查，发现问题 |
| 规划、plan、设计、实现 | `/codex-plan` | 需求分析，生成计划 |
| debug、报错、定位、bug | `/codex-debug` | 问题定位，修复建议 |

## 工作流程（简要）

Codex 协作遵循标准的6步工作流程：

1. **确认任务类型** - 分析用户意图（代码审查/需求分析/问题定位）
2. **收集上下文** - 获取相关文件、错误信息、需求描述
3. **初步分析**（必须）- 输出你的初步判断
4. **调用 Codex** - 使用 `mcp__codex__codex` 工具
5. **质疑与验证** - 审视 Codex 结论
6. **输出结论** - 整合分析结果

**完整的工作流程和 Codex 调用模板详见：**`reference/shared-patterns.md`

## SESSION_ID 管理（简要）

- 新会话：不传 SESSION_ID
- 继续会话：传入之前的 SESSION_ID
- 输出时回显 session ID

*详细的会话管理规范见 reference/shared-patterns.md*

## 可用命令参考

以下命令的详细文档见 `commands/` 目录。注意：在纯 Skill 模式下，这些命令不会自动注册为 slash commands，建议使用自然语言触发相应的协作流程。

- **代码审查**：详见 `commands/codex-review.md` - 审查代码改动、发现问题
- **需求分析**：详见 `commands/codex-plan.md` - 规划功能、拆解任务
- **Debug 定位**：详见 `commands/codex-debug.md` - 定位问题根因、修复建议
