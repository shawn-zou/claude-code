# Hook 开发工具链

在 Hook 开发过程中，项目提供了一套完整的工具来帮助你验证、测试和优化 Hook 配置。这套工具链可以大大提高开发效率，减少错误。

---

## 一、工具总览

项目提供了三个核心工具脚本：

| 工具 | 位置 | 作用 | 使用阶段 |
|------|------|------|----------|
| **validate-hook-schema.sh** | `plugins/plugin-dev/scripts/` | 验证 hooks.json 结构 | 配置完成后 |
| **test-hook.sh** | `plugins/plugin-dev/scripts/` | 测试单个 Hook 脚本 | 开发调试中 |
| **hook-linter.sh** | `plugins/plugin-dev/scripts/` | 检查 Hook 代码质量 | 提交前检查 |

---

## 二、validate-hook-schema.sh - 配置验证器

### 2.1 作用

验证 `hooks.json` 配置文件的正确性，确保：
- ✅ JSON 语法正确
- ✅ Hook 事件名称有效
- ✅ 超时时间设置合理
- ✅ 必填字段完整
- ✅ 检测硬编码路径

### 2.2 使用方法

**基本用法：**
```bash
cd my-plugin
./validate-hook-schema.sh hooks/hooks.json
```

**示例输出：**
```
🔍 Validating hooks configuration: hooks/hooks.json

Checking JSON syntax...
✅ Valid JSON

Checking root structure...
✅ Root structure valid

Validating individual hooks...

Hook: PreToolUse[0]
  ✅ Type: command
  ✅ Timeout: 30s (valid range)
  ⚠️  Warning: Hardcoded path detected: /home/user/script.sh
     Suggestion: Use ${CLAUDE_PLUGIN_ROOT} instead

Hook: Stop[0]
  ✅ Type: prompt
  ✅ Timeout: 45s (valid range)
  ✅ No issues found

Validation complete!
Errors: 0, Warnings: 1
```

### 2.3 检查项详解

#### 检查 1：JSON 语法
```bash
# ❌ 错误示例：缺少逗号
{
  "PreToolUse": [
    {
      "type": "command"
      "command": "echo hello"  # 缺少逗号
    }
  ]
}

# ✅ 正确示例
{
  "PreToolUse": [
    {
      "type": "command",
      "command": "echo hello"
    }
  ]
}
```

#### 检查 2：事件名称有效性
```bash
# ✅ 有效的事件名称
- PreToolUse
- PostToolUse
- Stop
- SubagentStop
- UserPromptSubmit
- SessionStart
- SessionEnd
- PreCompact
- Notification

# ❌ 无效的事件名称
- preToolUse (大小写错误)
- BeforeToolUse (名称错误)
- ToolUse (不完整)
```

#### 检查 3：超时时间范围
```bash
# ✅ 合理的超时设置
"timeout": 10   # 快速检查
"timeout": 30   # 标准 AI 分析
"timeout": 60   # 复杂脚本执行

# ❌ 不合理的超时设置
"timeout": 0    # 太短，无法完成
"timeout": 300  # 太长，用户体验差
```

#### 检查 4：硬编码路径检测
```bash
# ❌ 错误：硬编码绝对路径
{
  "command": "/home/david/my-plugin/scripts/check.sh"
}

# ✅ 正确：使用环境变量
{
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/check.sh"
}
```

---

## 三、test-hook.sh - Hook 测试器

### 3.1 作用

在部署前测试 Hook 脚本的实际运行效果，支持：
- ✅ 模拟真实输入
- ✅ 测量执行时间
- ✅ 验证输出格式
- ✅ 检查退出码

### 3.2 使用方法

**步骤 1：创建测试输入**
```bash
# 为 PreToolUse 事件创建测试数据
./test-hook.sh --create-sample PreToolUse > test-input.json

# 查看生成的测试数据
cat test-input.json
```

**生成的测试输入：**
```json
{
  "session_id": "test-session",
  "transcript_path": "/tmp/transcript.txt",
  "cwd": "/tmp/test-project",
  "permission_mode": "ask",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/tmp/test.txt",
    "content": "Test content"
  }
}
```

**步骤 2：运行测试**
```bash
# 基本测试
./test-hook.sh my-hook.sh test-input.json

# 详细模式（显示更多信息）
./test-hook.sh -v my-hook.sh test-input.json

# 设置超时时间（秒）
./test-hook.sh -t 30 my-hook.sh test-input.json
```

**示例输出：**
```
🧪 Testing Hook: my-hook.sh
Input: test-input.json

⏱️  Execution Time: 0.234s
Exit Code: 0 (Success)

Output:
{
  "continue": true,
  "systemMessage": "检查通过，允许执行"
}

✅ Test passed!
```

### 3.3 测试不同类型的事件

#### 测试 PreToolUse Hook
```bash
# 创建测试输入
./test-hook.sh --create-sample PreToolUse > pre-test.json

# 修改文件路径和内容
jq '.tool_input.file_path = "/path/to/file.js"' pre-test.json > temp.json
jq '.tool_input.content = "const x = eval(userInput)"' temp.json > final.json

# 运行测试
./test-hook.sh security-check.sh final.json
```

#### 测试 Stop Hook
```bash
# 创建 Stop 事件的测试输入
./test-hook.sh --create-sample Stop > stop-test.json

# 修改停止原因
jq '.reason = "Task appears complete"' stop-test.json > temp.json

# 运行测试
./test-hook.sh completion-check.sh temp.json
```

#### 测试 UserPromptSubmit Hook
```bash
# 创建 UserPromptSubmit 事件的测试输入
./test-hook.sh --create-sample UserPromptSubmit > prompt-test.json

# 修改用户提示词
jq '.user_prompt = "帮我写一个登录功能"' prompt-test.json > temp.json

# 运行测试
./test-hook.sh prompt-validator.sh temp.json
```

### 3.4 解读测试结果

#### 退出码含义
```bash
Exit Code 0  # ✅ 成功，允许执行
Exit Code 1  # ⚠️ 警告，但不阻止
Exit Code 2  # ❌ 严重错误，阻止执行
Exit Code 124 # ⏱️ 超时（由 test-hook.sh 检测）
```

#### 输出格式验证
```bash
# ✅ 正确的输出格式
{
  "continue": true,
  "systemMessage": "消息内容"
}

# ❌ 错误的输出格式
检查通过  # 不是有效的 JSON

# ✅ PreToolUse 的特殊输出格式
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow"
  },
  "systemMessage": "解释说明"
}
```

---

## 四、hook-linter.sh - 代码质量检查

### 4.1 作用

检查 Hook 脚本的代码质量，确保遵循最佳实践：
- ✅ 变量引用是否正确
- ✅ 错误处理是否完善
- ✅ 是否有硬编码路径
- ✅ JSON 输出格式是否正确
- ✅ 是否符合 Shell 规范

### 4.2 使用方法

**基本用法：**
```bash
./hook-linter.sh my-hook.sh
```

**检查多个脚本：**
```bash
./hook-linter.sh hook1.sh hook2.sh hook3.sh
```

**示例输出：**
```
🔍 Linting: my-hook.sh

[PASS] ✓ Shebang line present
[PASS] ✓ set -euo pipefail used
[PASS] ✓ Variables properly quoted
[WARN] ⚠ Line 23: Unquoted variable: $file_path
       Suggestion: Use "$file_path" instead
[FAIL] ✗ Missing error handling for jq command
       Suggestion: Add || echo '{"error": "..."}' >&2
[PASS] ✓ Exit codes are meaningful

Linting complete!
Passed: 5, Warnings: 1, Failures: 1
```

### 4.3 检查项详解

#### 检查 1：Shebang 行
```bash
#!/bin/bash
# ✅ 必需：指定解释器

# ❌ 缺失 Shebang
input=$(cat)
```

#### 检查 2：严格模式
```bash
# ✅ 推荐：启用严格模式
set -euo pipefail

# ❌ 不推荐：没有错误处理
input=$(cat)
# 如果 cat 失败，脚本继续执行
```

#### 检查 3：变量引用
```bash
# ✅ 正确：引号保护
file_path="$1"
cd "$CLAUDE_PROJECT_DIR"

# ❌ 错误：未引号保护
file_path=$1  # 如果包含空格会出错
cd $CLAUDE_PROJECT_DIR
```

#### 检查 4：错误处理
```bash
# ✅ 正确：处理命令失败
result=$(jq -r '.field' input.json) || {
  echo '{"error": "Invalid JSON"}' >&2
  exit 1
}

# ❌ 错误：不处理失败
result=$(jq -r '.field' input.json)
# 如果 JSON 无效，脚本崩溃
```

#### 检查 5：退出码语义
```bash
# ✅ 明确的退出码
if [ "$risk_level" = "high" ]; then
  echo "High risk detected" >&2
  exit 2  # 阻止
else
  exit 0  # 允许
fi

# ❌ 混乱的退出码
exit 1  # 什么意思？
exit 7  # 为什么是 7？
```

---

## 五、完整开发流程示例

### 5.1 从零开始开发一个 Hook

#### 步骤 1：创建配置文件
```bash
mkdir -p my-security-plugin/hooks
cd my-security-plugin

cat > hooks/hooks.json << 'EOF'
{
  "description": "SQL 注入检测 Hook",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/check-sql-injection.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
EOF
```

#### 步骤 2：创建 Hook 脚本
```bash
mkdir -p hooks/scripts

cat > hooks/scripts/check-sql-injection.sh << 'EOF'
#!/bin/bash
set -euo pipefail

# 读取输入
input=$(cat)
content=$(echo "$input" | jq -r '.tool_input.content // ""')

# 检测 SQL 注入模式
if echo "$content" | grep -qE "SELECT.*FROM.*\+.*\+" ; then
  echo "⚠️ Security Warning: Potential SQL injection detected!" >&2
  echo "Pattern: String concatenation in SQL query" >&2
  echo "Suggestion: Use parameterized queries" >&2
  exit 2
fi

# 通过检查
exit 0
EOF

chmod +x hooks/scripts/check-sql-injection.sh
```

#### 步骤 3：验证配置
```bash
# 运行验证器
../plugin-dev/scripts/validate-hook-schema.sh hooks/hooks.json

# 输出：
# ✅ Valid JSON
# ✅ Root structure valid
# ✅ No issues found
```

#### 步骤 4：测试 Hook
```bash
# 创建测试输入
../plugin-dev/scripts/test-hook.sh --create-sample PreToolUse > test-input.json

# 添加 SQL 注入代码到测试内容
jq '.tool_input.content = "SELECT * FROM users WHERE id = " + userId' test-input.json > sql-test.json

# 运行测试
../plugin-dev/scripts/test-hook.sh -v hooks/scripts/check-sql-injection.sh sql-test.json

# 输出：
# 🧪 Testing Hook: check-sql-injection.sh
# ⏱️  Execution Time: 0.156s
# Exit Code: 2 (Blocking)
# ✅ Test passed! (成功检测到 SQL 注入)
```

#### 步骤 5：代码质量检查
```bash
# 运行 linter
../plugin-dev/scripts/hook-linter.sh hooks/scripts/check-sql-injection.sh

# 输出：
# 🔍 Linting: check-sql-injection.sh
# [PASS] ✓ All checks passed!
```

#### 步骤 6：安装并测试插件
```bash
# 安装插件
claude plugins install .

# 启动 Claude Code 测试
claude

# 让 Claude 写入包含 SQL 注入的代码
# 观察 Hook 是否触发警告
```

---

## 六、调试技巧

### 6.1 启用调试日志

**在 Hook 脚本中添加：**
```bash
#!/bin/bash
set -x  # 打印执行的每一行

debug_log() {
  echo "[DEBUG] $1" >> /tmp/hook-debug.log
}

debug_log "Starting hook execution..."
debug_log "Input: $(cat)"
```

### 6.2 查看 Claude Code 日志

```bash
# 查看最近的日志
tail -f ~/.claude/debug.log

# 搜索 Hook 相关信息
grep -i "hook" ~/.claude/debug.log | tail -50
```

### 6.3 常见问题排查

#### 问题 1：Hook 不触发
```bash
# 检查清单：
1. ✅ hooks.json 是否在正确位置？
2. ✅ matcher 是否正确配置？
3. ✅ 是否重启了 Claude Code？
4. ✅ 插件是否已安装并启用？

# 调试方法：
/claude hooks list  # 查看已加载的 Hook
```

#### 问题 2：Hook 总是超时
```bash
# 可能原因：
1. ⏱️ timeout 设置太短
2. ⏱️ 脚本执行太慢
3. ⏱️ AI 响应延迟

# 解决方法：
- 增加 timeout 值（建议 30-60 秒）
- 优化脚本性能
- 使用 Command Hook 替代 Prompt Hook
```

#### 问题 3：输出格式错误
```bash
# 常见错误：
echo "检查通过"  # ❌ 不是 JSON

# 正确做法：
echo '{"continue": true}'  # ✅ 有效 JSON
```

---

## 七、自动化测试

### 7.1 创建测试套件

在项目根目录创建 `tests/` 目录：

```bash
mkdir -p tests
```

**测试文件结构：**
```
tests/
├── test-sql-injection-hook.sh
├── test-xss-hook.sh
├── test-stop-hook.sh
└── run-all-tests.sh
```

### 7.2 编写测试用例

**test-sql-injection-hook.sh：**
```bash
#!/bin/bash
# SQL 注入检测 Hook 的测试套件

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
HOOK_SCRIPT="$SCRIPT_DIR/../hooks/scripts/check-sql-injection.sh"
TEST_HELPER="$SCRIPT_DIR/../../plugin-dev/scripts/test-hook.sh"

# 测试计数器
TOTAL=0
PASSED=0
FAILED=0

# 测试函数
run_test() {
  local test_name="$1"
  local test_content="$2"
  local expected_exit="$3"
  
  TOTAL=$((TOTAL + 1))
  
  # 创建测试输入
  echo "{\"tool_input\": {\"content\": \"$test_content\"}}" > /tmp/test-input.json
  
  # 运行测试
  bash "$TEST_HELPER" "$HOOK_SCRIPT" /tmp/test-input.json > /tmp/test-output.txt 2>&1
  actual_exit=$?
  
  # 检查结果
  if [ "$actual_exit" -eq "$expected_exit" ]; then
    echo "✅ PASS: $test_name"
    PASSED=$((PASSED + 1))
  else
    echo "❌ FAIL: $test_name"
    echo "   Expected exit code: $expected_exit"
    echo "   Actual exit code: $actual_exit"
    FAILED=$((FAILED + 1))
  fi
}

# 运行测试
echo "🧪 Running SQL Injection Hook Tests..."
echo ""

run_test "Detect string concatenation" "SELECT * FROM users WHERE id = \" + id" 2
run_test "Detect SQL with plus" "query = 'SELECT' + userInput" 2
run_test "Allow parameterized query" "db.query('SELECT * FROM users WHERE id = ?', [id])" 0
run_test "Allow safe string" "This is just a comment about SQL" 0

# 输出总结
echo ""
echo "================================"
echo "Test Summary:"
echo "  Total:  $TOTAL"
echo "  Passed: $PASSED"
echo "  Failed: $FAILED"
echo "================================"

if [ "$FAILED" -gt 0 ]; then
  exit 1
fi
exit 0
```

### 7.3 运行所有测试

**run-all-tests.sh：**
```bash
#!/bin/bash
# 运行所有 Hook 测试

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

echo "🚀 Starting Hook Test Suite"
echo ""

# 运行各个测试文件
for test_file in "$SCRIPT_DIR"/test-*.sh; do
  if [ -f "$test_file" ]; then
    echo "Running: $(basename "$test_file")"
    echo ""
    bash "$test_file"
    echo ""
  fi
done

echo "✅ All tests completed!"
```

---

## 八、性能优化

### 8.1 缓存机制

对于频繁执行的检查，可以使用缓存：

```bash
#!/bin/bash

cache_key=$(echo -n "$content" | md5sum | cut -d' ' -f1)
cache_file="/tmp/hook-cache-$cache_key"

# 检查缓存（5 分钟有效期）
if [ -f "$cache_file" ]; then
  cache_age=$(($(date +%s) - $(stat -c%Y "$cache_file")))
  if [ "$cache_age" -lt 300 ]; then
    cat "$cache_file"
    exit $(cat "${cache_file}.exit")
  fi
fi

# 执行实际检查
# ... 检查逻辑 ...

# 保存结果到缓存
echo "$result" > "$cache_file"
echo "$exit_code" > "${cache_file}.exit"
```

### 8.2 并行检查优化

由于 Hook 是并行执行的，设计时要保证独立性：

```bash
# ✅ 好的设计：每个 Hook 独立工作
Hook 1: 检查路径安全 → 独立
Hook 2: 检查内容安全 → 独立
Hook 3: 检查命名规范 → 独立

# ❌ 坏的设计：Hook 之间相互依赖
Hook 1: 保存状态 → 被 Hook 2 依赖
Hook 2: 读取 Hook 1 的状态 → 可能失败（并行执行）
```

---

## 九、最佳实践总结

### ✅ 推荐做法

1. **开发前：先设计**
   - 明确 Hook 的目的
   - 选择合适的 Hook 类型
   - 定义清晰的输入输出

2. **开发中：使用工具**
   - 用 validate-hook-schema.sh 验证配置
   - 用 test-hook.sh 测试功能
   - 用 hook-linter.sh 检查质量

3. **开发后：充分测试**
   - 编写自动化测试
   - 覆盖边界情况
   - 文档化使用方法

### ❌ 避免的错误

1. **不测试就部署**
   ```bash
   # ❌ 错误
   写完直接用
   
   # ✅ 正确
   验证 → 测试 → 检查 → 部署
   ```

2. **忽视错误处理**
   ```bash
   # ❌ 错误
   result=$(jq '.field' input.json)
   
   # ✅ 正确
   result=$(jq '.field' input.json) || {
     echo "Error: Invalid input" >&2
     exit 1
   }
   ```

3. **过度复杂的逻辑**
   ```bash
   # ❌ 错误：100 行的复杂脚本
   # ✅ 正确：拆分为多个小函数，每个 < 20 行
   ```

---

## 十、参考资源

### 工具脚本位置
- `plugins/plugin-dev/scripts/validate-hook-schema.sh`
- `plugins/plugin-dev/scripts/test-hook.sh`
- `plugins/plugin-dev/scripts/hook-linter.sh`

### 示例代码
- `plugins/security-guidance/hooks/` - 安全检查示例
- `plugins/hookify/hooks/` - 自然语言规则示例
- `plugins/plugin-dev/examples/` - 开发模板

### 文档
- Hook API: `plugins/plugin-dev/skills/hook-development/SKILL.md`
- 高级模式：`plugins/plugin-dev/skills/hook-development/references/advanced.md`
- 常见模式：`plugins/plugin-dev/skills/hook-development/references/patterns.md`

---

**下一步：** 掌握工具链后，继续学习 [Hook 项目组织最佳实践](./03-Hook 项目组织最佳实践.md)！
