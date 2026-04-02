# MCP 调试与故障排除 - 常见问题解决指南

## 📋 本节学习目标

学完本节后，你将能够：
- ✅ 启用和解读调试日志
- ✅ 诊断连接问题
- ✅ 排查认证失败
- ✅ 解决工具调用错误
- ✅ 优化性能和响应速度

---

## 🛠️ 调试工具总览

### 核心调试命令

```bash
# 1. 查看 MCP 服务器状态
/mcp

# 2. 启用调试模式启动
claude --debug

# 3. 查看详细日志
tail -f ~/.claude/debug.log
```

---

## 1️⃣ 启用调试日志

### 方法 1: 命令行参数

```bash
claude --debug
```

**输出内容：**
- MCP 服务器连接过程
- 工具调用详情
- API 请求和响应
- 认证流程
- 错误堆栈信息

### 方法 2: 环境变量

```bash
# 设置详细日志级别
export CLAUDE_LOG_LEVEL=debug
claude
```

**日志级别：**
- `error` - 仅错误信息
- `warn` - 警告和错误
- `info` - 一般信息
- `debug` - 详细调试信息（推荐）
- `trace` - 最详细的追踪信息

---

### 解读调试日志

#### 日志格式

```
[时间戳] [级别] [模块] 消息内容
```

**实际例子：**
```
[2026-04-02 10:15:30.123] [INFO] [MCP] Connecting to asana server...
[2026-04-02 10:15:30.456] [DEBUG] [MCP] SSE URL: https://mcp.asana.com/sse
[2026-04-02 10:15:31.789] [INFO] [MCP] Connected successfully
[2026-04-02 10:15:31.790] [DEBUG] [MCP] Discovered tools: create_task, search_tasks
```

---

#### 关键日志段分析

##### 连接阶段日志

```log
# 开始连接
[MCP] Initializing MCP server: asana
[MCP] Server type: sse
[MCP] URL: https://mcp.asana.com/sse

# 建立连接
[MCP] Establishing SSE connection...
[MCP] HTTP status: 200 OK
[MCP] SSE stream opened

# 工具发现
[MCP] Discovering available tools...
[MCP] Found 5 tools:
  - mcp__plugin_asana_asana__create_task
  - mcp__plugin_asana_asana__search_tasks
  ...
[MCP] MCP server ready
```

**正常连接的特征：**
- ✅ "SSE stream opened"
- ✅ "Found X tools"
- ✅ "MCP server ready"

**异常连接的特征：**
- ❌ "Connection refused"
- ❌ "Timeout exceeded"
- ❌ "Failed to discover tools"

---

##### 工具调用日志

```log
# 调用开始
[TOOL] Invoking tool: mcp__plugin_asana_asana__create_task
[TOOL] Input parameters:
  name: "完成项目报告"
  workspace: "12345"

# API 请求
[HTTP] POST https://app.asana.com/api/1.0/tasks
[HTTP] Headers:
  Authorization: Bearer ***
  Content-Type: application/json
[HTTP] Request body:
  {"data": {"name": "完成项目报告", ...}}

# API 响应
[HTTP] Response status: 201 Created
[HTTP] Response time: 234ms
[HTTP] Response body:
  {"data": {"gid": "67890", ...}}

# 调用完成
[TOOL] Tool execution completed successfully
[TOOL] Output:
  id: "67890"
  url: "https://app.asana.com/0/67890"
```

**有用的调试信息：**
- 请求参数（验证输入）
- 响应时间（性能分析）
- HTTP 状态码（错误诊断）
- 响应体（结果验证）

---

## 2️⃣ 连接问题排查

### 问题 1: stdio 服务器无法启动

#### 症状

```bash
/mcp
# 显示服务器状态为 "disconnected" 或根本不显示
```

#### 诊断步骤

**Step 1: 检查配置**
```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
  }
}
```

**检查清单：**
- [ ] `command` 字段存在且正确
- [ ] `args` 数组完整
- [ ] 路径使用 `${CLAUDE_PLUGIN_ROOT}` 或绝对路径
- [ ] JSON 语法正确

**Step 2: 手动测试命令**

在终端中直接运行：
```bash
npx -y @modelcontextprotocol/server-filesystem /tmp/test
```

**预期结果：**
- 应该看到 JSON-RPC 的欢迎消息
- 不应该有错误输出

**可能的错误：**
```bash
# 错误 1: 命令不存在
bash: npx: command not found
→ 解决：安装 Node.js

# 错误 2: 权限不足
bash: ./server.sh: Permission denied
→ 解决：chmod +x server.sh

# 错误 3: Python 模块未找到
/usr/bin/python: No module named my_mcp_server
→ 解决：pip install my_mcp_server
```

**Step 3: 检查文件权限**

```bash
# macOS/Linux
ls -la servers/my-server.js
# 应该是可执行的：-rwxr-xr-x

# 如果不是，添加执行权限
chmod +x servers/my-server.js
```

**Step 4: 查看详细错误**

```bash
claude --debug 2>&1 | grep -A 10 "filesystem"
```

**常见错误模式：**
```log
# 进程启动失败
[ERROR] Failed to spawn process: ENOENT
→ 原因：command 指定的程序不存在

# 立即退出
[ERROR] Process exited with code 1
→ 原因：脚本执行出错

# 无响应
[WARN] Process not responding
→ 原因：进程卡死或崩溃
```

---

### 问题 2: SSE 服务器连接失败

#### 症状

```log
[ERROR] Failed to connect to SSE server
[ERROR] Connection timeout after 30s
```

#### 诊断步骤

**Step 1: 测试 URL 可达性**

```bash
curl -I https://mcp.asana.com/sse
```

**预期响应：**
```http
HTTP/2 200 
content-type: text/event-stream
```

**可能的错误：**
```http
# 404 Not Found
→ URL 错误或资源不存在

# 401 Unauthorized
→ 需要认证

# 403 Forbidden
→ IP 被阻止或权限不足

# 502 Bad Gateway
→ 服务端问题
```

**Step 2: 检查网络连接**

```bash
# ping 测试
ping mcp.asana.com

# DNS 解析
nslookup mcp.asana.com

# 端口连通性
telnet mcp.asana.com 443
```

**Step 3: 检查防火墙**

```bash
# macOS
sudo pfctl -s rules

# Windows
Get-NetFirewallRule | Where-Object {$_.Enabled -eq True}

# Linux
sudo iptables -L -n
```

**Step 4: 检查代理配置**

如果使用代理：
```bash
export https_proxy=http://proxy.company.com:8080
export no_proxy="localhost,127.0.0.1"
```

---

### 问题 3: HTTP 服务器认证失败

#### 症状

```log
[ERROR] HTTP 401 Unauthorized
[ERROR] Invalid or expired token
```

#### 诊断步骤

**Step 1: 验证 Token 有效性**

```bash
curl -H "Authorization: Bearer $API_TOKEN" \
     https://api.example.com/health
```

**预期响应：**
```json
{"status": "ok"}
```

**错误响应：**
```json
{"error": "invalid_token"}
→ Token 无效或过期

{"error": "insufficient_scope"}
→ Token 权限不足
```

**Step 2: 检查环境变量**

```bash
# 验证环境变量已设置
echo $API_TOKEN

# 如果不为空，说明已设置
# 如果为空，说明未设置
```

**Step 3: 检查 Token 格式**

```json
// ✅ 正确
"Authorization": "Bearer ${API_TOKEN}"

// ❌ 错误 1: 缺少 Bearer 前缀
"Authorization": "${API_TOKEN}"

// ❌ 错误 2: 拼写错误
"Authorization": "Bearr ${API_TOKEN}"

// ❌ 错误 3: 多余空格
"Authorization": "Bearer  ${API_TOKEN}"
```

---

## 3️⃣ 工具调用错误排查

### 错误类型 1: 工具不存在

#### 症状

```log
[ERROR] Tool not found: mcp__plugin_asana_asana__create_task
```

#### 可能原因

1. **MCP 服务器未连接**
   ```bash
   /mcp
   # 检查服务器状态
   ```

2. **工具名拼写错误**
   ```bash
   # 错误：create_tas (少 k)
   # 正确：create_task
   ```

3. **插件名或服务器名错误**
   ```bash
   # 错误：mcp__plugin_asana_other__create_task
   # 正确：mcp__plugin_asana_asana__create_task
   ```

#### 解决方案

**Step 1: 确认可用工具**
```bash
/mcp
```

**Step 2: 复制准确的工具名**
从输出中直接复制完整的工具名

**Step 3: 重启 Claude Code**
```bash
# 完全退出并重新启动
```

---

### 错误类型 2: 参数验证失败

#### 症状

```log
[ERROR] Tool call failed: Validation error
Missing required parameter: workspace
```

#### 诊断步骤

**Step 1: 查看工具 Schema**
```bash
/mcp
# 找到对应工具，查看 inputSchema
```

**Step 2: 检查必需参数**

```json
{
  "inputSchema": {
    "required": ["name", "workspace"],  // ← 这些是必需的
    "properties": {
      "name": {"type": "string"},
      "workspace": {"type": "string"},
      "notes": {"type": "string"}  // ← 这是可选的
    }
  }
}
```

**Step 3: 验证参数类型**

```typescript
// ✅ 正确
{
  "name": "任务标题",        // string
  "workspace": "12345",      // string
  "completed": false         // boolean
}

// ❌ 错误
{
  "name": 123,              // 应该是 string
  "workspace": true,        // 应该是 string
  "completed": "false"      // 应该是 boolean
}
```

---

### 错误类型 3: API 调用失败

#### 症状

```log
[ERROR] API returned 500 Internal Server Error
[ERROR] Rate limit exceeded (429)
```

#### 诊断步骤

**Step 1: 识别错误类型**

```log
# 客户端错误 (4xx)
400 Bad Request      → 参数错误
401 Unauthorized     → 认证失败
403 Forbidden        → 权限不足
404 Not Found        → 资源不存在
429 Too Many Requests → 触发限流

# 服务端错误 (5xx)
500 Internal Error   → 服务器 bug
502 Bad Gateway      → 网关错误
503 Unavailable      → 服务不可用
```

**Step 2: 查看错误详情**

```bash
# 在调试模式下查看详细错误
claude --debug

# 查找类似这样的输出：
[HTTP] Response: 400 Bad Request
[HTTP] Body: {
  "error": {
    "message": "Invalid workspace ID",
    "type": "invalid_request_error"
  }
}
```

**Step 3: 根据错误类型处理**

```markdown
## 处理策略

### 400 Bad Request
1. 检查参数格式
2. 验证参数值范围
3. 确认必填字段

### 401 Unauthorized
1. 检查 Token 是否过期
2. 重新认证
3. 更新环境变量

### 403 Forbidden
1. 确认用户权限
2. 申请必要权限
3. 检查资源所有权

### 429 Rate Limit
1. 读取 Retry-After 头
2. 等待指定时间
3. 实施指数退避
4. 考虑降低调用频率

### 5xx Server Error
1. 等待后重试（2-3 次）
2. 检查服务状态页面
3. 联系服务提供商
```

---

## 4️⃣ 认证问题排查

### 问题 1: OAuth 循环认证

#### 症状

- 反复打开浏览器要求授权
- 授权后仍然提示未认证
- 无法正常化用工具

#### 诊断步骤

**Step 1: 清除缓存的 Token**

macOS:
```bash
# 打开钥匙串访问
open -a "Keychain Access"

# 搜索 "Claude Code" 或 "Asana"
# 删除相关条目
```

Windows:
```powershell
# 打开凭据管理器
control keymgr.dll

# 找到 Claude Code 相关凭据
# 删除
```

**Step 2: 检查 OAuth Scopes**

```json
{
  "scopes": [
    "read:tasks",
    "write:tasks"
  ]
}
```

确保 scopes 与服务端要求一致。

**Step 3: 重新授权**

```bash
# 清除后首次使用工具
# 会自动打开浏览器
# 仔细查看授权页面
# 点击"允许"
```

---

### 问题 2: Token 快速过期

#### 症状

- Token 几分钟后就失效
- 需要频繁重新认证
- 影响正常使用

#### 可能原因

1. **Token 本身有效期短**
   - 某些 API 的 Access Token 只有 1 小时有效期

2. **刷新机制未生效**
   - Refresh Token 未正确使用

3. **时钟不同步**
   - 系统时间不准确

#### 解决方案

**方案 1: 实现自动刷新**

对于 HTTP MCP 服务器，在代码中实现：
```javascript
async function getValidToken() {
  if (token.isExpired()) {
    token = await refreshAccessToken();
  }
  return token;
}
```

**方案 2: 使用 OAuth（推荐）**

OAuth 会自动处理 Token 刷新：
```json
{
  "type": "sse",
  "url": "https://mcp.asana.com/sse"
  // Claude Code 自动管理 Token 刷新
}
```

**方案 3: 同步系统时间**

macOS:
```bash
# 自动同步时间
sudo sntp -sS time.apple.com
```

Windows:
```powershell
# 同步时间
w32tm /resync
```

---

## 5️⃣ 性能问题排查

### 问题 1: 工具调用响应慢

#### 症状

- 每次调用需要数秒甚至数十秒
- 用户体验差
- 可能触发超时错误

#### 诊断步骤

**Step 1: 测量响应时间**

在调试日志中查找：
```log
[HTTP] Request started
[HTTP] Response received: 2345ms  ← 响应时间
```

**Step 2: 识别瓶颈**

```markdown
可能的瓶颈位置:

1. 网络延迟
   - ping 测试延迟高
   - 解决：使用 CDN 或更近的服务器

2. 服务器处理慢
   - 服务端日志显示处理时间长
   - 解决：优化服务端逻辑

3. 客户端序列化慢
   - 数据量过大
   - 解决：分页或过滤

4. 并发限制
   - 串行调用过多
   - 解决：并行调用
```

**Step 3: 优化建议**

```markdown
## 网络优化
- 使用本地服务器（stdio）
- 选择地理距离近的服务器
- 使用高质量网络

## 调用优化
- 批量查询代替多次单个查询
- 缓存常用数据
- 并行无依赖的调用

## 数据优化
- 只请求需要的字段
- 使用分页
- 压缩大数据
```

---

### 问题 2: 内存占用高

#### 症状

- Claude Code 占用大量内存
- 系统变慢
- 可能崩溃

#### 诊断步骤

**Step 1: 监控内存使用**

```bash
# macOS
top -pid $(pgrep -f claude)

# Windows
Get-Process claude | Select-Object WorkingSet
```

**Step 2: 识别内存泄漏**

```markdown
常见原因:

1. 大量数据未释放
   - 查询了大量数据存储在内存
   - 解决：分批处理、及时清理

2. 长连接未关闭
   - WebSocket/SSE 连接累积
   - 解决：定期重连

3. 缓存无限增长
   - 缓存数据只增不减
   - 解决：实现 LRU 缓存淘汰
```

---

## 🔧 实用调试技巧

### 技巧 1: 隔离测试

```bash
# 创建最小化测试配置
cat > test.mcp.json << EOF
{
  "test-server": {
    "command": "echo",
    "args": ["hello"]
  }
}
EOF

# 单独测试这个配置
```

### 技巧 2: 日志过滤

```bash
# 只看 MCP 相关日志
claude --debug 2>&1 | grep MCP

# 只看错误日志
claude --debug 2>&1 | grep ERROR

# 看特定服务器的日志
claude --debug 2>&1 | grep asana
```

### 技巧 3: 网络抓包

```bash
# macOS
sudo tcpdump -i en0 -w mcp.pcap port 443

# Windows (需要 Wireshark)
# 开始捕获 → 重现问题 → 停止捕获
# 过滤 MCP 相关流量
```

### 技巧 4: 对比测试

```bash
# 在不同环境下测试
# 1. 本地开发环境
# 2. 生产环境
# 3. 不同操作系统

# 找出环境特定的问题
```

---

## 📊 常见问题速查表

| 问题 | 可能原因 | 快速检查 |
|------|----------|----------|
| 服务器不连接 | 配置错误 | 检查 JSON 语法 |
| 工具找不到 | 名称错误 | `/mcp` 查看准确名称 |
| 认证失败 | Token 过期 | 检查环境变量 |
| 调用超时 | 网络问题 | ping/curl 测试 |
| 限流错误 | 调用太频繁 | 添加延迟/重试 |
| 内存过高 | 数据泄漏 | 监控内存使用 |

---

## 📝 学习检查清单

完成本节后，你应该能够：

- [ ] 启用调试模式并解读日志
- [ ] 诊断 stdio 服务器启动问题
- [ ] 排查 SSE 连接失败
- [ ] 解决 HTTP 认证错误
- [ ] 处理工具调用错误
- [ ] 优化性能和响应速度
- [ ] 使用至少 3 种调试技巧

---

## 💡 常见问题

### Q1: 调试模式会影响性能吗？

**答：** 会轻微影响性能，因为要写入更多日志。建议在开发和排查问题时使用，生产环境可以关闭。

### Q2: 日志文件在哪里？

**答：** 通常在 `~/.claude/debug.log`，具体位置取决于操作系统。

### Q3: 如何保存调试日志？

**答：** 
```bash
claude --debug 2>&1 | tee debug.log
```

### Q4: 生产环境如何调试？

**答：** 
- 使用日志级别控制（info/warn/error）
- 实现结构化日志
- 使用日志聚合工具
- 添加监控告警

---

## 🎯 实战练习

### 练习 1: 日志分析

给出以下日志片段，判断问题所在：

```log
[INFO] Starting MCP server initialization
[DEBUG] Loading configuration from .mcp.json
[DEBUG] Server config: {"command": "npx", "args": ["server"]}
[ERROR] Failed to spawn process: ENOENT
```

**答案：** 
- 错误：ENOENT（文件或目录不存在）
- 原因：`npx` 命令找不到
- 解决：安装 Node.js

### 练习 2: 设计调试流程

为一个复杂的 MCP 集成问题设计调试流程：
- 涉及 3 个 MCP 服务器
- 间歇性失败
- 不确定是哪个环节的问题

**参考答案：**
```markdown
1. 启用详细日志（debug 级别）
2. 重现问题 3-5 次
3. 收集所有日志
4. 按时间线梳理事件
5. 识别失败模式
6. 隔离测试每个服务器
7. 确定根本原因
8. 实施修复
9. 验证修复效果
```

---

## 🔗 相关资源

### 参考文档
- [authentication.md](../../plugins/plugin-dev/skills/mcp-integration/references/authentication.md) - 认证调试
- [server-types.md](../../plugins/plugin-dev/skills/mcp-integration/references/server-types.md) - 服务器类型详解

### 下一篇预告
下一节我们将学习 **MCP 最佳实践与安全**，包括：
- 安全编码规范
- 性能优化技巧
- 生产环境部署
- 监控和告警

---

**恭喜你完成了 MCP 调试与故障排除的学习！** 🎉

接下来，让我们学习 [MCP 最佳实践与安全](./06-MCP 最佳实践与安全.md)。
