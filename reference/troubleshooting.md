# Codex 协作故障排除

本文档提供 Codex 协作命令的常见问题、解决方案和已知限制。

## 常见问题 FAQ

### 安装和配置

#### Q1: 如何安装 CodexMCP？

**答案**：
```bash
# 安装 CodexMCP MCP 服务器
claude mcp add codex -s user --transport stdio -- uvx --from git+https://github.com/GuDaStudio/codexmcp.git codexmcp

# 验证安装
claude mcp list
```

**检查点**：
- 确保输出中包含 "codex"
- 确保 Claude Code 版本 >= 0.3.0
- 确保 Codex CLI 版本 >= 0.7.0

---

#### Q2: 如何安装 Codex 协作 Skill？

**答案**：

**方法一：Release 下载（推荐）**
```bash
curl -L https://github.com/cyenxchen/codex-skill/releases/latest/download/codex-skill.zip -o codex-skill.zip
unzip codex-skill.zip -d ~/.claude/skills/
```

**方法二：Git Clone + Symlink**
```bash
git clone https://github.com/cyenxchen/codex-skill.git
ln -s $(pwd)/codex-skill ~/.claude/skills/codex
```

**检查点**：
- 确认 `~/.claude/skills/codex/` 目录存在
- 确认包含 SKILL.md 文件

---

#### Q3: 提示 "Codex MCP 工具不可用"

**原因**：
1. CodexMCP 未安装或未启动
2. Claude Code 配置中未允许 `mcp__codex__codex` 工具
3. MCP 服务器启动失败

**解决方案**：

**步骤 1**：验证 CodexMCP 安装
```bash
claude mcp list
# 应该看到 "codex" 在列表中
```

**步骤 2**：检查 MCP 服务器状态
```bash
# 手动测试 MCP 工具
claude mcp test codex
```

**步骤 3**：检查 Claude Code 配置
打开 `~/.claude/settings.json`，确保包含：
```json
{
  "mcpServers": {
    "codex": {
      "allowed": ["mcp__codex__codex"]
    }
  }
}
```

**步骤 4**：重启 Claude Code
```bash
# 重启 Claude Code 进程
# macOS/Linux: Ctrl+C 后重新启动
# Windows: 关闭命令行窗口后重新启动
```

---

### 命令使用

#### Q4: 命令没有自动触发

**症状**：提到 "codex" 但没有激活 codex-collaboration skill

**原因**：
1. Skill 未正确安装
2. Description 语义匹配未命中

**解决方案**：

**步骤 1**：验证 Skill 安装
```bash
claude plugin list
# 确认 codexmcp-commands 在列表中
```

**步骤 2**：手动调用命令
```bash
# 不依赖自动触发，直接使用命令
/codex-review src/app.py
```

**步骤 3**：使用更明确的触发词
- ✓ "帮我用 codex review 这个文件"
- ✓ "让 codex 分析一下这个 bug"
- ✗ "codex" （太简短）

---

#### Q5: 参数解析错误

**症状**：命令提示 "参数错误：缺少审查目标"

**原因**：参数格式不正确或缺少必需参数

**解决方案**：

**codex-review 正确格式**：
```bash
# 审查文件
/codex-review src/server.py

# 审查 git diff
/codex-review git diff

# 审查最近的提交
/codex-review git diff HEAD~1

# 带 focus 参数
/codex-review src/auth.py focus=security

# 继续会话
/codex-review src/utils.py session=019ba7a9-8b52-7723-971c-90454123c715
```

**codex-plan 正确格式**：
```bash
# 规划功能
/codex-plan 添加用户登录功能

# 带约束条件
/codex-plan 实现缓存系统 使用 Redis

# 继续会话
/codex-plan session=019ba7a9-8b52-7723-971c-90454123c716
```

**codex-debug 正确格式**：
```bash
# 分析错误
/codex-debug "TypeError: Cannot read property 'id' of undefined"

# 带文件路径
/codex-debug "Memory leak in server" src/worker.js

# 继续会话
/codex-debug 已经修复了监听器 session=019ba7a9-8b52-7723-971c-90454123c717
```

---

#### Q6: SESSION_ID 无法恢复会话

**症状**：使用 `session=<id>` 但 Codex 返回 "会话不存在"

**原因**：
1. SESSION_ID 格式错误
2. 会话已过期
3. SESSION_ID 不属于当前工作目录

**解决方案**：

**步骤 1**：验证 SESSION_ID 格式
```
正确格式：019ba7a9-8b52-7723-971c-90454123c715
长度：36 字符（包含连字符）
```

**步骤 2**：检查会话是否过期
Codex 会话有效期限制（通常为 24 小时）。超时后需要开启新会话。

**步骤 3**：确认工作目录一致
SESSION_ID 绑定到特定的工作目录。如果切换了项目目录，需要开启新会话。

**最佳实践**：
- 在同一个问题的多轮对话中使用同一个 SESSION_ID
- 跨项目或跨天的任务开启新会话
- 保存重要的 SESSION_ID 以便后续继续

---

#### Q7: 计划文件未被自动检测

**症状**：`.claude/plans/` 目录下有计划文件，但命令未使用

**原因**：
1. 计划文件不是有效的 Markdown 格式
2. 计划文件缺少必要的结构（标题、任务列表等）
3. 文件扩展名不是 `.md`

**解决方案**：

**步骤 1**：检查文件位置
```bash
ls -la .claude/plans/*.md
```

**步骤 2**：验证文件内容
计划文件应包含以下任一元素：
- `##` 二级标题
- `- [ ]` 或 `- [x]` 任务列表
- 关键词：「验收标准」「任务拆解」「排查进展」

**步骤 3**：手动指定计划
```bash
/codex-review src/app.py plan=.claude/plans/my-plan.md
```

---

#### Q8: 如何禁用计划自动检测

**场景**：不希望关联任何计划文件，进行独立审查

**解决方案**：
```bash
# 使用 plan=none 禁用自动检测
/codex-review src/app.py plan=none
/codex-plan 新功能需求 plan=none
/codex-debug "错误信息" plan=none
```

---

#### Q9: 计划文件过大导致超时

**症状**：计划文件超过 100 行，调用 Codex 时超时

**原因**：计划内容过长，超出上下文限制

**解决方案**：

**方案 1**：拆分计划文件
将大计划拆分为多个小计划文件，每次关联一个。

**方案 2**：精简计划内容
只保留与当前任务相关的部分（如只保留验收标准）。

**方案 3**：使用 plan=none
```bash
# 暂时禁用计划，完成后手动对照
/codex-review src/app.py plan=none
```

---

### 输出和结果

#### Q10: Codex 调用超时

**症状**：命令卡住很久后返回 "调用超时"

**原因**：
1. 代码量过大
2. 网络问题
3. Codex 服务器负载高

**解决方案**：

**方案 1**：缩小审查范围
```bash
# ❌ 审查整个项目（太大）
/codex-review .

# ✅ 审查单个文件
/codex-review src/server.py

# ✅ 审查最近的改动
/codex-review git diff
```

**方案 2**：检查网络连接
```bash
# 测试网络
ping google.com

# 检查代理设置
echo $HTTP_PROXY
echo $HTTPS_PROXY
```

**方案 3**：稍后重试
等待 5-10 分钟后重试，可能是 Codex 服务器负载高。

---

#### Q11: Codex 输出不完整

**症状**：Codex 的结论被截断，或只返回了部分结果

**原因**：
1. 输出超过了 token 限制
2. 网络传输中断
3. MCP 工具返回被截断

**解决方案**：

**方案 1**：分批处理
```bash
# 将大文件分成多个小部分审查
/codex-review src/server.py  # 先审查这个
/codex-review src/client.py  # 再审查这个
```

**方案 2**：使用 SESSION_ID 继续
```bash
# 第一轮
/codex-review src/large_file.py
# 输出：Codex session: 019ba7a9...

# 继续获取更多细节
/codex-review 能否详细说明第3个问题？ session=019ba7a9...
```

**方案 3**：简化 PROMPT
减少在 PROMPT 中包含的上下文，让 Codex 先给出概要，再深入细节。

---

#### Q12: Codex 的建议不准确

**症状**：Codex 指出的问题不存在，或建议的修复不合理

**原因**：
1. Codex 基于的上下文不完整
2. Codex 对特定领域的理解有限
3. 误判或幻觉

**解决方案**：

**方案 1**：质疑与验证（核心规则）
```
✅ Claude 必须对 Codex 结论进行审视
✅ 明确指出不同意的点
✅ 要求更多证据支持

示例：
"Codex 提到的性能问题不成立，代码复杂度为 O(1)，无需优化。"
```

**方案 2**：提供更多上下文
```bash
# ❌ 只给错误信息
/codex-debug "TypeError"

# ✅ 包含堆栈、文件、复现步骤
/codex-debug "TypeError at src/api.js:45 when clicking delete button.
Recent change: modified getUserData function."
```

**方案 3**：多轮对话澄清
```bash
# 第一轮
/codex-review src/auth.py

# 第二轮（针对误判）
/codex-review 第2个问题似乎不对，userId 是必需参数 session=<id>
```

**重要**：不要盲目接受 Codex 的建议，始终保持批判性思维。

---

## 错误排查流程

### Codex 调用失败的排查步骤

```
1. 检查 CodexMCP 是否安装
   ↓
   claude mcp list
   ↓
   是否包含 "codex"?
   ├─ 否 → 安装 CodexMCP（Q1）
   └─ 是 → 继续

2. 检查 Claude Code 配置
   ↓
   ~/.claude/settings.json
   ↓
   是否允许 mcp__codex__codex?
   ├─ 否 → 添加 allow 项（Q3）
   └─ 是 → 继续

3. 测试 MCP 工具
   ↓
   claude mcp test codex
   ↓
   是否成功?
   ├─ 否 → 查看错误日志，检查 Codex CLI 版本
   └─ 是 → 继续

4. 检查命令参数
   ↓
   是否提供了必需参数?
   ├─ 否 → 参考 Q5 正确格式
   └─ 是 → 继续

5. 检查网络连接
   ↓
   ping google.com
   ↓
   网络正常?
   ├─ 否 → 检查代理、防火墙设置
   └─ 是 → 继续

6. 缩小任务范围
   ↓
   是否审查/分析的范围过大?
   ├─ 是 → 分批处理（Q7）
   └─ 否 → 联系支持
```

---

## 已知限制

### Codex 能力限制

1. **代码量限制**
   - 单次调用不建议超过 1000 行代码
   - 超大文件建议分批审查

2. **上下文窗口**
   - SESSION_ID 可以保留历史对话
   - 但每次调用的 token 数仍有限制

3. **领域知识**
   - 对常见语言和框架支持较好
   - 对小众技术栈可能理解有限

### Claude Code Skill 限制

1. **跨文件引用**
   - 各命令文件可能无法直接加载 reference/ 中的内容
   - 因此保留了简化版的规则说明

2. **自动触发**
   - 依赖语义匹配，不是100%准确
   - 可以手动使用 `/codex-*` 命令

3. **参数解析**
   - 基于简单的字符串匹配
   - 复杂的参数格式可能解析失败

### MCP 协议限制

1. **Sandbox 限制**
   - 必须使用 `sandbox="read-only"`
   - Codex 无法直接修改文件系统

2. **输出格式**
   - 依赖 Codex CLI 的 JSON 输出
   - 输出格式变更可能影响解析

3. **会话持久化**
   - SESSION_ID 有有效期限制
   - 超时后无法恢复

---

## 调试技巧

### 启用详细日志

**方法 1**：使用 `--verbose` 标志
```bash
claude --verbose
```

**方法 2**：查看 MCP 日志
```bash
# 查看 CodexMCP 日志
tail -f ~/.claude/logs/mcp-codex.log
```

### 测试 Codex CLI

直接测试 Codex CLI 是否正常工作：
```bash
# 测试 Codex exec 命令
codex exec --json "分析这段代码：print('Hello')"
```

### 隔离问题

1. **测试最小案例**
   ```bash
   # 最简单的调用
   /codex-review README.md
   ```

2. **逐步增加复杂度**
   ```bash
   # 添加 focus 参数
   /codex-review README.md focus=style
   ```

3. **确定失败点**
   - 命令解析失败？
   - MCP 调用失败？
   - Codex 返回失败？
   - 输出解析失败？

---

## 获取帮助

### 报告 Bug

如果遇到 Bug，请提供以下信息：

1. **环境信息**
   ```bash
   claude --version
   codex --version
   claude mcp list
   ```

2. **复现步骤**
   - 完整的命令
   - 输入的参数
   - 预期结果 vs 实际结果

3. **错误日志**
   - 完整的错误信息
   - MCP 日志（如果有）

4. **提交到**
   - GitHub Issues: https://github.com/GuDaStudio/codexmcp/issues

### 社区资源

- **GitHub**: https://github.com/GuDaStudio/codexmcp
- **文档**: 查看 `skill/reference/` 目录下的文档
- **示例**: 查看 `skill/reference/examples.md`

---

## 性能优化建议

### 减少 Codex 调用延迟

1. **缩小审查范围** - 审查单个文件而非整个项目
2. **使用 focus 参数** - 只关注特定维度
3. **提供精确的上下文** - 包含相关文件路径和错误信息

### 提高准确性

1. **提供完整的错误堆栈** - Debug 时包含完整日志
2. **说明最近的变更** - 帮助 Codex 快速定位问题
3. **多轮对话澄清** - 使用 SESSION_ID 逐步深入

### 最佳工作流程

1. **先自己分析** - 遵循"先自检再调用"规则
2. **质疑 Codex 结论** - 不盲目接受建议
3. **分批处理大任务** - 将复杂任务拆分为多个小步骤
4. **保存 SESSION_ID** - 便于后续继续讨论

---

## 参考资料

- [shared-patterns.md](./shared-patterns.md) - 完整的协作规则和工作流程
- [examples.md](./examples.md) - 真实的使用案例
- [README.md](../README.md) - 插件安装和配置
