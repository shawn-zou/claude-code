# Hook 项目组织最佳实践

当你开始开发复杂的 Hook 系统时，良好的组织结构变得至关重要。本章将介绍如何组织 Hook 项目，使其易于维护、扩展和协作。

---

## 一、三种组织结构对比

根据项目规模和团队需求，有三种主流的组织方式：

### 1.1 集中式组织（适合小型项目）

**结构：**
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── hooks/
│   ├── hooks.json              # 所有 Hook 配置
│   └── scripts/
│       ├── check-security.sh
│       ├── validate-quality.sh
│       └── load-context.sh
└── README.md
```

**特点：**
- ✅ 所有配置在一个文件
- ✅ 脚本集中在一个目录
- ✅ 简单直观，易于理解

**适用场景：**
- 5-10 个 Hook 以内
- 个人项目或小团队
- 快速原型开发

**优缺点分析：**

| 优点 | 缺点 |
|------|------|
| 配置集中，一目了然 | 文件过大难以维护 |
| 结构简单，新人易上手 | 多人协作易冲突 |
| 部署方便 | 难以复用 |

---

### 1.2 按事件类型组织（适合中型项目）

**结构：**
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── hooks/
│   ├── hooks.json                  # 总配置文件
│   ├── pre-tool-use.json           # PreToolUse 专用配置
│   ├── post-tool-use.json          # PostToolUse 专用配置
│   ├── stop.json                   # Stop 专用配置
│   └── scripts/
│       ├── pre/
│       │   ├── security-check.sh
│       │   └── path-validator.sh
│       ├── post/
│       │   ├── quality-analyzer.sh
│       │   └── test-tracker.sh
│       └── stop/
│           └── completion-verifier.sh
└── README.md
```

**特点：**
- ✅ 按事件类型分离配置
- ✅ 每个事件独立管理
- ✅ 便于团队协作

**适用场景：**
- 10-20 个 Hook
- 中型项目
- 多人协作开发

**配置示例：**

**hooks.json（总配置）：**
```json
{
  "description": "模块化 Hook 系统",
  "hooks": {
    "PreToolUse": "${file:./pre-tool-use.json}",
    "PostToolUse": "${file:./post-tool-use.json}",
    "Stop": "${file:./stop.json}"
  }
}
```

**pre-tool-use.json：**
```json
[
  {
    "matcher": "Write|Edit",
    "hooks": [
      {
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/pre/security-check.sh",
        "timeout": 10
      }
    ]
  },
  {
    "matcher": "Bash",
    "hooks": [
      {
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/pre/path-validator.sh",
        "timeout": 5
      }
    ]
  }
]
```

**注意：** Claude Code 不直接支持文件引用语法，需要使用构建脚本合并。

---

### 1.3 按功能目的组织（适合大型项目）

**结构：**
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── hooks/
│   ├── hooks.json                  # 总入口
│   └── features/
│       ├── security/               # 安全相关
│       │   ├── hooks.json
│       │   ├── scripts/
│       │   │   ├── sql-injection-check.sh
│       │   │   ├── xss-detector.sh
│       │   │   └── path-validator.sh
│       │   └── tests/
│       │       └── test-security.sh
│       ├── quality/                # 质量相关
│       │   ├── hooks.json
│       │   ├── scripts/
│       │   │   ├── code-style.sh
│       │   │   ├── test-coverage.sh
│       │   │   └── doc-checker.sh
│       │   └── tests/
│       │       └── test-quality.sh
│       └── workflow/               # 工作流相关
│           ├── hooks.json
│           ├── scripts/
│           │   ├── notify-team.sh
│           │   └── update-status.sh
│           └── tests/
│               └── test-workflow.sh
└── README.md
```

**特点：**
- ✅ 按功能模块划分
- ✅ 每个模块独立完整
- ✅ 易于理解和维护

**适用场景：**
- 20+ 个 Hook
- 大型复杂项目
- 多团队分工协作

**优缺点分析：**

| 优点 | 缺点 |
|------|------|
| 模块化，职责清晰 | 结构复杂，学习曲线陡 |
| 团队可并行开发 | 需要良好的文档 |
| 易于测试和维护 | 初始 setup 成本高 |

---

## 二、选择适合你的组织结构

### 决策树

```
项目有多少个 Hook？
    ↓
  < 10 个 → 集中式
    ↓
  10-20 个 → 按事件类型
    ↓
  > 20 个 → 按功能模块
```

### 其他考虑因素

**团队规模：**
- 1-2 人 → 集中式或按事件
- 3-5 人 → 按事件或按功能
- 5+ 人 → 强烈推荐按功能模块

**项目生命周期：**
- 短期项目（< 3 个月）→ 集中式
- 中期项目（3-12 个月）→ 按事件
- 长期项目（> 1 年）→ 按功能模块

**变更频率：**
- 频繁变更 → 按功能模块（影响范围小）
- 稳定不变 → 集中式（简单高效）

---

## 三、性能优化技巧

### 3.1 缓存验证结果

对于耗时的检查，使用缓存避免重复计算：

**实现示例：**
```bash
#!/bin/bash
# 带缓存的安全检查

input=$(cat)
content=$(echo "$input" | jq -r '.tool_input.content // ""')

# 生成缓存键
cache_key=$(echo -n "$content" | md5sum | cut -d' ' -f1)
cache_file="/tmp/hook-cache-$cache_key"

# 检查缓存（5 分钟有效期）
if [ -f "$cache_file" ]; then
  cache_age=$(($(date +%s) - $(stat -c%Y "$cache_file")))
  if [ "$cache_age" -lt 300 ]; then
    # 命中缓存
    cached_result=$(cat "$cache_file")
    cached_exit=$(cat "${cache_file}.exit")
    
    echo "$cached_result"
    exit "$cached_exit"
  fi
fi

# 执行实际检查
# ... 检查逻辑 ...
result='{"continue": true}'
exit_code=0

# 保存到缓存
echo "$result" > "$cache_file"
echo "$exit_code" > "${cache_file}.exit"

echo "$result"
exit $exit_code
```

**效果：**
- 首次执行：0.5 秒
- 缓存命中：< 0.01 秒
- 性能提升：50 倍

---

### 3.2 并行执行优化

由于 Hook 是并行执行的，要充分利用这个特性：

**不好的设计（串行思维）：**
```json
{
  "PreToolUse": [
    {
      "hooks": [
        {"type": "command", "command": "check1.sh"},  // 1 秒
        {"type": "command", "command": "check2.sh"},  // 1 秒
        {"type": "command", "command": "check3.sh"}   // 1 秒
      ]
    }
  ]
}
```
总耗时：3 秒（顺序执行）

**好的设计（并行思维）：**
```json
{
  "PreToolUse": [
    {
      "hooks": [
        {"type": "command", "command": "check-all.sh"}  // 1 秒，内部并行
      ]
    }
  ]
}
```

**或者真正并行：**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {"type": "command", "command": "check-path.sh", "timeout": 2},   // 并行
        {"type": "command", "command": "check-content.sh", "timeout": 2}, // 并行
        {"type": "prompt", "command": "analyze.sh", "timeout": 10}       // 并行
      ]
    }
  ]
}
```
总耗时：max(2, 2, 10) = 10 秒

---

### 3.3 超时设置策略

合理的超时设置可以平衡性能和用户体验：

**推荐值：**

| Hook 类型 | 推荐超时 | 说明 |
|----------|---------|------|
| Command Hook（简单检查） | 5-10 秒 | 路径验证、格式检查 |
| Command Hook（复杂脚本） | 30-60 秒 | 外部 API 调用、数据库查询 |
| Prompt Hook（快速分析） | 15-20 秒 | 简单代码审查 |
| Prompt Hook（深度分析） | 30-45 秒 | 架构评估、安全审计 |
| Stop Hook | 45-60 秒 | 完整性验证 |

**设置原则：**
1. 宁可稍长，不可过短（避免误杀）
2. 根据历史数据调整
3. 提供友好的超时提示

---

## 四、条件执行策略

不是所有 Hook 都需要在所有情况下执行。条件执行可以提高效率并减少干扰。

### 4.1 基于环境变量的条件执行

**场景：** 只在 CI 环境中运行完整检查

```bash
#!/bin/bash

# 仅在 CI 环境中运行
if [ -z "$CI" ]; then
  echo '{"continue": true, "systemMessage": "本地开发环境，跳过详细检查"}' >&2
  exit 0
fi

# CI 环境中执行完整检查
# ... 详细的验证逻辑 ...
```

**配置示例：**
```yaml
# GitHub Actions
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      CI: true
    steps:
      - uses: actions/checkout@v2
      - run: claude plugins install ./my-plugin
```

---

### 4.2 基于用户身份的条件执行

**场景：** 对资深开发者放宽检查

```bash
#!/bin/bash

# 获取当前用户
CURRENT_USER="$USER"

# 读取团队配置
TRUSTED_USERS=("alice" "bob" "charlie")

# 检查是否是受信任用户
for trusted in "${TRUSTED_USERS[@]}"; do
  if [ "$CURRENT_USER" = "$trusted" ]; then
    echo '{"continue": true, "systemMessage": "受信任用户，简化检查"}' >&2
    exit 0
  fi
done

# 对其他用户执行完整检查
# ... 详细检查逻辑 ...
```

**使用建议：**
- ⚠️ 谨慎使用，避免特权滥用
- ✅ 记录所有豁免操作
- ✅ 定期审查受信任用户列表

---

### 4.3 基于标志文件的条件执行

**场景：** 选择性启用特定检查

```bash
#!/bin/bash

FLAG_FILE="$CLAUDE_PROJECT_DIR/.enable-strict-mode"

if [ ! -f "$FLAG_FILE" ]; then
  # 标准模式
  exit 0
fi

# 严格模式
# ... 额外的检查 ...
```

**使用方法：**
```bash
# 启用严格模式
touch .enable-strict-mode

# 禁用严格模式
rm .enable-strict-mode
```

**实际应用：**
- 开发阶段：标准模式（快速迭代）
- 发布前：严格模式（全面检查）
- 特殊项目：自定义模式

---

### 4.4 基于项目类型的条件执行

**场景：** 不同项目类型使用不同规则

```bash
#!/bin/bash

cd "$CLAUDE_PROJECT_DIR" || exit 1

# 检测项目类型
if [ -f "package.json" ]; then
  PROJECT_TYPE="nodejs"
elif [ -f "Cargo.toml" ]; then
  PROJECT_TYPE="rust"
elif [ -f "requirements.txt" ]; then
  PROJECT_TYPE="python"
else
  PROJECT_TYPE="unknown"
fi

# 根据项目类型加载相应规则
case "$PROJECT_TYPE" in
  nodejs)
    RULES_FILE="$CLAUDE_PLUGIN_ROOT/rules/nodejs-rules.md"
    ;;
  rust)
    RULES_FILE="$CLAUDE_PLUGIN_ROOT/rules/rust-rules.md"
    ;;
  *)
    echo "未知项目类型，使用通用规则"
    RULES_FILE="$CLAUDE_PLUGIN_ROOT/rules/generic-rules.md"
    ;;
esac

# 加载规则文件
if [ -f "$RULES_FILE" ]; then
  cat "$RULES_FILE" >> "$TRANSCRIPT_PATH"
fi
```

---

## 五、状态跟踪与审计

在复杂的工作流中，Hook 之间需要共享状态信息。

### 5.1 使用临时文件跟踪状态

**场景：** 跟踪测试执行次数

**PostToolUse Hook - 跟踪测试：**
```bash
#!/bin/bash
# test-tracker.sh

input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')
tool_result=$(echo "$input" | jq -r '.tool_result // ""')

# 只跟踪 Bash 工具
if [ "$tool_name" != "Bash" ]; then
  exit 0
fi

# 检测是否是测试命令
if [[ "$tool_result" == *"test"* ]] || [[ "$tool_result" == *"npm test"* ]]; then
  # 增加计数
  count_file="/tmp/test-count-$$"
  count=$(cat "$count_file" 2>/dev/null || echo "0")
  echo $((count + 1)) > "$count_file"
  
  echo "✅ 测试已运行（第 $count 次）" >&2
fi

exit 0
```

**Stop Hook - 验证测试：**
```bash
#!/bin/bash
# test-verifier.sh

count_file="/tmp/test-count-$$"
test_count=$(cat "$count_file" 2>/dev/null || echo "0")

if [ "$test_count" -eq 0 ]; then
  echo '{"decision": "block", "reason": "❌ 未运行测试！代码修改后必须运行测试。"}' >&2
  exit 2
fi

echo "✅ 已运行 $test_count 次测试" >&2
exit 0
```

---

### 5.2 使用 JSON 文件存储复杂状态

**场景：** 记录代码修改历史

**状态文件结构：**
```json
{
  "session_id": "abc123",
  "modifications": [
    {
      "timestamp": "2026-04-02T10:30:00Z",
      "file": "src/userService.js",
      "action": "edit",
      "lines_changed": 45
    },
    {
      "timestamp": "2026-04-02T10:35:00Z",
      "file": "src/authController.js",
      "action": "write",
      "lines_changed": 120
    }
  ],
  "tests_run": 2,
  "last_build_status": "success"
}
```

**写入状态：**
```bash
#!/bin/bash

STATE_FILE="/tmp/code-changes-$$"

# 读取现有状态
if [ -f "$STATE_FILE" ]; then
  state=$(cat "$STATE_FILE")
else
  state='{"modifications": [], "tests_run": 0}'
fi

# 添加新的修改记录
new_mod=$(jq -n \
  --arg ts "$(date -u +"%Y-%m-%dT%H:%M:%SZ")" \
  --arg file "$file_path" \
  --arg action "$tool_name" \
  '{timestamp: $ts, file: $file, action: $action}')

# 更新状态
updated_state=$(echo "$state" | jq ".modifications += [$new_mod]")
echo "$updated_state" > "$STATE_FILE"
```

---

### 5.3 跨会话状态持久化

**场景：** 记住用户的偏好设置

```bash
#!/bin/bash

CONFIG_FILE="$HOME/.claude/my-plugin-config.json"

# 读取配置
if [ -f "$CONFIG_FILE" ]; then
  strict_mode=$(jq -r '.strict_mode' "$CONFIG_FILE")
else
  strict_mode="false"
fi

# 根据配置决定行为
if [ "$strict_mode" = "true" ]; then
  # 执行严格检查
  # ...
else
  # 执行标准检查
  # ...
fi
```

---

## 六、与其他系统集成

### 6.1 Slack 通知集成

**场景：** Hook 阻止操作时通知团队

```bash
#!/bin/bash

SLACK_WEBHOOK_URL="$SLACK_WEBHOOK_URL"  # 从环境变量读取

input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')
file_path=$(echo "$input" | jq -r '.tool_input.file_path // "N/A"')

# 发送通知到 Slack
curl -X POST "$SLACK_WEBHOOK_URL" \
  -H 'Content-Type: application/json' \
  -d "{
    \"text\": \"🚨 Hook 阻止了危险操作\",
    \"attachments\": [{
      \"color\": \"danger\",
      \"fields\": [
        {\"title\": \"工具\", \"value\": \"$tool_name\", \"short\": true},
        {\"title\": \"文件\", \"value\": \"$file_path\", \"short\": true},
        {\"title\": \"时间\", \"value\": \"$(date)\", \"short\": false}
      ]
    }]
  }" 2>/dev/null

# 继续执行 Hook 逻辑
# ...
```

---

### 6.2 数据库日志记录

**场景：** 审计所有 Hook 执行记录

```bash
#!/bin/bash

DB_URL="$DATABASE_URL"

input=$(cat)
event_type=$(echo "$input" | jq -r '.hook_event_name')
tool_name=$(echo "$input" | jq -r '.tool_name')

# 记录到数据库
psql "$DB_URL" -c \
  "INSERT INTO hook_logs (event_type, tool_name, raw_data, created_at) 
   VALUES ('$event_type', '$tool_name', '$input', NOW())" \
  2>/dev/null

exit 0
```

---

### 6.3 指标收集（StatsD/Prometheus）

**场景：** 监控 Hook 性能

```bash
#!/bin/bash

STATSD_HOST="statsd.local"
STATSD_PORT=8125

start_time=$(date +%s%N)

# 执行 Hook 逻辑
# ...

end_time=$(date +%s%N)
duration=$(( (end_time - start_time) / 1000000 ))  # 毫秒

# 发送指标
echo "hook.execution_time:${duration}|ms" | nc -u -w1 "$STATSD_HOST" "$STATSD_PORT"
echo "hook.executions:1|c" | nc -u -w1 "$STATSD_HOST" "$STATSD_PORT"
```

---

## 七、测试策略

### 7.1 单元测试

为每个 Hook 脚本编写单元测试：

**测试文件结构：**
```
hooks/
├── scripts/
│   └── validate-path.sh
└── tests/
    ├── test-validate-path.sh
    └── test-all.sh
```

**测试示例：**
```bash
#!/bin/bash
# test-validate-path.sh

SCRIPT_DIR="$(dirname "$0")"
VALIDATE_SCRIPT="$SCRIPT_DIR/../scripts/validate-path.sh"

# 测试用例 1：允许安全路径
echo '{"tool_input": {"file_path": "/tmp/test.txt"}}' | \
  bash "$VALIDATE_SCRIPT"
if [ $? -eq 0 ]; then
  echo "✅ Test 1 passed"
else
  echo "❌ Test 1 failed"
  exit 1
fi

# 测试用例 2：阻止系统路径
echo '{"tool_input": {"file_path": "/etc/passwd"}}' | \
  bash "$VALIDATE_SCRIPT"
if [ $? -eq 2 ]; then
  echo "✅ Test 2 passed"
else
  echo "❌ Test 2 failed"
  exit 1
fi

echo "All tests passed!"
```

---

### 7.2 集成测试

测试多个 Hook 的协作：

```bash
#!/bin/bash
# integration-test.sh

export CLAUDE_PROJECT_DIR="/tmp/test-project"
export CLAUDE_PLUGIN_ROOT="$(pwd)"

# 清理环境
rm -rf "$CLAUDE_PROJECT_DIR"
mkdir -p "$CLAUDE_PROJECT_DIR"

echo "🧪 Integration Test: Full Workflow"

# 1. 模拟 SessionStart
echo '{}' | bash hooks/scripts/session-start.sh
if [ -f "/tmp/session-initialized" ]; then
  echo "✅ SessionStart works"
else
  echo "❌ SessionStart failed"
  exit 1
fi

# 2. 模拟 PreToolUse
echo '{"tool_input": {"file_path": "/tmp/test.js", "content": "console.log(1)"}}' | \
  bash hooks/scripts/pre/security-check.sh
if [ $? -eq 0 ]; then
  echo "✅ PreToolUse works"
else
  echo "❌ PreToolUse failed"
  exit 1
fi

# 3. 模拟 Stop
echo '{"reason": "Task complete"}' | bash hooks/scripts/stop/completion-check.sh
if [ $? -eq 0 ]; then
  echo "✅ Stop works"
else
  echo "❌ Stop failed"
  exit 1
fi

# 清理
rm -rf "$CLAUDE_PROJECT_DIR"

echo "✅ All integration tests passed!"
```

---

## 八、文档化

### 8.1 为每个 Hook 编写说明

**模板：**
```markdown
## Hook 名称：SQL 注入检测

### 位置
`hooks/scripts/security/sql-injection-check.sh`

### 作用
检测代码中是否存在 SQL 注入风险

### 触发条件
- 工具：Write, Edit
- 内容：包含 SQL 查询字符串

### 检测规则
❌ 违规：字符串拼接 SQL
✅ 合规：参数化查询

### 输出示例
⚠️ Security Warning: SQL injection detected
File: src/userService.js:45
Pattern: "SELECT * FROM users WHERE id = " + userId
Suggestion: Use parameterized query

### 测试方法
bash tests/test-sql-injection.sh

### 负责人
@team-security
```

---

### 8.2 维护变更日志

**CHANGELOG.md：**
```markdown
# Hook 系统变更日志

## [1.2.0] - 2026-04-02

### Added
- 新增 XSS 检测 Hook
- 添加性能监控指标

### Changed
- 优化 SQL 注入检测算法（性能提升 50%）
- 调整超时设置

### Fixed
- 修复路径验证的正则表达式 bug
- 修复缓存清理逻辑

### Deprecated
- 旧版 path-check.sh（将在 v2.0 移除）

### Security
- 增强敏感文件检测规则
```

---

## 九、版本控制

### 9.1 Git 工作流

**分支策略：**
```
main (生产)
  ↓
develop (开发)
  ↓
feature/* (功能分支)
```

**提交规范：**
```bash
# 格式
<type>(scope): <subject>

# 示例
feat(security): 添加 XSS 检测 Hook
fix(validation): 修复路径验证 bug
docs(readme): 更新安装说明
perf(cache): 优化缓存命中率
```

---

### 9.2 标签和发布

```bash
# 打标签
git tag -a v1.2.0 -m "Release version 1.2.0"

# 推送标签
git push origin v1.2.0

# 创建 Release（GitHub）
gh release create v1.2.0 \
  --title "Release v1.2.0" \
  --notes "See CHANGELOG.md for details"
```

---

## 十、最佳实践总结

### ✅ 推荐做法

1. **选择合适的组织结构**
   - 小项目：集中式
   - 中项目：按事件
   - 大项目：按功能

2. **充分测试**
   - 单元测试覆盖每个脚本
   - 集成测试验证协作
   - 性能测试确保响应速度

3. **详细文档**
   - 每个 Hook 都有说明
   - 维护变更日志
   - 提供使用示例

4. **持续优化**
   - 监控性能指标
   - 收集用户反馈
   - 定期重构改进

### ❌ 避免的错误

1. **组织混乱**
   ```
   ❌ 所有文件堆在一起
   ✅ 清晰的目录结构
   ```

2. **缺乏测试**
   ```
   ❌ 写完直接用
   ✅ 先测试后部署
   ```

3. **没有文档**
   ```
   ❌ 只有代码
   ✅ 代码 + 文档 + 示例
   ```

4. **忽视性能**
   ```
   ❌ 每个 Hook 都执行 5 秒+
   ✅ 优化到 1 秒内
   ```

---

## 十一、参考资源

### 项目中的示例
- 集中式：`plugins/security-guidance/`
- 按事件：`plugins/hookify/`
- 按功能：`plugins/plugin-dev/`

### 工具脚本
- 验证器：`plugins/plugin-dev/scripts/validate-hook-schema.sh`
- 测试器：`plugins/plugin-dev/scripts/test-hook.sh`
- Linter: `plugins/plugin-dev/scripts/hook-linter.sh`

### 文档
- Hook API: `plugins/plugin-dev/skills/hook-development/SKILL.md`
- 高级模式：`plugins/plugin-dev/skills/hook-development/references/advanced.md`

---

**下一步：** 学习如何 [开发完整的插件](../07-插件系统/01-插件系统详解.md)！
