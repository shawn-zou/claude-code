# 02-Hookify 轻量级 Hook 配置

## 🎯 本章导读

Hookify 是 Claude Code 插件系统中**最强大且易用**的 Hook 配置方式。它让你无需编写复杂的 Python/Shell 脚本，只需简单的 markdown 文件就能创建自定义 Hook 规则。

**你将学到**：
- ✅ Hookify 的设计理念与核心优势
- ✅ .local.md 文件的完整格式
- ✅ YAML frontmatter 配置详解
- ✅ 正则表达式 pattern 语法
- ✅ 5 种 event 类型的使用场景
- ✅ action: warn vs block 的区别
- ✅ 实战：创建 10 个常用规则

**前置知识**：
- 了解 Hook 系统基础（参考 02-Hook 系统基础.md）
- 基本的正则表达式知识

**预计耗时**：1.5-2 小时

---

## 一、Hookify 简介

### 1.1 什么是 Hookify？

**Hookify** 是一个基于 markdown 配置文件的轻量级 Hook 系统，它允许你通过简单的配置文件来定义 Hook 规则，而无需编写代码。

**类比理解**：

```
传统 Hook (security-guidance)          Hookify 轻量级 Hook
    ├── hooks.json 配置                     ├── .local.md 配置文件
    ├── Python 脚本执行                     ├── 声明式规则定义
    ├── 需要编程能力                        ├── 无需编程
    ├── 修改后需重启                        ├── 修改后立即生效
    └── 适合复杂逻辑                        └── 适合简单规则
```

---

### 1.2 为什么需要 Hookify？

#### 场景对比

**传统 Hook 开发流程**：

```python
# 1. 创建 Python 脚本
# plugins/my-hook/hooks/check_something.py

#!/usr/bin/env python3
import json
import sys

def check_security(content):
    # 编写检查逻辑
    if "dangerous_pattern" in content:
        print("⚠️ Security warning!", file=sys.stderr)
        return False
    return True

if __name__ == "__main__":
    input_data = json.load(sys.stdin)
    result = check_security(input_data['content'])
    sys.exit(0 if result else 1)
```

```json
// 2. 配置 hooks.json
{
  "PreToolUse": [{
    "matcher": "Edit|Write",
    "hooks": [{
      "type": "command",
      "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/check_something.py",
      "timeout": 10
    }]
  }]
}
```

**Hookify 开发流程**：

```markdown
<!-- 1. 创建 .local.md 文件 -->
<!-- .claude/hookify.dangerous-pattern.local.md -->

---
name: block-dangerous-pattern
enabled: true
event: file
pattern: dangerous_pattern
action: block
---

⚠️ **Dangerous pattern detected!**

This code contains unsafe patterns. Please review.
```

**对比结果**：
- ✅ Hookify **无需编程**，只需写配置文件
- ✅ Hookify **立即生效**，无需重启
- ✅ Hookify **易于维护**，markdown 格式清晰
- ✅ Hookify **易于分享**，配置文件小巧

---

### 1.3 Hookify 的核心优势

| 特性 | 传统 Hook | Hookify | 优势 |
|------|----------|---------|------|
| **开发门槛** | 需要编程能力 | 会写 markdown 即可 | ⭐⭐⭐⭐⭐ |
| **开发效率** | 编写 + 调试脚本 | 直接写配置 | ⭐⭐⭐⭐⭐ |
| **修改成本** | 修改脚本 + 重启 | 修改配置 + 立即生效 | ⭐⭐⭐⭐⭐ |
| **灵活性** | 高（可编写任意逻辑） | 中（基于模式匹配） | ⭐⭐⭐ |
| **适用场景** | 复杂检查、外部 API | 简单规则、快速原型 | - |

**结论**：
- 🔥 **简单规则** → 优先使用 Hookify
- 🔥 **复杂逻辑** → 使用传统 Hook
- 🔥 **快速验证** → Hookify 原型，确认后再用传统 Hook

---

## 二、.local.md 文件格式详解

### 2.1 文件位置与命名

**标准位置**：
```
你的项目目录/
└── .claude/
    ├── hookify.rule-name.local.md
    ├── hookify.another-rule.local.md
    └── ...
```

**命名规则**：
```
hookify.<rule-name>.local.md
   │         │          │
   │         │          └── 固定后缀
   │         └── 自定义规则名（kebab-case）
   └── 固定前缀
```

**示例**：
```bash
.claude/hookify.dangerous-rm.local.md      # 危险 rm 命令拦截
.claude/hookify.console-log.local.md       # console.log 检测
.claude/hookify.api-key-detection.local.md # API key 检测
.claude/hookify.require-tests.local.md     # 要求运行测试
```

---

### 2.2 文件结构

**.local.md 文件 = YAML frontmatter + Markdown body**

```markdown
---
name: rule-name              # 规则名称（必需）
enabled: true                # 是否启用（可选，默认 true）
event: bash                  # 触发事件类型（必需）
pattern: your-regex          # 简单模式（可选，legacy）
action: warn                 # 动作：warn 或 block（可选，默认 warn）
tool_matcher: Edit|Write     # 工具匹配器（可选）
conditions:                  # 复杂条件列表（可选，替代 pattern）
  - field: file_path
    operator: regex_match
    pattern: \.env$
---

这里是警告消息内容，支持 **Markdown 格式**。

可以包含：
- 列表
- 代码块
- 链接
- 格式化文本
```

---

### 2.3 YAML Frontmatter 字段详解

#### ① name（必需）

**作用**：规则的唯一标识符

**要求**：
- 使用 kebab-case（小写字母 + 连字符）
- 要有描述性，能看出规则用途
- 不能重复

**示例**：
```yaml
name: block-dangerous-rm          # ✅ 好：清晰描述
name: dangerous-command           # ✅ 可以
name: rule1                       # ❌ 不好：无意义
name: Block Dangerous RM          # ❌ 错误：应该小写
```

---

#### ② enabled（可选）

**作用**：控制规则是否生效

**取值**：
- `true` - 启用规则（默认值）
- `false` - 禁用规则（临时关闭）

**示例**：
```yaml
enabled: true    # ✅ 启用
enabled: false   # ✅ 禁用（临时关闭，方便调试）
# 不写这个字段 → 默认启用
```

**使用场景**：
```markdown
<!-- 调试时临时禁用某个规则 -->
---
name: strict-check
enabled: false  # 先禁用，调试完再启用
event: bash
pattern: dangerous
---

Warning message...
```

---

#### ③ event（必需）

**作用**：指定规则在什么事件下触发

**取值**：5 种事件类型

| Event | 触发时机 | 监控对象 | 示例 |
|-------|---------|---------|------|
| **bash** | Bash 工具执行 | 命令行字符串 | `rm -rf /tmp` |
| **file** | 文件编辑工具 | Edit, Write, MultiEdit | 修改代码文件 |
| **stop** | Claude 想停止时 | 会话状态 | 完成任务想退出 |
| **prompt** | 用户提交问题时 | 用户输入 | 提问内容 |
| **all** | 所有事件 | 全部 | 全局规则 |

**详细说明**：

##### bash 事件

```yaml
event: bash
```

**触发时机**：Claude 调用 Bash 工具执行命令时

**监控字段**：`command`（命令字符串）

**示例规则**：
```markdown
---
name: block-rm-rf
enabled: true
event: bash
pattern: rm\s+-rf
action: block
---

🛑 **危险操作！**

`rm -rf` 是危险命令，可能导致数据丢失。
请使用更安全的方式删除文件。
```

**实际效果**：
```
用户：删除 /tmp/test 目录
Claude：准备执行 → rm -rf /tmp/test
Hookify：检测到 "rm -rf" → 触发规则 → 显示警告并阻止
```

---

##### file 事件

```yaml
event: file
```

**触发时机**：Claude 使用 Edit、Write、MultiEdit 工具修改文件时

**监控字段**：
- `file_path` - 文件路径
- `new_text` - 新添加的内容
- `old_text` - 被替换的内容（仅 Edit）
- `content` - 完整文件内容（仅 Write）

**示例规则**：
```markdown
---
name: detect-console-log
enabled: true
event: file
pattern: console\.log\(
action: warn
---

🐛 **Debug 代码检测！**

发现 `console.log`，记得在提交前删除。
```

**实际效果**：
```
Claude：编辑 src/app.ts，添加 console.log("test")
Hookify：检测到 "console.log(" → 触发规则 → 显示警告但允许继续
```

---

##### stop 事件

```yaml
event: stop
```

**触发时机**：Claude 完成任务想要停止会话时

**监控字段**：使用 `transcript`（会话记录）等通用字段

**特殊之处**：
- 用于**完成度检查**
- 可以**阻止 Claude 停止**，直到满足条件

**示例规则**：
```markdown
---
name: require-tests-before-stop
enabled: true
event: stop
action: block
conditions:
  - field: transcript
    operator: not_contains
    pattern: npm test|pytest|cargo test
---

🛑 **测试未运行！**

在停止之前，请先运行测试确保功能正常。
请执行：npm test
```

**实际效果**：
```
Claude：任务完成，准备停止
Hookify：检查会话记录 → 没有测试命令 → 触发规则 → 阻止停止
Claude：看到提示 → 运行测试 → 再次尝试停止
Hookify：检测到测试命令 → 允许停止
```

---

##### prompt 事件

```yaml
event: prompt
```

**触发时机**：用户提交问题或指令时

**监控字段**：`user_prompt`（用户输入内容）

**示例规则**：
```markdown
---
name: detect-security-question
enabled: true
event: prompt
pattern: 如何绕过认证
action: warn
---

⚠️ **敏感问题检测！**

我无法提供绕过安全认证的方法。
建议学习合法的安全防护知识。
```

---

##### all 事件

```yaml
event: all
```

**触发时机**：所有事件都触发

**用途**：全局规则，适用于多种场景

**示例**：
```markdown
---
name: global-security-check
enabled: true
event: all
pattern: SECURITY_BREACH
action: warn
---

🚨 **安全关键词检测！**

检测到敏感词汇，请注意合规性。
```

---

#### ④ pattern（可选，Legacy）

**作用**：简单的正则表达式匹配

**注意**：这是**旧版本**的简单模式，推荐使用新的 `conditions` 格式

**语法**：Python 正则表达式

**示例**：
```yaml
# 简单场景 - 可以使用 pattern
pattern: rm\s+-rf
pattern: console\.log\(
pattern: \.env$

# 复杂场景 - 推荐使用 conditions
# （见下文）
```

**正则表达式速查**：

| 模式 | 含义 | 示例 | 匹配 |
|------|------|------|------|
| `\s+` | 1 个或多个空白字符 | `rm\s+-rf` | `rm -rf`, `rm  -rf` |
| `\.` | 字面意义的点 | `console\.log` | `console.log` |
| `$` | 行尾 | `\.env$` | `.env`, `config.env` |
| `^` | 行首 | `^import` | `import os` |
| `|` | 或 | `eval\|exec` | `eval()`, `exec()` |
| `()` | 分组 | `(foo\|bar)` | `foo`, `bar` |
| `.*` | 任意字符任意次 | `TODO.*FIXME` | `TODO later FIXME` |

---

#### ⑤ action（可选）

**作用**：指定规则触发后的行为

**取值**：

| Action | 行为 | 使用场景 |
|--------|------|---------|
| **warn**（默认） | 显示警告，但**允许继续** | 提醒、建议、最佳实践 |
| **block** | 显示警告，**阻止执行** | 危险操作、安全检查 |

**示例对比**：

```yaml
# warn - 警告但允许
action: warn
# 场景：console.log 检测、代码风格建议
# 效果：Claude 看到警告，但仍会执行操作

# block - 阻止
action: block
# 场景：rm -rf 检测、硬编码密码检测
# 效果：Claude 看到警告，操作被阻止，必须换其他方式
```

**实际案例**：

```markdown
<!-- 案例 1：warn - 只是提醒 -->
---
name: remind-remove-debug
event: file
pattern: console\.log
action: warn  # 只是提醒，允许继续
---

🐛 发现调试代码，记得删除。

<!-- 案例 2：block - 严格阻止 -->
---
name: block-hardcoded-password
event: file
pattern: password\s*=\s*["']\w+["']
action: block  # 严格阻止，必须修改
---

🔐 检测到硬编码密码！

请使用环境变量存储密码。
```

---

#### ⑥ tool_matcher（可选）

**作用**：进一步细化哪些工具触发规则

**取值**：工具名称，支持 `|` 分隔多个工具

**常见工具**：
- `Bash` - Bash 命令
- `Edit` - 编辑文件
- `Write` - 写入文件
- `MultiEdit` - 多处编辑
- `Read` - 读取文件

**示例**：
```yaml
# 只在 Edit 和 Write 时触发
tool_matcher: Edit|Write

# 只在 Bash 时触发
tool_matcher: Bash

# 多个工具
tool_matcher: Edit|Write|MultiEdit
```

**与 event 的关系**：
```yaml
# event: file 已经隐含了 Edit|Write|MultiEdit
# 所以通常不需要 tool_matcher，除非要更精细控制

event: file
tool_matcher: Edit  # 只在 Edit 时触发，不包括 Write
```

---

#### ⑦ conditions（可选，推荐）

**作用**：复杂的条件列表，替代简单的 `pattern`

**为什么要用 conditions？**

```yaml
# ❌ 简单 pattern 无法实现的场景：
# "当文件路径以 .env 结尾，并且新内容包含 KEY 时触发"

# ✅ 使用 conditions 可以实现：
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.env$
  - field: new_text
    operator: contains
    pattern: KEY
```

**Condition 结构**：

```yaml
conditions:
  - field: 字段名
    operator: 操作符
    pattern: 模式
```

**可用字段**：

| Event | 可用字段 | 说明 |
|-------|---------|------|
| **bash** | `command` | Bash 命令字符串 |
| **file** | `file_path` | 文件路径 |
| | `new_text` | 新添加的内容 |
| | `old_text` | 被替换的内容 |
| | `content` | 完整文件内容 |
| **prompt** | `user_prompt` | 用户输入 |
| **stop** | `transcript` | 会话记录 |
| **all** | `content` | 通用内容字段 |

**可用操作符**：

| Operator | 含义 | 示例 |
|----------|------|------|
| `regex_match` | 正则匹配 | `pattern: \.env$` |
| `contains` | 包含 | `pattern: KEY` |
| `equals` | 等于 | `pattern: production` |
| `not_contains` | 不包含 | `pattern: TODO` |
| `starts_with` | 开头是 | `pattern: /src/` |
| `ends_with` | 结尾是 | `pattern: .test.ts` |

---

### 2.4 Markdown Body（消息体）

**作用**：规则触发时显示的警告消息

**格式**：完整的 Markdown，支持：
- 粗体 `**text**`
- 斜体 `*text*`
- 代码块 `` `code` `` 或 ` ```language `
- 列表 `- item` 或 `1. item`
- 链接 `[text](url)`
- 引用 `> quote`

**写作技巧**：

```markdown
---
name: example-rule
# ... 配置 ...
---

## 🚨 警告标题

**问题描述：**
这里详细说明检测到了什么问题。

**影响：**
- 可能的后果 1
- 可能的后果 2

**解决方案：**
1. 第一步怎么做
2. 第二步怎么做

**参考链接：**
- [官方文档](https://example.com/docs)
- [最佳实践](https://example.com/guide)
```

**优秀示例**：

```markdown
---
name: block-hardcoded-api-key
enabled: true
event: file
conditions:
  - field: new_text
    operator: regex_match
    pattern: (API_KEY|SECRET|TOKEN)\s*=\s*["']\w+["']
action: block
---

## 🔐 硬编码 API Key 检测！

**检测到：**
您的代码中包含硬编码的 API 密钥或令牌。

**风险：**
- 🔴 **严重安全风险** - 密钥可能泄露到代码仓库
- 🔴 **未授权访问** - 攻击者可使用您的 API 配额
- 🔴 **数据泄露** - 敏感数据可能被窃取

**正确做法：**
1. 使用环境变量存储密钥
   ```typescript
   // ❌ 错误
   const apiKey = "sk-1234567890";
   
   // ✅ 正确
   const apiKey = process.env.API_KEY;
   ```

2. 在 `.env` 文件中配置
   ```bash
   # .env
   API_KEY=sk-1234567890
   ```

3. 将 `.env` 添加到 `.gitignore`

**参考资料：**
- [12-Factor App - 配置](https://12factor.net/config)
- [OWASP - 密钥管理](https://owasp.org/www-project-web-security-testing-guide/)
```

---

## 三、完整规则示例

### 3.1 入门级示例（5 个）

#### 示例 1：拦截危险 rm 命令

```markdown
<!-- .claude/hookify.dangerous-rm.local.md -->
---
name: block-dangerous-rm
enabled: true
event: bash
pattern: rm\s+-rf
action: block
---

## 🛑 危险操作拦截！

**检测到：** `rm -rf` 命令

**风险：**
- 该命令会**强制递归删除**目录
- 如果路径错误，可能导致**灾难性数据丢失**
- **无法恢复**，没有回收站

**建议：**
1. 使用 `trash` 命令代替（移动到回收站）
   ```bash
   npm install -g trash-cli
   trash file-or-directory
   ```

2. 或者先确认路径
   ```bash
   ls -la /path/to/delete
   # 确认无误后再删除
   ```

**已阻止此操作，请改用更安全的方式。**
```

---

#### 示例 2：检测 console.log

```markdown
<!-- .claude/hookify.console-log.local.md -->
---
name: detect-console-log
enabled: true
event: file
pattern: console\.log\(
action: warn
---

## 🐛 Debug 代码检测

**检测到：** `console.log` 语句

**提醒：**
- 开发环境可以使用调试输出
- **生产环境应该移除**或使用正式日志库

**建议：**
```typescript
// ❌ 生产环境不应该有
console.log("User logged in:", user);

// ✅ 使用正式日志
import logger from './logger';
logger.info('User logged in', { userId: user.id });
```

**如果这是故意的，请忽略此警告。**
```

---

#### 示例 3：保护 .env 文件

```markdown
<!-- .claude/hookify.protect-env.local.md -->
---
name: protect-env-files
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.env(\..+)?$
  - field: new_text
    operator: regex_match
    pattern: (PASSWORD|SECRET|KEY|TOKEN)\s*=
action: warn
---

## 🔐 环境配置文件检测

**检测到：** 正在编辑 `.env` 文件并包含敏感配置

**检查清单：**
- [ ] 确认此文件已在 `.gitignore` 中
- [ ] 不要提交到 Git 仓库
- [ ] 使用强密码/密钥
- [ ] 定期轮换密钥

**验证 `.gitignore`：**
```bash
# 确保 .gitignore 包含
.env
.env.local
.env.*.local
```

**如果是故意要提交的模板文件，请使用 `.env.example`。**
```

---

#### 示例 4：要求运行测试

```markdown
<!-- .claude/hookify.require-tests.local.md -->
---
name: require-tests-before-stop
enabled: true
event: stop
action: block
conditions:
  - field: transcript
    operator: not_contains
    pattern: npm test|pytest|cargo test|go test
---

## ✅ 测试未运行！

**检测：** 会话记录中没有找到测试命令

**要求：**
在停止之前，请运行测试验证功能正常。

**请选择适合的测试命令：**

**Node.js / TypeScript:**
```bash
npm test
# 或
yarn test
```

**Python:**
```bash
pytest
# 或
python -m pytest
```

**Rust:**
```bash
cargo test
```

**Go:**
```bash
go test ./...
```

**运行测试后再尝试停止。**
```

---

#### 示例 5：检测 eval 使用

```markdown
<!-- .claude/hookify.detect-eval.local.md -->
---
name: detect-eval-usage
enabled: true
event: file
pattern: (eval|Function)\s*\(
action: warn
---

## ⚠️ eval() 使用检测

**检测到：** `eval()` 或 `Function()` 构造函数

**风险：**
- 🔴 **代码注入** - 如果参数来自用户输入
- 🔴 **XSS 攻击** - 在浏览器环境中
- 🔴 **性能问题** - 无法优化

**替代方案：**

```javascript
// ❌ 危险
const result = eval(userInput);

// ✅ 使用 JSON.parse
const data = JSON.parse(jsonString);

// ✅ 使用函数映射
const operations = {
  add: (a, b) => a + b,
  sub: (a, b) => a - b
};
const result = operations[operation](a, b);
```

**如果确实需要动态代码执行，请确保：**
1. 输入经过严格验证和过滤
2. 在沙箱环境中执行
3. 有完善的错误处理
```

---

### 3.2 进阶级示例（5 个）

#### 示例 6：多条件组合 - API Key 检测

```markdown
<!-- .claude/hookify.api-key-in-code.local.md -->
---
name: detect-api-key-in-code
enabled: true
event: file
action: block
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.(ts|js|py|java|go)$
  - field: new_text
    operator: regex_match
    pattern: >
      (?i)(api[_-]?key|apikey|secret|token|auth|password)\s*[=:]\s*["'][a-zA-Z0-9_\-]{16,}["']
---

## 🔐 API 密钥硬编码检测！

**检测到：** 源代码中包含疑似 API 密钥的硬编码值

**特征：**
- 文件名：`{{file_path}}`
- 变量名包含：`api_key`, `secret`, `token`, `password` 等
- 值长度：≥ 16 字符

**严重性：** 🔴 **高危**

**必须立即修复：**

```typescript
// ❌ 错误写法
const config = {
  apiKey: "sk-1234567890abcdef",
  secret: "super_secret_value_here"
};

// ✅ 正确写法
const config = {
  apiKey: process.env.API_KEY,
  secret: process.env.SECRET
};
```

**步骤：**
1. 立即从代码中删除硬编码值
2. 添加到环境变量或配置文件
3. 如果已提交到 Git，需要：
   - 使用 `git filter-branch` 清理历史
   - 或重置相关 commit
4. 轮换已泄露的密钥

**预防：**
- 使用预提交钩子检查
- IDE 安装密钥检测插件
- 定期扫描代码仓库
```

---

#### 示例 7：SQL 注入检测

```markdown
<!-- .claude/hookify.sql-injection.local.md -->
---
name: detect-sql-injection-risk
enabled: true
event: file
action: block
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.(ts|js|py|php|java)$
  - field: new_text
    operator: regex_match
    pattern: >
      (execute|query|raw)\s*\(\s*["'`]?\s*(SELECT|INSERT|UPDATE|DELETE).*\$\{|["'`]\s*\+\s*\w+\s*\+
---

## 🚨 SQL 注入风险检测！

**检测到：** 疑似字符串拼接的 SQL 查询

**风险等级：** 🔴 **极度危险**

**问题代码：**
```typescript
// ❌ 字符串拼接 - 极易受 SQL 注入攻击
const query = `SELECT * FROM users WHERE id = ${userId}`;
await db.execute(query);

// ❌ 模板字符串插值
db.query(`UPDATE users SET name = '${name}' WHERE id = ${id}`);
```

**正确做法：**

```typescript
// ✅ 使用参数化查询
const query = 'SELECT * FROM users WHERE id = ?';
await db.execute(query, [userId]);

// ✅ 使用 ORM
const user = await User.findByPk(userId);

// ✅ 使用查询构建器
await db('users').where('id', userId).first();
```

**为什么参数化查询安全？**
- SQL 语句和数据分离
- 数据库自动转义特殊字符
- 从根本上杜绝 SQL 注入

**参考：**
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [参数化查询最佳实践](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
```

---

#### 示例 8：GitHub Actions 安全

```markdown
<!-- .claude/hookify.github-actions.local.md -->
---
name: github-actions-security
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.github/workflows/.*\.(yml|yaml)
  - field: new_text
    operator: contains
    pattern: ${{ github.event
action: warn
---

## 🐙 GitHub Actions 安全提醒

**检测到：** 在 Workflow 文件中使用 `${{ github.event.* }}`

**风险：**
- 🔴 **命令注入** - 攻击者可构造恶意输入
- 🔴 **未授权执行** - 在 CI 环境中执行任意代码

**危险模式：**
```yaml
# ❌ 直接使用未信任输入
- run: echo "${{ github.event.issue.title }}"

# ❌ 在 run 命令中插值
- run: git checkout ${{ github.event.pull_request.head.ref }}
```

**安全模式：**

```yaml
# ✅ 使用环境变量（经过 shell 转义）
- env:
    ISSUE_TITLE: ${{ github.event.issue.title }}
  run: echo "$ISSUE_TITLE"

# ✅ 使用中间变量
- name: Get branch
  id: vars
  run: echo "BRANCH=${{ github.head_ref }}" >> $GITHUB_OUTPUT
- run: git checkout "${{ steps.vars.outputs.BRANCH }}"
```

**需要特别小心的输入：**
- `github.event.issue.title`
- `github.event.issue.body`
- `github.event.pull_request.title`
- `github.event.pull_request.body`
- `github.event.comment.body`
- `github.head_ref`

**参考：**
- [GitHub Actions 安全硬化](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [GitHub 安全博客](https://github.blog/security/)
```

---

#### 示例 9：XSS 攻击检测

```markdown
<!-- .claude/hookify.xss-detection.local.md -->
---
name: detect-xss-risk
enabled: true
event: file
action: block
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.(tsx?|jsx?|vue|svelte)$
  - field: new_text
    operator: regex_match
    pattern: (dangerouslySetInnerHTML|v-html|@html)|(\.innerHTML\s*=)|(\.outerHTML\s*=)
---

## 🌐 XSS 攻击风险检测！

**检测到：** 疑似危险的 HTML 渲染方式

**风险等级：** 🔴 **高危**

**危险模式：**

```tsx
// ❌ React - dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// ❌ Vue - v-html
<div v-html="userInput"></div>

// ❌ 原生 JS - innerHTML
element.innerHTML = userInput;
```

**安全做法：**

```tsx
// ✅ React - 使用文本内容
<div>{userContent}</div>

// ✅ Vue - 使用文本插值
<div>{{ userInput }}</div>

// ✅ 原生 JS - textContent
element.textContent = userInput;

// ✅ 如果确实需要 HTML，先消毒
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userContent);
<div dangerouslySetInnerHTML={{ __html: clean }} />
```

**何时可以使用 dangerouslySetInnerHTML？**
1. 内容是**完全可信**的（非用户输入）
2. 已经过**严格的消毒处理**（如 DOMPurify）
3. 设置了**CSP（内容安全策略）**
4. 有**充分的理由**必须使用原始 HTML

**参考：**
- [OWASP XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [DOMPurify](https://github.com/cure53/DOMPurify)
```

---

#### 示例 10：Docker 安全配置

```markdown
<!-- .claude/hookify.docker-security.local.md -->
---
name: docker-security-check
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: Dockerfile$
  - field: content
    operator: regex_match
    pattern: (USER\s+root|ADD\s+http|EXPOSE\s+22)
action: warn
---

## 🐳 Dockerfile 安全检查

**检测到：** Dockerfile 中包含潜在不安全配置

**常见问题：**

### 1. 使用 root 用户

```dockerfile
# ❌ 以 root 身份运行容器
USER root

# ✅ 创建并使用普通用户
RUN groupadd -r app && useradd -r -g app app
USER app
```

### 2. ADD 指令下载远程文件

```dockerfile
# ❌ ADD 会自动解压且难以验证
ADD https://example.com/file.tar.gz /app/

# ✅ 使用 curl/wget + 验证哈希
RUN curl -fsSL https://example.com/file.tar.gz -o /tmp/file.tar.gz \
    && echo "expected_hash /tmp/file.tar.gz" | sha256sum -c - \
    && tar -xzf /tmp/file.tar.gz -C /app/ \
    && rm /tmp/file.tar.gz
```

### 3. 暴露 SSH 端口

```dockerfile
# ❌ 在生产镜像中暴露 SSH
EXPOSE 22

# ✅ 生产环境不需要 SSH
# 如需调试，使用 kubectl exec 或 docker exec
```

**其他检查项：**
- [ ] 不使用 `latest` 标签（使用具体版本号）
- [ ] 及时清理缓存（apt-get clean, rm -rf /var/cache）
- [ ] 使用多阶段构建减小镜像体积
- [ ] 不在镜像中存储密钥

**参考：**
- [Docker 安全最佳实践](https://docs.docker.com/engine/security/best-practices/)
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
```

---

## 四、实战练习

### 练习 1：创建你的第一个 Hookify 规则

**目标**：创建一个规则，当 Claude 尝试删除 `.git` 目录时阻止

**步骤**：

1. 创建文件：`.claude/hookify.protect-git.local.md`

2. 填写内容：
```markdown
---
name: protect-git-directory
enabled: true
event: bash
pattern: rm\s+(-[rf]+\s+)?\.git
action: block
---

## 🛑 保护 Git 仓库！

**检测到：** 尝试删除 `.git` 目录

**后果：**
- 整个 Git 历史记录将丢失
- 所有分支和标签信息消失
- 无法恢复到之前的版本

**如果确实需要删除：**
1. 先备份重要信息
2. 确认在正确的目录
3. 考虑使用 `git reset --hard` 代替

**操作已被阻止。**
```

3. 测试规则：
```bash
# 让 Claude 尝试删除 .git 目录
rm -rf .git
# 应该看到警告并被阻止
```

---

### 练习 2：创建代码规范检查规则

**目标**：检测 TypeScript 文件中的 `any` 类型使用

**提示**：
- event: file
- 条件 1：file_path 匹配 `.tsx?$`
- 条件 2：new_text 包含 `: any` 或 `<any>`

**参考答案**：
```markdown
---
name: detect-any-type
enabled: true
event: file
action: warn
conditions:
  - field: file_path
    operator: regex_match
    pattern: \.(ts|tsx)$
  - field: new_text
    operator: regex_match
    pattern: :\s*any|<any>
---

## ⚠️ any 类型使用检测

**检测到：** 使用了 `any` 类型

**问题：**
- 失去 TypeScript 的类型检查
- 容易引入运行时错误
- 降低代码可维护性

**更好的选择：**

```typescript
// ❌ 使用 any
function process(data: any) {
  return data.value;
}

// ✅ 使用 unknown + 类型守卫
function process(data: unknown) {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return (data as { value: string }).value;
  }
  throw new Error('Invalid data');
}

// ✅ 使用接口
interface Processable {
  value: string;
}
function process(data: Processable) {
  return data.value;
}
```

**例外情况：**
- 迁移 JavaScript 代码的过渡期
- 处理第三方库缺少类型定义
- 真正的泛型场景（考虑用 unknown）
```

---

### 练习 3：创建完成度检查规则

**目标**：确保在提交代码前运行 lint

**提示**：
- event: stop
- 检查 transcript 是否包含 `npm run lint` 或 `eslint`

**参考答案**：
```markdown
---
name: require-lint-before-stop
enabled: true
event: stop
action: block
conditions:
  - field: transcript
    operator: not_contains
    pattern: npm run lint|eslint|yarn lint
---

## 🧹 代码检查未运行！

**检测：** 会话记录中没有找到 lint 命令

**要求：**
在停止之前，请运行代码检查确保风格一致。

**请执行：**
```bash
npm run lint
# 或
yarn lint
# 或
npx eslint src/
```

**如果有 lint 错误：**
1. 优先自动修复：`npm run lint -- --fix`
2. 手动修复剩余问题
3. 再次运行确认通过

**特殊情况：**
- 如果是全新项目还没有 lint 配置，请忽略此警告
- 如果明确说明不需要 lint，请告知我

**运行 lint 后再尝试停止。**
```

---

## 五、高级技巧

### 5.1 规则组织与管理

**问题**：规则多了之后如何管理？

**方案 1：按功能分类命名**

```bash
.claude/
├── hookify.security-*.local.md      # 安全相关
│   ├── hookify.security-api-key.local.md
│   ├── hookify.security-sql.local.md
│   └── hookify.security-xss.local.md
├── hookify.quality-*.local.md       # 质量相关
│   ├── hookify.quality-tests.local.md
│   ├── hookify.quality-lint.local.md
│   └── hookify.quality-documentation.local.md
└── hookify.workflow-*.local.md      # 工作流相关
    ├── hookify.workflow-commit.local.md
    └── hookify.workflow-deploy.local.md
```

**方案 2：使用 enabled 字段分组**

```yaml
# 开发环境启用的规则
enabled: true  # 开发时

# 生产环境才启用的规则
enabled: false  # 平时关闭，部署前手动开启
```

---

### 5.2 调试技巧

**问题**：规则不生效怎么办？

**调试步骤**：

1. **检查文件格式**
```bash
# 确保文件在正确位置
ls -la .claude/hookify.*.local.md

# 检查 YAML 格式
cat .claude/hookify.your-rule.local.md
```

2. **验证 YAML 语法**
```bash
# 使用在线工具或 VS Code YAML 插件
# 确保 frontmatter 格式正确
```

3. **测试正则表达式**
```bash
# 使用 regex101.com 测试 pattern
# 或 Python 交互式测试
python3
>>> import re
>>> re.search(r'your-pattern', 'test string')
```

4. **查看 Hookify 日志**
```bash
# Hookify 通常会记录匹配的规则
# 查看相关日志输出
```

5. **简化测试**
```yaml
# 先用最简单的规则测试
---
name: test-rule
enabled: true
event: bash
pattern: echo
action: warn
---

Test message.
```

---

### 5.3 性能优化

**问题**：规则多了会影响性能吗？

**优化建议**：

1. **使用具体的 pattern**
```yaml
# ❌ 太宽泛，每次都要检查
pattern: .*

# ✅ 具体明确，快速匹配
pattern: console\.log\(
```

2. **利用 tool_matcher 减少不必要的检查**
```yaml
# ❌ 所有文件操作都检查
event: file

# ✅ 只检查特定文件
event: file
tool_matcher: Edit|Write
```

3. **禁用不用的规则**
```yaml
# 暂时不用的规则，设置 enabled: false
# 而不是删除文件
enabled: false
```

---

### 5.4 规则分享

**制作可分享的规则包**：

```bash
my-hookify-rules/
├── README.md              # 说明文档
├── security/              # 安全规则
│   ├── api-key.local.md
│   ├── sql-injection.local.md
│   └── xss.local.md
├── quality/               # 质量规则
│   ├── console-log.local.md
│   ├── any-type.local.md
│   └── tests.local.md
└── workflow/              # 工作流规则
    ├── commit-message.local.md
    └── pr-description.local.md
```

**README.md 示例**：
```markdown
# My Hookify Rules Collection

精选的 Hookify 规则集合，适用于 TypeScript/Node.js 项目。

## 安装

```bash
# 复制所有规则到你的项目
cp my-hookify-rules/*.local.md your-project/.claude/
```

## 分类

### 安全规则
- API Key 检测
- SQL 注入防护
- XSS 攻击防护

### 质量规则
- Console.log 检测
- Any 类型检测
- 测试覆盖要求

## 自定义

根据实际情况调整规则的 enabled 字段。
```

---

## 六、常见问题 FAQ

### Q1: Hookify 和传统 Hook 能同时使用吗？

**A**: ✅ **可以！**它们互不冲突。

```bash
plugins/
├── security-guidance/     # 传统 Hook
│   └── hooks/
│       ├── hooks.json
│       └── security_reminder_hook.py
│
└── hookify/              # Hookify
    └── hooks/
        └── hooks.json    # 加载 .local.md 规则
```

**最佳实践**：
- 稳定、通用的规则 → 传统 Hook（打包成插件）
- 个性化、临时的需求 → Hookify（快速配置）

---

### Q2: 规则修改后多久生效？

**A**: ⚡ **立即生效！**

```markdown
1. 编辑 .local.md 文件
2. 保存
3. 下一次工具调用立即生效
```

**无需重启 Claude**，这是 Hookify 的一大优势！

---

### Q3: 如何临时禁用所有规则？

**A**: 有 3 种方法：

**方法 1**：重命名目录
```bash
mv .claude .claude.backup  # 所有规则失效
# ... 做一些事 ...
mv .claude.backup .claude  # 恢复
```

**方法 2**：批量修改 enabled
```bash
# 用脚本批量改成 enabled: false
find .claude -name "hookify.*.local.md" -exec \
  sed -i 's/enabled: true/enabled: false/g' {} \;
```

**方法 3**：在 Hookify 插件中配置
```json
// hooks/hooks.json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "python3 ...",
        "enabled": false  # 如果支持的话
      }]
    }]
  }
}
```

---

### Q4: 规则太多会影响性能吗？

**A**: 轻微影响，但通常可接受。

**实测数据**：
- 10 条规则 → < 10ms
- 50 条规则 → ~50ms
- 100 条规则 → ~100ms

**优化建议**：
- 保持规则数量在 50 条以内
- 使用精确的 pattern
- 定期清理不用的规则

---

### Q5: 如何备份和恢复规则？

**A**: 使用 Git 或简单的文件复制。

**Git 方式**：
```bash
# 在项目根目录
git add .claude/hookify.*.local.md
git commit -m "Add hookify rules"
git push

# 恢复
git pull
```

**手动备份**：
```bash
# 备份
cp -r .claude/hookify.*.local.md ~/backups/hookify-rules/

# 恢复
cp ~/backups/hookify-rules/*.local.md .claude/
```

---

## 七、总结

### 📝 知识点回顾

**核心概念**：
- ✅ Hookify 是基于 markdown 配置的轻量级 Hook 系统
- ✅ 无需编程，只需写配置文件
- ✅ 修改后立即生效，无需重启

**文件格式**：
```markdown
---
name: 规则名称（必需）
enabled: true/false
event: bash|file|stop|prompt|all
pattern: 正则表达式（简单场景）
action: warn/block
conditions: [复杂条件列表]
---

警告消息内容（支持 Markdown）
```

**5 种事件类型**：
- bash - Bash 命令
- file - 文件编辑
- stop - 完成度检查
- prompt - 用户输入
- all - 全局规则

**实战技能**：
- ✅ 创建安全检测规则
- ✅ 创建代码规范规则
- ✅ 创建工作流规则
- ✅ 调试和优化规则

---

### 🎯 下一步

学完本章后，你可以：

1. **立即实践**：创建 3-5 个实用的 Hookify 规则
2. **深入学习**：阅读【03-MCP 服务器集成】了解更强大的插件功能
3. **参考源码**：研究 `plugins/hookify/` 的实现细节
4. **分享交流**：在社区分享你的规则集合

---

### 📚 参考资料

**官方文档**：
- [Hookify Plugin README](../../../plugins/hookify/README.md)
- [Hookify Core Code](../../../plugins/hookify/core/)

**外部资源**：
- [Python 正则表达式文档](https://docs.python.org/zh-cn/3/library/re.html)
- [Regex101 在线测试](https://regex101.com/)
- [OWASP 安全指南](https://owasp.org/)

---

**恭喜！** 🎉 你已经掌握了 Hookify 的完整使用方法，开始创建你自己的规则吧！
