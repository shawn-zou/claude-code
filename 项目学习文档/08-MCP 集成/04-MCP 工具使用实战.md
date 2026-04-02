# MCP 工具使用实战 - Commands/Agents 中的 MCP 工具调用

## 📋 本节学习目标

学完本节后，你将能够：
- ✅ 在 Commands 中正确使用 MCP 工具
- ✅ 在 Agents 中配置 MCP 工具自主调用
- ✅ 掌握常见的工具调用模式
- ✅ 实现健壮的错误处理和重试机制

---

## 🎯 工具命名规则回顾

### 完整的工具名格式

```
mcp__plugin_<插件名>_<服务器名>__<工具名>
```

### 实际例子

| 组件 | 值 | 示例 |
|------|-----|------|
| 固定前缀 | `mcp__plugin_` | `mcp__plugin_` |
| 插件名称 | 你的插件名 | `asana` |
| 服务器名称 | MCP 服务器名 | `asana` |
| 工具名称 | 具体工具名 | `create_task` |
| **完整工具名** | - | `mcp__plugin_asana_asana__create_task` |

---

## 🔍 如何发现可用工具？

### 方法 1: 使用 /mcp 命令

```bash
/mcp
```

**输出示例：**
```
已配置的 MCP 服务器:

📌 asana (SSE)
   URL: https://mcp.asana.com/sse
   状态：已连接
   
   可用工具:
   - mcp__plugin_asana_asana__create_task
     描述：创建新的 Asana 任务
     参数:
       - name (string, 必需): 任务标题
       - notes (string): 任务描述
       - workspace (string): 工作区 ID
       
   - mcp__plugin_asana_asana__search_tasks
     描述：搜索任务
     参数:
       - project (string): 项目 ID
       - assignee (string): 负责人 ID
       - completed (boolean): 是否完成
```

### 方法 2: 查看工具 Schema

每个工具都有详细的输入模式定义：

```json
{
  "name": "create_task",
  "description": "Create a new Asana task",
  "inputSchema": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string",
        "description": "Task title"
      },
      "notes": {
        "type": "string",
        "description": "Task description"
      },
      "workspace": {
        "type": "string",
        "description": "Workspace GID"
      }
    },
    "required": ["name", "workspace"]
  }
}
```

**重要信息：**
- ✅ 必需参数（required 数组）
- ✅ 参数类型（string/number/boolean）
- ✅ 参数描述
- ✅ 默认值（如果有）

---

## 1️⃣ 在 Commands 中使用 MCP 工具

### Command 的基本结构

```markdown
---
description: 命令的简短描述
allowed-tools: [
  "允许使用的工具列表"
]
---

# 命令标题

命令的详细步骤...
```

---

### 示例 1: 简单的单工具调用

```markdown
---
description: 创建新的 Asana 任务
allowed-tools: [
  "mcp__plugin_asana_asana__create_task"
]
---

# 创建 Asana 任务

## 角色

你是一个 Asana 任务管理助手，帮助用户快速创建任务。

## 工作流程

### Step 1: 收集任务信息

向用户询问以下信息：

1. **任务标题**（必需）
   - 简短描述任务内容
   
2. **任务描述**（可选）
   - 详细说明
   
3. **截止日期**（可选）
   - 格式：YYYY-MM-DD
   
4. **负责人**（可选）
   - Asana 用户 ID 或邮箱

### Step 2: 验证信息

检查必需字段：
- ✅ 任务标题不为空
- ✅ 如果有截止日期，验证格式正确

### Step 3: 调用 MCP 工具

使用工具创建任务：

```
调用：mcp__plugin_asana_asana__create_task
参数：
{
  "name": "任务标题",
  "notes": "任务描述",
  "due_on": "2026-04-15"
}
```

### Step 4: 处理响应

成功时：
```
✅ 任务创建成功！

标题：{task_name}
ID: {task_id}
链接：{task_url}
```

失败时：
```
❌ 创建任务失败

错误原因：{error_message}
请检查：
1. 你是否已授权 Asana
2. 工作区是否存在
3. 网络连接是否正常
```

### Step 5: 确认

询问用户：
- 是否需要创建更多任务？
- 是否需要查看刚创建的任务？
```

---

### 示例 2: 多工具顺序调用

```markdown
---
description: 搜索并更新 Asana 任务
allowed-tools: [
  "mcp__plugin_asana_asana__search_tasks",
  "mcp__plugin_asana_asana__update_task",
  "mcp__plugin_asana_asana__get_task"
]
---

# 批量更新任务状态

## 场景

用户想要批量更新符合某些条件的任务状态。

## 完整流程

### Phase 1: 搜索任务

**调用工具:** `mcp__plugin_asana_asana__search_tasks`

```typescript
参数:
{
  "project": "123456789",      // 项目 ID
  "completed": false,          // 未完成的任务
  "assignee": "987654321"      // 特定负责人
}
```

**预期响应:**
```json
{
  "tasks": [
    {
      "gid": "111",
      "name": "任务 1",
      "completed": false
    },
    {
      "gid": "222",
      "name": "任务 2",
      "completed": false
    }
  ]
}
```

### Phase 2: 显示搜索结果

向用户展示：
```
找到 5 个未完成任务:

□ 任务 1 (ID: 111)
□ 任务 2 (ID: 222)
□ 任务 3 (ID: 333)
□ 任务 4 (ID: 444)
□ 任务 5 (ID: 555)

你想将这些任务都标记为已完成吗？
(是/否/选择部分任务)
```

### Phase 3: 批量更新

如果用户确认，逐个更新任务：

**循环调用:** `mcp__plugin_asana_asana__update_task`

对每个任务：
```typescript
参数:
{
  "task_id": "111",
  "completed": true
}
```

**进度跟踪:**
```
正在更新任务...
✓ 任务 1 (1/5)
✓ 任务 2 (2/5)
✓ 任务 3 (3/5)
⏳ 任务 4 (4/5)
...
```

### Phase 4: 总结

更新完成后报告：
```
✅ 批量更新完成！

成功：5 个任务
失败：0 个任务

所有任务已标记为已完成。
```
```

---

### 示例 3: 带错误处理的复杂流程

```markdown
---
description: 创建项目并初始化任务模板
allowed-tools: [
  "mcp__plugin_asana_asana__create_project",
  "mcp__plugin_asana_asana__create_task",
  "mcp__plugin_asana_asana__add_project_member"
]
---

# 创建新项目并初始化

## 高级工作流

这个命令展示如何处理复杂的错误情况和重试逻辑。

## 详细步骤

### Step 1: 创建项目

**工具:** `mcp__plugin_asana_asana__create_project`

```typescript
参数:
{
  "name": "新用户入职培训",
  "owner": "123456",
  "team": "789012"
}
```

**错误处理:**

```markdown
如果创建失败:

1. **检查错误类型:**
   
   - 403 Forbidden → 权限不足
     - 提示："你没有权限创建项目，请联系管理员"
   
   - 400 Bad Request → 参数错误
     - 提示："项目名称可能重复，请换一个名字"
   
   - Network Error → 网络问题
     - 重试策略：等待 2 秒，最多重试 3 次
   
2. **提供替代方案:**
   - "或者，我可以帮你在现有项目中创建任务"
```

### Step 2: 添加项目成员

**工具:** `mcp__plugin_asana_asana__add_project_member`

```typescript
参数:
{
  "project_id": "{上一步返回的项目 ID}",
  "user_id": "新用户 ID",
  "role": "member"
}
```

**并发处理:**

如果需要添加多个成员：
```typescript
// 并行调用（提高效率）
Promise.all([
  add_member(user1),
  add_member(user2),
  add_member(user3)
])
```

**错误容忍:**
```markdown
如果某个成员添加失败:
- 记录失败的用户列表
- 继续处理其他用户
- 最后统一报告失败情况
```

### Step 3: 创建任务模板

**工具:** `mcp__plugin_asana_asana__create_task`

创建标准任务模板：

```typescript
const templates = [
  { name: "完成入职表格", due_offset: 0 },
  { name: "参加安全培训", due_offset: 1 },
  { name: "与导师见面", due_offset: 3 },
  { name: "完成第一个任务", due_offset: 7 }
];

for (const template of templates) {
  await create_task({
    name: template.name,
    project: project_id,
    due_on: calculate_date(today, template.due_offset)
  });
}
```

### Step 4: 完整性检查

创建完成后验证：

```markdown
检查清单:
- [ ] 项目创建成功
- [ ] 所有成员添加成功
- [ ] 至少创建了 3 个任务
- [ ] 项目可见性正确

如果任何一项失败:
→ 回滚操作（删除部分创建的内容）
→ 通知用户手动检查
```

### Step 5: 最终报告

```markdown
## ✅ 项目创建完成！

### 项目信息
- 名称：新用户入职培训
- ID: 123456789
- 链接：https://app.asana.com/0/123456789

### 添加的成员 (3 人)
✓ 张三 (zhang@example.com)
✓ 李四 (li@example.com)
✓ 王五 (wang@example.com)

### 创建的任务 (4 个)
□ 完成入职表格 (今天截止)
□ 参加安全培训 (明天截止)
□ 与导师见面 (3 天后截止)
□ 完成第一个任务 (7 天后截止)

### 下一步建议
1. 自定义任务模板
2. 邀请更多团队成员
3. 设置项目里程碑
```
```

---

## 2️⃣ 在 Agents 中使用 MCP 工具

### Agent 与 Command 的区别

| 特性 | Command | Agent |
|------|---------|-------|
| **工具权限** | 需要预声明 (allowed-tools) | 自动获得所有工具权限 |
| **自主性** | 按步骤执行 | 可自主决策 |
| **使用场景** | 明确定义的任务 | 复杂、需要判断的任务 |
| **配置复杂度** | 简单 | 较复杂 |

---

### Agent 配置模板

```markdown
---
name: asana-project-manager
description: 这个 agent 应该被用于管理 Asana 项目，包括创建项目、分配任务、跟踪进度等
model: sonnet
color: blue
---

## 角色定位

你是 Asana 项目管理专家，帮助用户高效管理项目和任务。

## 核心能力

### 1. 项目管理
- 创建和配置项目
- 设置项目里程碑
- 管理项目成员
- 跟踪项目进度

### 2. 任务管理
- 创建、更新、删除任务
- 分配任务给团队成员
- 设置任务优先级和截止日期
- 批量操作任务

### 3. 数据分析
- 生成项目状态报告
- 分析团队工作量
- 识别延期风险
- 提供优化建议

## 可用工具

你有权使用所有 Asana MCP 工具：
- `mcp__plugin_asana_asana__create_project`
- `mcp__plugin_asana_asana__create_task`
- `mcp__plugin_asana_asana__update_task`
- `mcp__plugin_asana_asana__search_tasks`
- `mcp__plugin_asana_asana__delete_task`
- `mcp__plugin_asana_asana__add_project_member`
- ... (无需全部列出)

## 工作流程

### 场景 1: 创建新项目

当用户要求创建项目时：

1. **需求澄清**
   ```
   为了创建项目，我需要了解：
   - 项目名称和目标
   - 团队成员有哪些
   - 项目时间线
   - 关键里程碑
   ```

2. **创建项目**
   ```
   调用 create_project API
   保存返回的项目 ID
   ```

3. **初始化项目**
   ```
   - 添加项目成员
   - 创建初始任务
   - 设置项目模板
   ```

4. **确认和交接**
   ```
   向用户展示项目详情
   提供项目链接
   建议下一步行动
   ```

### 场景 2: 批量更新任务

当用户需要批量操作时：

1. **确认操作范围**
   ```
   - 哪些任务需要更新
   - 更新什么字段
   - 是否有特殊条件
   ```

2. **搜索目标任务**
   ```
   使用 search_tasks 找到符合条件的任务
   向用户确认列表
   ```

3. **执行批量更新**
   ```
   - 并行调用 update_task（如果支持）
   - 或串行调用（如果需要依赖前一个结果）
   - 跟踪进度和错误
   ```

4. **报告结果**
   ```
   成功：X 个任务
   失败：Y 个任务（说明原因）
   ```

### 场景 3: 生成项目报告

当用户需要项目状态报告时：

1. **收集数据**
   ```
   - 获取项目详情
   - 获取所有任务列表
   - 获取团队成员信息
   ```

2. **分析数据**
   ```
   - 计算完成率
   - 识别延期任务
   - 分析工作量分布
   ```

3. **生成报告**
   ```markdown
   # 项目状态报告
   
   ## 总体进度
   - 总任务数：50
   - 已完成：35 (70%)
   - 进行中：10
   - 未开始：5
   
   ## 风险识别
   ⚠️ 3 个任务已延期
   ⚠️ 2 个任务即将到期
   
   ## 团队负载
   👤 张三：8 个任务（高负载）
   👤 李四：5 个任务（正常）
   👤 王四：3 个任务（低负载）
   
   ## 建议
   1. 优先处理延期的 3 个任务
   2. 考虑重新分配张三的部分任务
   3. 给王四分配更多任务
   ```

## 错误处理策略

### 通用错误处理

```markdown
遇到错误时的处理流程：

1. **识别错误类型**
   - API 错误（4xx/5xx）
   - 网络错误
   - 认证错误
   - 业务逻辑错误

2. **采取相应行动**
   
   - API 限流 (429)
     → 等待 Retry-After 秒后重试
   
   - 认证失败 (401)
     → 提示用户重新授权
   
   - 资源不存在 (404)
     → 确认 ID 是否正确
   
   - 服务器错误 (5xx)
     → 重试 2-3 次，仍失败则报告

3. **给用户清晰的反馈**
   ```
   ❌ 操作失败
   
   错误类型：权限不足
   原因：你没有权限执行此操作
   解决方案：联系项目管理员授予权限
   
   错误详情：403 Forbidden
   ```
```

### 重试策略

```markdown
对于临时性错误，实施指数退避重试：

第 1 次重试：等待 1 秒
第 2 次重试：等待 2 秒
第 3 次重试：等待 4 秒

最大重试次数：3 次

重试后仍失败 → 报告错误并提供替代方案
```

## 最佳实践

### 1. 工具调用优化

```markdown
✅ 好的做法:
- 批量查询而非多次单个查询
- 并行调用无依赖的 API
- 缓存常用数据（如用户列表）

❌ 避免的做法:
- 在循环中逐个查询（N+1 问题）
- 重复查询相同的数据
- 忽略 API 限流
```

### 2. 用户体验优化

```markdown
- 操作前确认（特别是删除操作）
- 提供进度反馈
- 操作后总结
- 给出下一步建议
```

### 3. 数据安全

```markdown
- 不记录敏感信息（Token、密码）
- 删除操作前备份
- 验证用户权限
- 遵循最小权限原则
```
```

---

## 3️⃣ 常见工具调用模式

### 模式 1: CRUD 操作

```markdown
## Create（创建）

Steps:
1. 验证必需参数
2. 调用 create_* 工具
3. 验证创建成功
4. 返回新资源的信息

## Read（读取）

Steps:
1. 调用 get_* 或 list_* 工具
2. 处理返回的数据
3. 格式化展示给用户

## Update（更新）

Steps:
1. 获取当前状态（可选）
2. 调用 update_* 工具
3. 验证更新成功
4. 展示更新后的状态

## Delete（删除）

Steps:
1. ⚠️ 确认删除（重要！）
2. 调用 delete_* 工具
3. 验证删除成功
4. 提供撤销方案（如果有）
```

---

### 模式 2: 搜索 - 过滤 - 处理

```markdown
## 三步模式

### Step 1: 搜索

```typescript
调用：search_*
参数：宽泛的搜索条件
目的：获取候选集
```

### Step 2: 过滤

```typescript
在本地进一步过滤
应用业务规则
缩小范围
```

### Step 3: 处理

```typescript
对过滤后的结果
批量或逐个处理
```

## 实际例子

```markdown
场景：找出所有逾期的任务并发送提醒

Step 1: 搜索
→ 搜索所有未完成任务

Step 2: 过滤  
→ 在本地检查每个任务的 due_date < today

Step 3: 处理
→ 对逾期任务调用 send_reminder
```
```

---

### 模式 3: 创建依赖链

```markdown
## 有依赖的创建顺序

某些资源的创建有依赖关系：

```
创建用户
    ↓ (获得 user_id)
创建项目并设置 owner=user_id
    ↓ (获得 project_id)
创建任务并设置 project=project_id
    ↓
发送通知
```

## 错误回滚

如果后续步骤失败：

```
任务创建失败
    ↓
删除刚创建的项目
    ↓
（可选）删除刚创建的用户
    ↓
报告错误
```
```

---

### 模式 4: 并行批处理

```markdown
## 适用场景

- 批量更新独立的任务
- 同时添加多个项目成员
- 并行查询多个资源

## 实现方式

```typescript
// JavaScript 伪代码
const taskIds = [1, 2, 3, 4, 5];

// 并行调用
const results = await Promise.all(
  taskIds.map(id => update_task({ id, completed: true }))
);

// 统计结果
const success = results.filter(r => r.success).length;
const failed = results.length - success;
```

## 注意事项

```markdown
✅ 适合：
- 操作之间无依赖
- API 支持并发
- 不触发限流

❌ 不适合：
- 每个操作依赖前一个的结果
- 资源之间有约束
- 可能触发速率限制
```
```

---

### 模式 5: 轮询等待

```markdown
## 适用场景

某些操作是异步的：
- 长时间运行的任务
- 第三方集成
- 审批流程

## 轮询模式

```markdown
Steps:
1. 启动异步操作
   → 获得 operation_id

2. 定期查询状态
   → 每 5 秒调用 get_status(operation_id)

3. 检查完成条件
   → status === "completed"

4. 超时处理
   → 如果超过 5 分钟未完成，报错

5. 获取最终结果
   → 调用 get_result(operation_id)
```

## 实际例子

```markdown
场景：导出项目报告

1. 调用 export_project(project_id)
   → 返回 export_id

2. 轮询检查:
   while (true):
     status = get_export_status(export_id)
     
     if status === "completed":
       download_export(export_id)
       break
     
     if status === "failed":
       report_error()
       break
     
     wait(5 秒)
```
```

---

## 4️⃣ 错误处理详解

### 错误类型分类

```markdown
## 1. 客户端错误 (4xx)

### 400 Bad Request
原因：参数错误
解决：验证参数格式和类型

### 401 Unauthorized  
原因：未认证或 Token 过期
解决：重新认证或刷新 Token

### 403 Forbidden
原因：权限不足
解决：申请相应权限

### 404 Not Found
原因：资源不存在
解决：检查 ID 是否正确

### 429 Too Many Requests
原因：触发限流
解决：等待后重试

## 2. 服务端错误 (5xx)

### 500 Internal Server Error
原因：服务器内部错误
解决：稍后重试

### 502 Bad Gateway
原因：网关错误
解决：检查服务可用性

### 503 Service Unavailable
原因：服务不可用
解决：等待服务恢复

## 3. 网络错误

原因：连接超时、DNS 解析失败
解决：检查网络连接
```

---

### 错误处理模板

```markdown
## 通用错误处理框架

```markdown
Steps:
1. 尝试执行操作

2. 如果失败，捕获错误并分类:
   
   case 400:
     message = "请求参数有误"
     suggestion = "请检查：1... 2... 3..."
   
   case 401:
     message = "认证失败"
     suggestion = "请重新登录或检查 Token 配置"
   
   case 403:
     message = "权限不足"
     suggestion = "需要 XX 权限，请联系管理员"
   
   case 404:
     message = "资源不存在"
     suggestion = "请确认 ID 是否正确"
   
   case 429:
     message = "请求过于频繁"
     suggestion = "等待 X 秒后重试"
     retry_after = parse_retry_header()
   
   case 5xx:
     message = "服务器错误"
     suggestion = "稍后重试"
     retry_count++
   
   default:
     message = "未知错误"
     suggestion = "查看详细错误日志"

3. 向用户展示友好的错误信息

4. 记录详细日志（用于调试）

5. 根据错误类型决定：
   - 重试
   - 回滚
   - 终止
   - 提供替代方案
```
```

---

### 重试策略实现

```markdown
## 指数退避算法

```javascript
async function callWithRetry(toolCall, maxRetries = 3) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await toolCall();
    } catch (error) {
      lastError = error;
      
      // 只有 5xx 和网络错误才重试
      if (!isRetryableError(error)) {
        throw error;
      }
      
      // 计算等待时间（指数退避）
      const delay = Math.pow(2, attempt) * 1000; // 2s, 4s, 8s
      await sleep(delay);
    }
  }
  
  throw new Error(`重试${maxRetries}次后仍然失败：${lastError.message}`);
}

function isRetryableError(error) {
  // 5xx 错误
  if (error.status >= 500 && error.status < 600) {
    return true;
  }
  
  // 网络错误
  if (error.type === 'network-error') {
    return true;
  }
  
  // 限流错误（如果有 Retry-After 头）
  if (error.status === 429) {
    return true;
  }
  
  return false;
}
```

## 实际应用示例

```markdown
场景：创建任务，可能遇到各种错误

Steps:
1. 尝试验证参数
   - 失败 → 立即返回错误（不重试）

2. 尝试调用 API
   - 401 → 提示重新认证（不重试）
   - 403 → 提示权限不足（不重试）
   - 429 → 等待后重试
   - 500 → 等待后重试
   - 网络错误 → 等待后重试

3. 重试逻辑:
   第 1 次重试：等待 2 秒
   第 2 次重试：等待 4 秒
   第 3 次重试：等待 8 秒
   
   仍然失败 → 报告错误

4. 成功 → 返回结果
```
```

---

## 📝 学习检查清单

完成本节后，你应该能够：

- [ ] 解释 MCP 工具的命名规则
- [ ] 使用 `/mcp` 命令查看可用工具
- [ ] 在 Commands 中预声明 MCP 工具
- [ ] 在 Agents 中自主使用 MCP 工具
- [ ] 实现基本的错误处理逻辑
- [ ] 应用指数退避重试策略
- [ ] 说出 3 种常见的工具调用模式

---

## 💡 常见问题

### Q1: allowed-tools 必须列出所有使用的工具吗？

**答：** 是的，Commands 必须在 frontmatter 中声明所有可能用到的工具。这是安全机制，防止未授权的访问。

### Q2: Agent 可以无条件使用任何工具吗？

**答：** 理论上是的，但应该在 Agent 定义中说明通常会使用哪些工具，便于理解和维护。

### Q3: 如何处理工具的异步操作？

**答：** 使用轮询模式或回调机制。有些 MCP 服务器支持 WebSocket 推送结果。

### Q4: 工具调用失败就一定要重试吗？

**答：** 不是。只有临时性错误（5xx、网络问题、限流）才重试。永久性错误（400、401、403）应该直接报告。

---

## 🎯 实战练习

### 练习 1: 设计 Command

为一个 GitHub MCP 插件设计 Command：
- 功能：创建 Pull Request
- 需要哪些工具？
- 写出完整的 Command 框架

**参考答案：**
```markdown
---
description: 创建 GitHub Pull Request
allowed-tools: [
  "mcp__plugin_github_github__create_pull_request",
  "mcp__plugin_github_github__get_branch",
  "mcp__plugin_github_github__list_commits"
]
---

# 创建 Pull Request

## 步骤

1. 收集必需信息（仓库、分支、标题、描述）
2. 验证源分支和目标分支存在
3. 调用 create_pull_request
4. 处理响应并返回 PR 链接
```

### 练习 2: 错误处理设计

为数据库查询操作设计错误处理：
- 可能遇到哪些错误？
- 每种错误如何处理？
- 哪些错误应该重试？

**参考答案：**
```markdown
可能的错误:
1. 连接失败 → 重试 3 次
2. SQL 语法错误 → 不重试，直接报错
3. 超时 → 重试 2 次
4. 权限不足 → 不重试，提示用户
5. 表不存在 → 不重试，检查表名
```

---

## 🔗 相关资源

### 参考文档
- [tool-usage.md](../../plugins/plugin-dev/skills/mcp-integration/references/tool-usage.md) - 完整工具使用指南

### 下一篇预告
下一节我们将学习 **MCP 调试与故障排除**，包括：
- 启用调试日志
- 常见连接问题排查
- 使用调试工具
- 性能优化技巧

---

**恭喜你完成了 MCP 工具使用实战的学习！** 🎉

接下来，让我们学习 [MCP 调试与故障排除](./05-MCP 调试与故障排除.md)。
