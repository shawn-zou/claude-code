# MCP 最佳实践与安全 - 安全规范与性能优化

## 📋 本节学习目标

学完本节后，你将能够：
- ✅ 遵循 MCP 安全编码规范
- ✅ 实施最小权限原则
- ✅ 优化 MCP 性能
- ✅ 在生产环境安全部署
- ✅ 建立监控和告警机制

---

## 🔐 安全最佳实践

### 原则 1: 最小权限原则（Principle of Least Privilege）

#### 什么是_min_权限原则？

**定义：** 只授予执行任务所必需的最小权限，不多不少。

**为什么重要？**
- ✅ 限制潜在损害范围
- ✅ 降低误操作风险
- ✅ 符合安全合规要求

---

#### 实践指南

##### OAuth Scopes 的最小化

```json
// ❌ 错误：请求过多权限
{
  "scopes": [
    "read:*",
    "write:*",
    "delete:*"
  ]
}

// ✅ 正确：只请求需要的权限
{
  "scopes": [
    "read:tasks",      // 只读取任务
    "write:tasks"      // 只写入任务
    // 不需要删除权限
  ]
}
```

**实际案例：**

场景：创建一个只需要读取任务的插件

```markdown
## 权限分析

需要的功能：
1. 读取任务列表 ✓ read:tasks
2. 查看任务详情 ✓ read:tasks  
3. 搜索任务 ✓ read:tasks

不需要的功能：
- 创建任务 ✗ write:tasks
- 更新任务 ✗ write:tasks
- 删除任务 ✗ delete:tasks

最终配置：
{
  "scopes": ["read:tasks"]
}
```

---

##### Token 权限的最小化

```bash
# ❌ 错误：使用管理员 Token
export API_TOKEN="admin_full_access_token"

# ✅ 正确：创建专用受限 Token
# 在 API 服务中创建只读 Token
curl -X POST https://api.example.com/tokens \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -d '{
    "name": "claude-plugin-readonly",
    "scopes": ["read:only"],
    "expires_in": 86400
  }'
```

---

##### 文件系统访问的最小化

```json
// ❌ 错误：允许访问整个文件系统
{
  "filesystem": {
    "command": "npx",
    "args": ["@modelcontextprotocol/server-filesystem", "/"]
  }
}

// ✅ 正确：只允许访问项目目录
{
  "filesystem": {
    "command": "npx",
    "args": ["@modelcontextprotocol/server-filesystem", "/Users/username/projects/myapp"]
  }
}

// ✅ 更好：只允许访问特定子目录
{
  "filesystem": {
    "command": "npx",
    "args": ["@modelcontextprotocol/server-filesystem", "/Users/username/projects/myapp/src"]
  }
}
```

---

### 原则 2: 凭证安全管理

#### 凭证存储的最佳实践

##### 层级 1: 环境变量（基础）

```bash
# 设置环境变量
export DB_PASSWORD="secret123"

# 在配置中使用
{
  "env": {
    "DB_PASSWORD": "${DB_PASSWORD}"
  }
}
```

**优点：**
- ✅ 不硬编码在配置文件中
- ✅ 易于管理
- ✅ 支持不同环境

**缺点：**
- ❌ 可能被其他进程读取
- ❌ 可能在日志中泄露

---

##### 层级 2: 加密存储（推荐）

**macOS Keychain:**
```bash
# 存储密码
security add-generic-password -s claude-plugin -a db_password -w "secret123"

# 读取密码
security find-generic-password -s claude-plugin -a db_password -w
```

**Windows Credential Manager:**
```powershell
# 使用 PowerShell 管理凭据
# 需要 CredentialManager 模块
Install-Module -Name CredentialManager

# 存储凭据
New-StoredCredential -Target "ClaudePlugin" `
                     -UserName "db_user" `
                     -Password "secret123" `
                     -Persist LocalMachine
```

**Linux Secret Service:**
```bash
# 使用 secret-tool
secret-tool store --label='Claude Plugin DB' \
  service 'database' \
  username 'db_user' \
  password 'secret123'
```

---

##### 层级 3: 密钥管理服务（企业级）

**AWS Secrets Manager:**
```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getSecret(secretName) {
  const response = await secretsManager.getSecretValue({
    SecretId: secretName
  }).promise();
  
  return JSON.parse(response.SecretString);
}

// 使用
const credentials = await getSecret('prod/db/credentials');
console.log(credentials.password); // 安全获取
```

**HashiCorp Vault:**
```bash
# 启用 Vault
vault server -dev

# 存储密钥
vault kv put secret/claude-plugin/db \
  username=db_user \
  password=secret123

# 读取密钥
vault kv get secret/claude-plugin/db
```

---

#### 凭证轮换策略

##### 自动轮换时间表

| 凭证类型 | 轮换周期 | 提醒方式 |
|----------|----------|----------|
| 数据库密码 | 30 天 | 邮件 + 日历 |
| API Token | 90 天 | 邮件 |
| OAuth Refresh Token | 180 天 | 应用内通知 |
| SSH 密钥 | 1 年 | 邮件 |

##### 轮换流程

```markdown
## Token 轮换步骤

### Step 1: 生成新 Token

在管理后台：
1. 创建新的 API Token
2. 设置适当的权限
3. 记录新 Token ID

### Step 2: 并行运行期

保持新旧 Token 同时有效（24-48 小时）：
- 更新配置使用新 Token
- 监控旧 Token 的使用情况
- 确保所有服务切换完成

### Step 3: 撤销旧 Token

确认无使用后：
1. 在管理后台撤销旧 Token
2. 从所有存储中删除
3. 记录撤销时间

### Step 4: 验证

测试新 Token：
- 基本功能测试
- 权限验证
- 性能测试
```

---

### 原则 3: 输入验证

#### 为什么要验证输入？

**风险：**
- SQL 注入攻击
- 命令注入攻击
- 路径遍历攻击
- XSS 跨站脚本

---

#### 验证模式

##### 模式 1: White List（白名单）

```javascript
// ✅ 推荐：只允许已知安全的值
const ALLOWED_SORT_FIELDS = ['name', 'created_at', 'priority'];
const ALLOWED_SORT_ORDERS = ['asc', 'desc'];

function validateSort(field, order) {
  if (!ALLOWED_SORT_FIELDS.includes(field)) {
    throw new Error(`Invalid sort field: ${field}`);
  }
  if (!ALLOWED_SORT_ORDERS.includes(order)) {
    throw new Error(`Invalid sort order: ${order}`);
  }
}
```

##### 模式 2: 类型检查

```javascript
// ✅ 严格类型检查
function createTask(params) {
  // 检查必需字段
  if (typeof params.name !== 'string') {
    throw new Error('Name must be a string');
  }
  
  if (typeof params.completed !== 'boolean') {
    throw new Error('Completed must be a boolean');
  }
  
  if (params.due_date && !isValidDate(params.due_date)) {
    throw new Error('Due date must be a valid date');
  }
}
```

##### 模式 3: 范围限制

```javascript
// ✅ 限制数值范围
function searchTasks(options) {
  // 限制返回数量
  if (options.limit < 1 || options.limit > 100) {
    options.limit = 50; // 使用默认值
  }
  
  // 限制分页
  if (options.page < 1) {
    options.page = 1;
  }
}
```

##### 模式 4: 清理危险字符

```javascript
// ✅ 清理文件路径
function sanitizePath(userPath) {
  // 移除 .. 防止路径遍历
  const clean = userPath.replace(/\.\./g, '');
  
  // 确保在允许的目录内
  const allowedBase = '/projects/myapp';
  const fullPath = path.join(allowedBase, clean);
  
  if (!fullPath.startsWith(allowedBase)) {
    throw new Error('Path traversal detected');
  }
  
  return fullPath;
}
```

---

### 原则 4: 输出编码

#### 防止信息泄露

```javascript
// ❌ 错误：暴露内部错误详情
try {
  await database.query(userInput);
} catch (error) {
  res.send(`Database error: ${error.message} at ${error.stack}`);
  // 暴露了 SQL 语句和堆栈信息
}

// ✅ 正确：友好的错误消息
try {
  await database.query(userInput);
} catch (error) {
  logger.error('Database query failed', error);
  res.send('操作失败，请稍后重试');
  // 用户看到友好提示，详细错误记录在日志
}
```

---

#### 敏感数据脱敏

```javascript
// ✅ 脱敏显示
function maskSensitiveData(data) {
  // 密码
  if (data.password) {
    data.password = '***';
  }
  
  // API Key（只显示前 8 位和后 4 位）
  if (data.api_key) {
    data.api_key = data.api_key.substring(0, 8) + '...' + data.api_key.substring(data.api_key.length - 4);
  }
  
  // 邮箱（隐藏部分）
  if (data.email) {
    const [user, domain] = data.email.split('@');
    data.email = user.substring(0, 2) + '***@' + domain;
  }
  
  return data;
}
```

---

## ⚡ 性能优化

### 优化 1: 批量查询

#### N+1 问题示例

```javascript
// ❌ 错误：N+1 查询
const tasks = await search_tasks({ project_id: 123 });

for (const task of tasks) {
  // 每个任务都要查询一次负责人
  const user = await get_user(task.assignee_id);
  console.log(`${task.name} - ${user.name}`);
}

// 如果有 100 个任务，就要执行 101 次查询！
```

#### 批量查询优化

```javascript
// ✅ 正确：批量查询
const tasks = await search_tasks({ project_id: 123 });

// 收集所有用户 ID
const userIds = [...new Set(tasks.map(t => t.assignee_id))];

// 一次性查询所有用户
const users = await get_users_by_ids(userIds);
const userMap = new Map(users.map(u => [u.id, u]));

// 使用缓存的用户数据
for (const task of tasks) {
  const user = userMap.get(task.assignee_id);
  console.log(`${task.name} - ${user.name}`);
}

// 只执行 2 次查询！
```

---

### 优化 2: 智能缓存

#### 缓存策略

```javascript
class MCPCache {
  constructor() {
    this.cache = new Map();
    this.ttl = new Map();
  }
  
  // 带 TTL 的缓存
  async get(key, fetchFn, ttlSeconds = 300) {
    const cached = this.cache.get(key);
    const expiry = this.ttl.get(key);
    
    // 检查缓存是否有效
    if (cached && expiry && Date.now() < expiry) {
      return cached;
    }
    
    // 缓存失效，重新获取
    const value = await fetchFn();
    this.cache.set(key, value);
    this.ttl.set(key, Date.now() + ttlSeconds * 1000);
    
    return value;
  }
  
  // 清除特定缓存
  invalidate(pattern) {
    for (const key of this.cache.keys()) {
      if (key.match(pattern)) {
        this.cache.delete(key);
        this.ttl.delete(key);
      }
    }
  }
}

// 使用示例
const cache = new MCPCache();

// 缓存用户列表 5 分钟
const users = await cache.get(
  'users:list',
  () => mcp__plugin_api__list_users(),
  300
);
```

---

#### 缓存什么数据？

| 数据类型 | 建议 TTL | 理由 |
|----------|----------|------|
| 用户列表 | 5 分钟 | 变化不频繁 |
| 项目配置 | 10 分钟 | 相对稳定 |
| 任务状态 | 1 分钟 | 可能频繁变化 |
| 统计数据 | 30 分钟 | 计算成本高 |
| 元数据 | 1 小时 | 几乎不变 |

---

### 优化 3: 并发控制

#### 并行调用无依赖的操作

```javascript
// ❌ 错误：串行调用
async function initProject(projectId) {
  const project = await get_project(projectId);     // 100ms
  const users = await list_users();                 // 100ms
  const tasks = await list_tasks(projectId);        // 100ms
  
  return { project, users, tasks };
  // 总耗时：300ms
}

// ✅ 正确：并行调用
async function initProject(projectId) {
  const [project, users, tasks] = await Promise.all([
    get_project(projectId),
    list_users(),
    list_tasks(projectId)
  ]);
  
  return { project, users, tasks };
  // 总耗时：100ms！
}
```

---

#### 限制并发数

```javascript
// 实现并发限制
async function batchUpdate(taskIds, updateFn, maxConcurrency = 5) {
  const results = [];
  
  for (let i = 0; i < taskIds.length; i += maxConcurrency) {
    const batch = taskIds.slice(i, i + maxConcurrency);
    
    // 处理当前批次
    const batchResults = await Promise.all(
      batch.map(id => updateFn(id).catch(err => ({ id, error: err.message })))
    );
    
    results.push(...batchResults);
  }
  
  return results;
}

// 使用：每次最多 5 个并发
await batchUpdate(taskIds, 
  id => mcp__plugin_api__update_task(id, { completed: true }),
  5
);
```

---

### 优化 4: 限流保护

#### 实现令牌桶算法

```javascript
class RateLimiter {
  constructor(tokensPerSecond, maxTokens) {
    this.tokensPerSecond = tokensPerSecond;
    this.maxTokens = maxTokens;
    this.tokens = maxTokens;
    this.lastRefill = Date.now();
  }
  
  async acquire(tokens = 1) {
    // 补充令牌
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(
      this.maxTokens,
      this.tokens + elapsed * this.tokensPerSecond
    );
    this.lastRefill = now;
    
    // 如果令牌不足，等待
    if (this.tokens < tokens) {
      const waitTime = (tokens - this.tokens) / this.tokensPerSecond * 1000;
      await sleep(waitTime);
      return this.acquire(tokens);
    }
    
    // 消耗令牌
    this.tokens -= tokens;
    return true;
  }
}

// 使用：每秒最多 10 次请求
const limiter = new RateLimiter(10, 20);

async function callAPI() {
  await limiter.acquire();
  return mcp__plugin_api__call();
}
```

---

## 🏭 生产环境部署

### 部署清单

#### 部署前检查

```markdown
## 安全检查清单

### 凭证管理
- [ ] 所有敏感信息使用环境变量
- [ ] .env 文件已添加到 .gitignore
- [ ] 生产环境使用独立凭证
- [ ] 凭证轮换计划已制定

### 权限配置
- [ ] MCP 服务器使用最小权限
- [ ] OAuth scopes 已最小化
- [ ] 文件访问范围已限制
- [ ] API Token 权限已审查

### 代码质量
- [ ] 所有输入都已验证
- [ ] 错误处理完善
- [ ] 日志不包含敏感信息
- [ ] 超时和重试已配置

### 性能优化
- [ ] 实现了缓存策略
- [ ] 批量查询已优化
- [ ] 并发控制已添加
- [ ] 限流保护已实现

### 监控告警
- [ ] 关键指标已监控
- [ ] 错误率告警已配置
- [ ] 性能指标已追踪
- [ ] 日志聚合已设置
```

---

### 监控指标

#### 关键指标

| 指标 | 正常值 | 告警阈值 | 说明 |
|------|--------|----------|------|
| 响应时间 | <200ms | >1000ms | P95 延迟 |
| 错误率 | <1% | >5% | 失败请求占比 |
| 成功率 | >99% | <95% | 成功请求占比 |
| 并发数 | <100 | >500 | 同时连接数 |
| CPU 使用 | <50% | >80% | 服务器负载 |
| 内存使用 | <500MB | >1GB | 内存占用 |

---

#### 监控实现

```javascript
// 简单的监控埋点
class Monitor {
  constructor() {
    this.metrics = {
      requests: 0,
      errors: 0,
      latency: []
    };
  }
  
  async track(name, fn) {
    const start = Date.now();
    this.metrics.requests++;
    
    try {
      const result = await fn();
      const latency = Date.now() - start;
      this.metrics.latency.push(latency);
      
      // 保持最近 1000 个数据点
      if (this.metrics.latency.length > 1000) {
        this.metrics.latency.shift();
      }
      
      return result;
    } catch (error) {
      this.metrics.errors++;
      logger.error(`Error in ${name}`, error);
      throw;
    }
  }
  
  getStats() {
    const latencies = this.metrics.latency.sort((a, b) => a - b);
    const p95 = latencies[Math.floor(latencies.length * 0.95)];
    
    return {
      totalRequests: this.metrics.requests,
      errorRate: (this.metrics.errors / this.metrics.requests * 100).toFixed(2) + '%',
      p95Latency: p95 + 'ms',
      avgLatency: (latencies.reduce((a, b) => a + b, 0) / latencies.length).toFixed(0) + 'ms'
    };
  }
}

// 使用
const monitor = new Monitor();

await monitor.track('create_task', async () => {
  return mcp__plugin_api__create_task(params);
});

// 定期输出统计
setInterval(() => {
  console.log('MCP Stats:', monitor.getStats());
}, 60000); // 每分钟
```

---

### 日志规范

#### 结构化日志格式

```javascript
// ✅ 好的日志
logger.info({
  event: 'mcp_tool_call',
  tool: 'create_task',
  duration: 234,
  status: 'success',
  requestId: 'req-123'
});

// ❌ 差的日志
logger.info('Called create_task, took 234ms, success');
```

#### 日志级别使用

```javascript
// ERROR - 需要立即处理的错误
logger.error({
  event: 'mcp_connection_failed',
  server: 'asana',
  error: error.message,
  stack: error.stack
});

// WARN - 警告但不影响功能
logger.warn({
  event: 'token_expiring_soon',
  expiresAt: tokenExpiry,
  daysRemaining: 3
});

// INFO - 一般信息
logger.info({
  event: 'mcp_server_connected',
  server: 'asana',
  toolsCount: 5
});

// DEBUG - 调试信息
logger.debug({
  event: 'tool_input',
  tool: 'create_task',
  input: { name: 'Task 1' }
});
```

---

## 📊 性能基准测试

### 测试方法

```javascript
class PerformanceTest {
  constructor() {
    this.results = [];
  }
  
  async runTest(name, testFn, iterations = 100) {
    const times = [];
    
    for (let i = 0; i < iterations; i++) {
      const start = Date.now();
      await testFn();
      const duration = Date.now() - start;
      times.push(duration);
    }
    
    const sorted = times.sort((a, b) => a - b);
    const stats = {
      name,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      avg: times.reduce((a, b) => a + b, 0) / iterations,
      p50: sorted[Math.floor(iterations * 0.5)],
      p95: sorted[Math.floor(iterations * 0.95)],
      p99: sorted[Math.floor(iterations * 0.99)]
    };
    
    this.results.push(stats);
    return stats;
  }
}

// 测试示例
const tester = new PerformanceTest();

await tester.runTest('Single Task Creation', async () => {
  await mcp__plugin_api__create_task({ name: 'Test' });
});

await tester.runTest('Batch Task Creation (10)', async () => {
  await Promise.all(
    Array(10).fill().map((_, i) => 
      mcp__plugin_api__create_task({ name: `Test ${i}` })
    )
  );
});

// 输出结果
console.table(tester.results);
```

---

### 性能目标

| 操作 | P50 | P95 | P99 |
|------|-----|-----|-----|
| 单次工具调用 | <100ms | <300ms | <500ms |
| 批量查询（10 个） | <200ms | <500ms | <800ms |
| 复杂操作（创建 + 分配） | <300ms | <700ms | <1000ms |
| 认证流程 | <500ms | <1000ms | <1500ms |

---

## 📝 学习检查清单

完成本节后，你应该能够：

- [ ] 解释最小权限原则并应用
- [ ] 说出 3 种凭证存储方式
- [ ] 实施输入验证防止注入攻击
- [ ] 实现批量查询优化性能
- [ ] 设计智能缓存策略
- [ ] 配置并发控制和限流保护
- [ ] 列出生产环境部署检查项
- [ ] 实现基本的监控指标

---

## 💡 常见问题

### Q1: 如何平衡安全性和便利性？

**答：** 
- 开发环境可以适当放宽
- 生产环境必须严格遵守
- 使用自动化工具减少手动操作
- 实施审计日志追踪异常

### Q2: 缓存会导致数据不一致吗？

**答：** 
有可能。解决方案：
- 设置合适的 TTL
- 写操作时使相关缓存失效
- 对实时性要求高的数据不缓存

### Q3: 限流会影响用户体验吗？

**答：** 
合理的限流不会：
- 设置足够高的限制（如每秒 10 次）
- 对用户友好的提示
- 实现排队而非直接拒绝

### Q4: 如何选择监控指标？

**答：** 
关注直接影响用户的指标：
- 响应时间（用户体验）
- 错误率（系统稳定性）
- 成功率（功能可用性）

---

## 🎯 实战练习

### 练习 1: 安全审计

审查以下配置的安全性：

```json
{
  "database": {
    "command": "node",
    "args": ["server.js"],
    "env": {
      "DB_PASSWORD": "admin123",
      "API_KEY": "sk-prod-key-12345"
    }
  },
  "filesystem": {
    "command": "npx",
    "args": ["@modelcontextprotocol/server-filesystem", "/"]
  }
}
```

**发现的安全问题：**
1. ❌ 硬编码密码
2. ❌ 硬编码 API Key
3. ❌ 文件系统访问范围过大（根目录）

**改进建议：**
```json
{
  "database": {
    "command": "node",
    "args": ["server.js"],
    "env": {
      "DB_PASSWORD": "${DB_PASSWORD}",
      "API_KEY": "${API_KEY}"
    }
  },
  "filesystem": {
    "command": "npx",
    "args": ["@modelcontextprotocol/server-filesystem", "/projects/myapp"]
  }
}
```

### 练习 2: 性能优化设计

为以下场景设计优化方案：

场景：一个插件需要查询 1000 个任务的状态，并为每个任务获取负责人信息。

**参考答案：**
```javascript
// 优化方案
async function getTasksWithUsers() {
  // 1. 批量查询任务
  const tasks = await search_tasks({ limit: 1000 });
  
  // 2. 收集唯一的用户 ID
  const userIds = [...new Set(tasks.map(t => t.assignee_id))];
  
  // 3. 批量查询用户（带缓存）
  const users = await cache.get(
    'users:batch',
    () => get_users_by_ids(userIds),
    300 // 5 分钟 TTL
  );
  
  // 4. 构建映射
  const userMap = new Map(users.map(u => [u.id, u]));
  
  // 5. 合并结果
  return tasks.map(task => ({
    ...task,
    assignee: userMap.get(task.assignee_id)
  }));
}

// 性能对比
// 优化前：1000 次任务查询 + 1000 次用户查询 = 2000 次 API 调用
// 优化后：1 次任务查询 + 1 次用户查询 = 2 次 API 调用
// 提升：1000 倍！
```

---

## 🔗 相关资源

### 参考文档
- [SKILL.md](../../plugins/plugin-dev/skills/mcp-integration/SKILL.md) - MCP 集成完整技能指南
- [authentication.md](../../plugins/plugin-dev/skills/mcp-integration/references/authentication.md) - 认证最佳实践
- [server-types.md](../../plugins/plugin-dev/skills/mcp-integration/references/server-types.md) - 服务器类型详解

### 安全资源
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Web 应用安全风险
- [CWE/SANS Top 25](https://cwe.mitre.org/top25/) - 最危险软件缺陷

### 性能资源
- [12 Factor App](https://12factor.net/) - 现代应用最佳实践
- [Google SRE Book](https://sre.google/books/) - 站点可靠性工程

---

## 🎓 MCP 集成完整学习总结

恭喜你完成了 MCP 集成的全部学习！现在你已经掌握了：

### 知识体系

1. **基础概念** - MCP 是什么、为什么重要
2. **服务器类型** - stdio/SSE/HTTP/WebSocket 四种类型
3. **认证鉴权** - OAuth/Token/环境变量三种方式
4. **工具使用** - Commands/Agents 中的实际应用
5. **调试排错** - 日志分析、问题诊断
6. **最佳实践** - 安全规范、性能优化

### 实战能力

- ✅ 配置各种类型的 MCP 服务器
- ✅ 实施安全的认证机制
- ✅ 在 Commands 和 Agents 中使用 MCP 工具
- ✅ 排查常见连接和认证问题
- ✅ 优化性能和实施监控

### 下一步建议

1. **动手实践** - 为你的项目配置第一个 MCP 服务器
2. **深入学习** - 阅读官方文档和参考资源
3. **分享交流** - 在社区分享你的经验
4. **持续改进** - 关注 MCP 生态发展

---

**恭喜你完成了 MCP 集成模块的全部学习！** 🎉

你已经从零开始，系统学习了 MCP 的所有核心知识点。现在可以将这些知识应用到实际项目中，构建强大的集成解决方案！

记住：理论很重要，但实践出真知。立即开始你的第一个 MCP 集成项目吧！
