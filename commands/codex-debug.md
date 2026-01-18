---
name: codex-debug
description: Locate bug root causes by analyzing errors and code with Codex help. Identify probable causes, provide investigation steps, and suggest minimal fixes with test cases.
allowed-tools: mcp__codex__codex, Read, Grep, Glob, Bash(git:*)
argument-hint: [symptom] [error] [plan=<path>] [session=<id>]
---

# Codex Debug 定位

你正在执行 `/codex-debug`，与 Codex 协作进行问题定位和修复。

**用户参数**：`$ARGUMENTS`

**完整的协作规则和工作流程详见**：`reference/shared-patterns.md`

## 核心规则（简要）

遵循 Codex 协作的 6 条核心规则：计划检测优先、先自检、提问、安全调用、只读输出、质疑验证。

*详见 reference/shared-patterns.md 的"核心规则"章节*

## 任务特定说明

### 输入解析

从 `$ARGUMENTS` 中提取：
- **问题症状**：错误信息、异常、日志片段
- **可疑路径**：相关文件或模块
- **可选参数**：
  - `plan=<path>` - 指定排查计划文件（自动检测项目级 `.claude/plans/*.md` 和全局级 `~/.claude/plans/*.md`，`plan=none` 禁用）
  - `session=<id>` - 继续之前的会话

**排查计划用途**：若检测到排查计划文件，将读取已排查的假设和进展，避免重复验证已排除的假设。

**若信息不足，向用户追问**：
- 具体的错误信息或日志
- 复现步骤
- 环境信息
- 最近的代码变更

### Debug 重点

问题定位应输出：
1. **根因假设** - 最可能的原因及支持证据（按可能性排序）
2. **排查步骤** - 优先级排序的验证步骤
3. **修复方案** - 最小修复（unified diff patch）
4. **回归风险** - 修复可能引入的问题
5. **测试建议** - 验证修复的测试用例

### Codex 调用配置

Debug 场景特殊配置：
- `return_all_messages: true` - 需要详细的推理过程

使用 `reference/shared-patterns.md` 中的"Debug 定位场景" PROMPT 模板。

## 工作流程

遵循标准的6步 Codex 协作工作流程：

1. **确认输入** - 解析错误症状、可疑路径
2. **收集上下文** - 阅读相关代码、查看 git 变更
3. **初步分析**（必须）- 输出根因假设列表、证据、优先验证路径
4. **调用 Codex** - 使用上述 PROMPT 模板，开启详细推理
5. **质疑与验证** - 审视假设是否与代码/日志一致
6. **输出结论** - 包含根因分析、排查步骤、修复建议、测试用例、session ID

**详细的工作流程和输出格式详见**：`reference/shared-patterns.md`
