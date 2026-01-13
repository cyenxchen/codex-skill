# Codex 协作使用示例

本文档提供真实的 Codex 协作命令使用案例，包含输入、输出和完整的工作流程展示。

## `/codex-review` 代码审查示例

### 示例 1：审查 git diff

**用户输入**：
```
/codex-review git diff
```

**场景**：审查当前未提交的代码改动

**工作流程**：

1. **确认输入** - 解析到审查目标为 `git diff`
2. **收集上下文** - 运行 `git diff` 获取改动内容
3. **初步分析**（Claude 输出）：
   ```
   初步分析：
   - 改动范围：修改了 src/server.py 中的参数验证逻辑
   - 潜在风险：新增的 get() 调用可能缺少默认值处理
   - 重点关注：边界条件和异常处理
   ```
4. **调用 Codex** - 使用代码审查 PROMPT 模板
5. **质疑与验证**（Claude 输出）：
   ```
   Codex 指出了 3 个问题，我同意其中 2 个：
   - ✓ 缺少默认值处理 - 确实存在
   - ✓ 边界条件未检查 - 确实需要添加
   - ? 性能问题 - 代码复杂度为 O(1)，不存在性能问题
   ```
6. **输出结论**：
   ```
   ## 结论概述
   发现 2 个 Medium 级别问题，需要添加默认值处理和边界条件检查。

   ## 问题清单

   ### Medium: 缺少默认值处理
   **位置**：src/server.py:45
   **问题**：`data.get('value')` 在 key 不存在时返回 None，可能导致后续代码崩溃
   **风险**：若客户端未发送 value 字段，服务器会抛出 TypeError

   **建议修复**：
   ```diff
   --- a/src/server.py
   +++ b/src/server.py
   @@ -42,7 +42,7 @@ def process_data(data):
   -    value = data.get('value')
   +    value = data.get('value', 0)
        return calculate(value)
   ```

   ### Medium: 边界条件未检查
   **位置**：src/server.py:48
   **问题**：未检查 value 是否为负数
   **风险**：负数输入可能导致 calculate() 函数返回错误结果

   **建议修复**：
   ```diff
   --- a/src/server.py
   +++ b/src/server.py
   @@ -45,6 +45,8 @@ def process_data(data):
        value = data.get('value', 0)
   +    if value < 0:
   +        raise ValueError("value must be non-negative")
        return calculate(value)
   ```

   ## 质疑与验证
   Codex 提到的性能问题不成立，代码复杂度为 O(1)，无需优化。

   ## 建议测试
   1. 测试缺少 value 字段的请求
   2. 测试 value 为负数的请求
   3. 测试 value 为 0 的边界情况

   Codex session: 019ba7a9-8b52-7723-971c-90454123c715
   ```

---

### 示例 2：审查特定文件，关注安全问题

**用户输入**：
```
/codex-review src/auth.py focus=security
```

**场景**：审查认证模块的安全性

**简化输出**：
```
## 结论概述
发现 1 个 Critical 和 2 个 High 安全问题，涉及 SQL 注入、密码存储和会话管理。

## 问题清单

### Critical: SQL 注入风险
**位置**：src/auth.py:23
**问题**：直接拼接用户输入到 SQL 查询
**风险**：攻击者可通过恶意输入执行任意 SQL 命令

**建议修复**：使用参数化查询
```diff
--- a/src/auth.py
+++ b/src/auth.py
@@ -20,7 +20,7 @@ def login(username, password):
-    query = f"SELECT * FROM users WHERE username='{username}'"
+    query = "SELECT * FROM users WHERE username=?"
-    result = db.execute(query)
+    result = db.execute(query, (username,))
```

### High: 明文存储密码
...（省略详细内容）

Codex session: 019ba7a9-8b52-7723-971c-90454123c716
```

---

### 示例 3：继续之前的审查会话

**用户输入**：
```
/codex-review src/auth.py session=019ba7a9-8b52-7723-971c-90454123c716
```

**场景**：基于上次审查继续讨论

**简化输出**：
```
继续上次会话，已修复 SQL 注入问题，现在审查其他部分...

Codex session: 019ba7a9-8b52-7723-971c-90454123c716 (继续)
```

---

## `/codex-plan` 需求分析示例

### 示例 1：规划用户登录功能

**用户输入**：
```
/codex-plan 实现用户登录功能，需要支持 OAuth 和邮箱登录
```

**工作流程**：

1. **确认输入** - 目标明确：实现双登录方式
2. **收集上下文** - 阅读现有的认证相关代码
3. **初步分析**（Claude 输出）：
   ```
   初步分析：
   - 需求拆解：OAuth 登录、邮箱登录、会话管理
   - 关键假设：使用 JWT 作为会话令牌
   - 主要风险：OAuth 回调处理、密码安全存储
   - 初步思路：创建统一的认证接口，分别实现两种登录策略
   ```
4. **调用 Codex** - 使用需求分析 PROMPT 模板
5. **质疑与验证**（Claude 输出）：
   ```
   Codex 的计划合理，但有几点需要澄清：
   - ✓ 任务拆分清晰
   - ? OAuth 提供商未明确（Google? GitHub? 两者都支持?）
   - ? 邮箱验证流程未提及
   ```
6. **输出计划**：
   ```
   ## 结论概述
   分3个阶段实现双登录方式，预计涉及 5 个新文件和 3 个修改。

   ## 需求拆解

   1. **OAuth 登录**
      - OAuth 授权流程（redirects, callback）
      - 支持 Google 和 GitHub 两种提供商
      - 获取用户信息并创建/更新账户

   2. **邮箱登录**
      - 用户注册（邮箱 + 密码）
      - 邮箱验证（发送验证链接）
      - 登录验证（密码哈希比对）

   3. **会话管理**
      - JWT 令牌生成和验证
      - 刷新令牌机制
      - 登出处理

   ## 实施计划

   ### 第一阶段：基础架构
   - [ ] 创建 `src/auth/interface.py` - 统一认证接口
   - [ ] 创建 `src/auth/jwt_manager.py` - JWT 工具
   - [ ] 创建 `src/auth/models.py` - 用户数据模型

   ### 第二阶段：OAuth 实现
   - [ ] 创建 `src/auth/oauth_provider.py` - OAuth 抽象类
   - [ ] 实现 Google OAuth 登录
   - [ ] 实现 GitHub OAuth 登录
   - [ ] 添加 OAuth 回调路由

   ### 第三阶段：邮箱登录实现
   - [ ] 创建 `src/auth/email_auth.py` - 邮箱登录逻辑
   - [ ] 实现用户注册接口
   - [ ] 实现邮箱验证流程
   - [ ] 实现登录验证

   **依赖关系**：
   - 第二、三阶段依赖第一阶段完成
   - 第二、三阶段可并行开发

   ## 风险与未知

   ### 已识别风险
   1. OAuth 回调处理的安全性（CSRF 攻击）
   2. 密码存储需使用 bcrypt 或 argon2
   3. 邮箱验证链接的过期时间管理

   ### 待澄清问题
   - OAuth 提供商是否只支持 Google 和 GitHub？
   - 邮箱验证是否必需（还是可选）？
   - 是否需要"忘记密码"功能？

   ## 验收标准
   - [ ] 用户可以通过 Google 登录
   - [ ] 用户可以通过 GitHub 登录
   - [ ] 用户可以通过邮箱注册并登录
   - [ ] 邮箱注册需验证邮箱
   - [ ] 所有登录方式都能正确生成 JWT
   - [ ] 通过安全审计（无 SQL 注入、密码安全存储）

   ## 质疑与验证
   Codex 的计划较为完善，但需要用户澄清上述待确认问题。

   Codex session: 019ba7a9-8b52-7723-971c-90454123c717
   ```

---

## `/codex-debug` 问题定位示例

### 示例 1：分析 TypeError

**用户输入**：
```
/codex-debug "TypeError: Cannot read property 'id' of undefined"
```

**工作流程**：

1. **确认输入** - 错误信息明确，但缺少上下文
2. **追问用户**（Claude 输出）：
   ```
   需要更多信息来定位问题：
   - 完整的错误堆栈（包含文件名和行号）
   - 触发错误的操作步骤
   - 最近的代码变更（如果有）
   ```

**用户补充**：
```
/codex-debug "TypeError at src/api.js:45 - Cannot read property 'id' of undefined.
发生在用户点击删除按钮时。最近修改了 getUserData 函数。"
```

3. **收集上下文** - 阅读 `src/api.js:45` 及相关函数
4. **初步分析**（Claude 输出）：
   ```
   根因假设（按可能性排序）：
   1. getUserData 返回了 undefined（80% 可能）
      - 证据：最近修改了此函数
      - 缺少证据：未看到具体的修改内容
   2. 用户对象未正确初始化（15% 可能）
   3. 异步问题导致数据未加载（5% 可能）

   优先验证：检查 getUserData 的返回值处理
   ```
5. **调用 Codex** - 使用 Debug PROMPT 模板，开启详细推理
6. **质疑与验证**（Claude 输出）：
   ```
   Codex 定位到问题：getUserData 在用户未登录时返回 undefined，
   但调用处未检查。我同意这个诊断，与错误信息一致。
   ```
7. **输出结论**：
   ```
   ## 结论概述
   根因是 getUserData 在用户未登录时返回 undefined，调用处未检查导致访问 undefined.id 抛出错误。

   ## 根因分析

   **最可能的根因**：getUserData 函数缺少登录状态检查

   **支持证据**：
   - 错误发生在 `src/api.js:45`，代码为 `const userId = user.id`
   - getUserData 在用户未登录时返回 undefined
   - 最近的修改移除了默认返回值

   **排除的假设**：
   - 异步问题：错误堆栈显示为同步调用
   - 初始化问题：其他地方调用正常

   ## 排查步骤

   1. 验证用户未登录时 getUserData 的返回值
   2. 检查是否有其他类似的未检查调用
   3. 测试修复后的登录和未登录场景

   ## 修复建议

   **最小修复**（src/api.js）：
   ```diff
   --- a/src/api.js
   +++ b/src/api.js
   @@ -42,7 +42,10 @@ function deleteUser() {
        const user = getUserData()
   -    const userId = user.id
   +    if (!user) {
   +        throw new Error("User not logged in")
   +    }
   +    const userId = user.id
        api.delete(`/users/${userId}`)
    }
   ```

   **回归风险**：修复后，未登录用户会看到明确的错误提示而非崩溃

   ## 建议测试

   测试用例：
   1. **已登录用户删除** - 应成功调用 API
   2. **未登录用户删除** - 应抛出 "User not logged in" 错误
   3. **其他 getUserData 调用** - 验证是否也需要添加检查

   ## 质疑与验证
   Codex 的诊断准确，修复方案合理。建议全局搜索 getUserData 调用，确保所有地方都有检查。

   Codex session: 019ba7a9-8b52-7723-971c-90454123c718
   ```

---

## SESSION_ID 继续会话示例

### 场景：多轮 Debug 对话

**第一轮**：
```
用户: /codex-debug "Memory leak in server"
Claude: [分析并输出]
Codex session: 019ba7a9-8b52-7723-971c-90454123c719
```

**第二轮**（继续会话）：
```
用户: /codex-debug 已经修复了事件监听器，还有其他可能的原因吗？session=019ba7a9-8b52-7723-971c-90454123c719
Claude: [基于上次会话继续分析]
Codex session: 019ba7a9-8b52-7723-971c-90454123c719 (继续)
```

**第三轮**（继续会话）：
```
用户: /codex-debug 帮我检查一下数据库连接池的配置 session=019ba7a9-8b52-7723-971c-90454123c719
Claude: [在同一会话中分析连接池配置]
Codex session: 019ba7a9-8b52-7723-971c-90454123c719 (继续)
```

**优势**：
- Codex 保留了之前的上下文（已经讨论过的假设、已排除的原因）
- 无需重复说明问题背景
- 可以逐步深入分析

---

## 综合场景：完整的功能开发流程

### 1. 需求分析
```
/codex-plan 添加用户点赞功能，支持点赞文章和评论
```

### 2. 代码实现
（由开发者手动实现计划中的任务）

### 3. 代码审查
```
/codex-review src/likes.py focus=security
```

### 4. 问题修复
（根据审查结果修复问题）

### 5. Bug 定位
```
/codex-debug "点赞后计数不更新"
```

### 6. 继续调试
```
/codex-debug 已经修复了 Redis 缓存，但计数还是不准确 session=<id>
```

### 7. 最终审查
```
/codex-review git diff
```

---

## 使用技巧总结

### 最佳实践

1. **提供足够的上下文**
   - ✓ `/codex-review src/server.py focus=security`
   - ✗ `/codex-review` （缺少审查目标）

2. **利用 focus 参数**
   - 安全审查：`focus=security`
   - 性能优化：`focus=perf`
   - 代码风格：`focus=style`

3. **善用 SESSION_ID**
   - 复杂任务分多轮进行
   - 保留上下文避免重复
   - 逐步深入分析

4. **明确描述问题症状**
   - ✓ "TypeError at line 45 when deleting user"
   - ✗ "有个错误" （太模糊）

### 常见错误

1. **跳过初步分析**
   - ❌ 直接调用 Codex 而不思考
   - ✅ 先输出自己的判断

2. **盲目接受建议**
   - ❌ 不验证就应用 Codex 的修复
   - ✅ 交叉验证并指出疑问

3. **信息不足时继续**
   - ❌ 猜测用户意图
   - ✅ 明确提问获取更多信息

---

## 参考资料

- [shared-patterns.md](./shared-patterns.md) - 完整的协作规则和工作流程
- [troubleshooting.md](./troubleshooting.md) - 常见问题排查
- [README.md](../README.md) - 插件安装和配置
