# MCP 认证与鉴权 - OAuth/Token/环境变量管理

## 📋 本节学习目标

学完本节后，你将能够：
- ✅ 理解 MCP 的三种认证方式
- ✅ 配置 OAuth 自动认证流程
- ✅ 安全管理 Token 和环境变量
- ✅ 排查常见的认证问题

---

## 🎯 认证方式总览

### 为什么需要认证？

MCP 服务器通常访问敏感资源：
- 数据库（包含用户数据）
- API（可能产生费用）
- 文件系统（私密代码和配置）
- 云服务（项目管理和代码仓库）

**认证的目的：** 确保只有授权的用户和服务可以访问这些资源。

---

### 三种认证方式对比

| 认证方式 | 适用场景 | 安全性 | 复杂度 |
|----------|----------|--------|--------|
| **OAuth 2.0** | SSE/HTTP云服务 | ⭐⭐⭐⭐⭐ | 简单（自动） |
| **Token 认证** | HTTP/WebSocket | ⭐⭐⭐⭐ | 中等 |
| **环境变量** | stdio 本地服务 | ⭐⭐⭐ | 简单 |

---

## 1️⃣ OAuth 2.0 认证 - 最安全的云服务认证

### 什么是 OAuth 2.0？

**OAuth 2.0** 是一种开放标准，允许用户授权第三方应用访问其资源，而无需分享密码。

**生活例子：**
```
你想用微信登录某个网站
    ↓
网站请求微信授权
    ↓
你在微信中点击"同意"
    ↓
网站获得访问权限（但不知道你的微信密码）
```

---

### OAuth 在 MCP 中的工作流程

```mermaid
graph LR
    A[用户首次使用工具] --> B[Claude Code 检测到未认证]
    B --> C[打开浏览器进行 OAuth 授权]
    C --> D[用户登录并授权]
    D --> E[Claude Code 获得 Access Token]
    E --> F[Token 加密存储]
    F --> G[正常使用工具]
    G --> H{Token 快过期？}
    H -->|是 | I[自动刷新 Token]
    H -->|否 | G
```

---

### OAuth 配置示例

#### 最简单的 OAuth 配置（推荐）

```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  }
}
```

**就这么简单！** Claude Code 会自动处理：
- ✅ 检测是否需要认证
- ✅ 打开浏览器进行授权
- ✅ 存储 Access Token
- ✅ 自动刷新 Token
- ✅ Token 加密保护

---

### OAuth 的详细流程

#### Step 1: 触发认证

当用户第一次尝试使用 Asana 工具时：
```
用户：帮我创建一个 Asana 任务
    ↓
Claude: 检测到需要 Asana MCP 认证
```

#### Step 2: 浏览器授权

Claude Code 自动：
1. 生成唯一的 PKCE challenge
2. 构建 OAuth 授权 URL
3. 在默认浏览器打开 URL

```
https://app.asana.com/-/oauth_authorize?
  client_id=xxx&
  redirect_uri=yyy&
  response_type=code&
  scope=read+write&
  code_challenge=zzz
```

#### Step 3: 用户授权

用户在浏览器中：
1. 登录 Asana（如果未登录）
2. 看到授权页面："Claude Code 请求访问你的 Asana 账户"
3. 点击"允许"

#### Step 4: 交换 Token

浏览器重定向回 Claude Code：
```
claude://oauth/callback?code=AUTH_CODE
```

Claude Code 用授权码换取 Token：
```
POST https://app.asana.com/-/oauth_token
{
  grant_type: "authorization_code",
  code: "AUTH_CODE",
  redirect_uri: "claude://oauth/callback"
}

Response:
{
  "access_token": "ACCESS_TOKEN",
  "refresh_token": "REFRESH_TOKEN",
  "expires_in": 3600
}
```

#### Step 5: 安全存储

Claude Code 将 Token 加密存储在：
- macOS: Keychain
- Windows: Credential Manager
- Linux: Secret Service

**重要：** Token 对插件不可见，完全由 Claude Code 管理。

#### Step 6: 自动刷新

Token 快过期时（通常剩余 5 分钟）：
```
POST https://app.asana.com/-/oauth_token
{
  grant_type: "refresh_token",
  refresh_token: "REFRESH_TOKEN"
}

Response:
{
  "access_token": "NEW_ACCESS_TOKEN",
  "refresh_token": "NEW_REFRESH_TOKEN"
}
```

---

### OAuth Scopes（权限范围）

**什么是 Scopes？**

Scopes 定义了应用可以访问的资源范围。

**常见 Scopes：**
```
read:tasks     - 读取任务
write:tasks    - 创建/更新任务
read:projects  - 读取项目
write:projects - 创建/更新项目
delete:tasks   - 删除任务
```

**在配置中指定 Scopes（可选）：**
```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse",
    "scopes": ["read:tasks", "write:tasks"]
  }
}
```

**最佳实践：**
- ✅ 只请求需要的最小权限
- ✅ 在 README 中说明需要的 scopes
- ✅ 让用户知道为什么需要这些权限

---

### OAuth 故障排查

#### 问题 1: 认证循环

**症状：** 反复要求授权，无法正常化用

**可能原因：**
1. OAuth 回调 URL 配置错误
2. Scopes 不匹配
3. Token 存储失败

**解决方法：**
```bash
# 1. 清除缓存的 Token
# 在系统钥匙串中找到 "Claude Code" 相关条目并删除

# 2. 重新授权
# 再次使用工具，会重新打开浏览器

# 3. 检查 scopes
# 确认配置的 scopes 与服务端一致
```

#### 问题 2: Scope 不足

**症状：** 某些工具调用失败，返回 403 Forbidden

**解决方法：**
1. 查看错误信息，确认缺少的 scope
2. 更新配置添加所需 scope
3. 重新授权

```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse",
    "scopes": [
      "read:tasks",
      "write:tasks",
      "read:projects"  // 新增
    ]
  }
}
```

---

## 2️⃣ Token 认证 - HTTP/WebSocket 的标准方式

### 什么是 Token 认证？

Token 认证是在 HTTP 请求头中携带令牌来证明身份的方式。

**工作原理：**
```
┌──────────────┐
│ Claude Code  │
└──────┬───────┘
       │ Authorization: Bearer YOUR_TOKEN
       ↓
┌──────────────┐
│ MCP 服务器    │ 验证 Token
└──────────────┘
```

---

### Token 配置示例

#### 示例 1: Bearer Token

```json
{
  "api-service": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

**解读：**
- `Authorization`: 标准 HTTP 认证头
- `Bearer`: Token 类型标识
- `${API_TOKEN}`: 从环境变量读取

#### 示例 2: API Key

```json
{
  "internal-api": {
    "type": "http",
    "url": "https://api.internal.com/mcp",
    "headers": {
      "X-API-Key": "${INTERNAL_API_KEY}"
    }
  }
}
```

**解读：**
- `X-API-Key`: 自定义 API Key 头
- 很多内部服务使用这种方式

#### 示例 3: 多 Token 组合

```json
{
  "advanced-api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${ACCESS_TOKEN}",
      "X-Refresh-Token": "${REFRESH_TOKEN}",
      "X-Client-ID": "${CLIENT_ID}"
    }
  }
}
```

---

### Token 的管理方式

#### 方式 1: 用户环境变量（推荐）

**设置方法：**

macOS/Linux:
```bash
# 临时设置（当前终端会话）
export API_TOKEN="your_token_here"

# 永久设置
echo 'export API_TOKEN="your_token_here"' >> ~/.bashrc
source ~/.bashrc
```

Windows PowerShell:
```powershell
# 临时设置
$env:API_TOKEN="your_token_here"

# 永久设置（需要重启终端）
[System.Environment]::SetEnvironmentVariable(
  "API_TOKEN", 
  "your_token_here", 
  "User"
)
```

**验证设置：**
```bash
# macOS/Linux
echo $API_TOKEN

# Windows
echo $env:API_TOKEN
```

---

#### 方式 2: .env 文件（开发环境）

**创建 .env 文件：**
```bash
# 在插件根目录创建 .env 文件
API_TOKEN=your_token_here
DB_URL=postgresql://user:pass@localhost/db
```

**加载 .env 文件：**

在启动脚本中：
```bash
#!/bin/bash
set -a
source .env
set +a
claude
```

**⚠️ 重要：** .env 文件必须添加到 `.gitignore`！

```gitignore
# .gitignore
.env
*.local
```

---

#### 方式 3: 密钥管理服务（企业级）

**使用 AWS Secrets Manager:**
```bash
# 获取 Token
aws secretsmanager get-secret-value \
  --secret-id my-plugin/api-token \
  --query SecretString \
  --output text > API_TOKEN
```

**使用 HashiCorp Vault:**
```bash
# 读取 Token
vault kv get -field=token secret/my-plugin
```

---

### Token 的安全最佳实践

#### ✅ DO（正确做法）

1. **使用环境变量**
```json
✅ "Authorization": "Bearer ${API_TOKEN}"
❌ "Authorization": "Bearer sk-abc123"
```

2. **限制 Token 权限**
```bash
# 创建只读 Token（而不是完全访问）
curl -X POST https://api.example.com/tokens \
  -d '{"scope": "read:only"}'
```

3. **定期轮换 Token**
```bash
# 设置提醒，每 90 天更换一次 Token
# 在日历中添加周期性事件
```

4. **使用 Token 池**
```bash
# 生产环境：多个 Token 负载均衡
API_TOKEN_1=xxx
API_TOKEN_2=yyy
API_TOKEN_3=zzz
```

5. **监控 Token 使用**
```bash
# 定期检查 Token 使用情况
# 发现异常立即撤销
```

#### ❌ DON'T（错误做法）

1. **不要硬编码 Token**
```json
❌ {
  "headers": {
    "Authorization": "Bearer sk-hardcoded-token-123"
  }
}
```

2. **不要提交 Token 到 Git**
```bash
# 检查 .gitignore
.env
*.local
config.local.json
```

3. **不要在日志中显示完整 Token**
```python
# ❌ 错误
print(f"Using token: {token}")

# ✅ 正确
masked = token[:8] + "..." + token[-4:]
print(f"Using token: {masked}")
```

4. **不要共享 Token**
- 每个用户应该有自己的 Token
- 不要通过邮件/聊天发送 Token

5. **不要使用过期的 Token**
- 实现自动刷新逻辑
- 监控 Token 过期时间

---

### Token 刷新策略

#### 策略 1: 定时刷新

```python
import schedule
import time

def refresh_token():
    # 调用 API 刷新 Token
    new_token = api.refresh(old_token)
    update_environment(new_token)

# 每 55 分钟刷新一次（Token 有效期 60 分钟）
schedule.every(55).minutes.do(refresh_token)
```

#### 策略 2: 按需刷新

```python
def call_api():
    if token.is_expired():
        token = refresh_token()
    
    return api.request(token)
```

#### 策略 3: 双 Token 机制

```json
{
  "headers": {
    "Authorization": "Bearer ${ACCESS_TOKEN}",
    "X-Refresh-Token": "${REFRESH_TOKEN}"
  }
}
```

**工作原理：**
- `Access Token`: 短期有效（1 小时）
- `Refresh Token`: 长期有效（30 天）
- Access Token 过期后用 Refresh Token 换取新的 Access Token

---

## 3️⃣ 环境变量认证 - stdio 服务器的标准方式

### 什么是环境变量认证？

通过环境变量向 stdio MCP 服务器传递认证信息。

**工作原理：**
```
┌──────────────┐
│ Claude Code  │
└──────┬───────┘
       │ spawn 进程
       │ env: { DB_PASSWORD: "xxx" }
       ↓
┌──────────────┐
│ MCP 服务器    │ 读取环境变量
│ (本地进程)   │
└──────────────┘
```

---

### 环境变量配置示例

#### 示例 1: 数据库连接

```json
{
  "database": {
    "command": "npx",
    "args": ["-y", "mcp-server-postgres"],
    "env": {
      "DATABASE_URL": "${DB_URL}",
      "PGPASSWORD": "${DB_PASSWORD}",
      "PGUSER": "${DB_USER}",
      "PGDATABASE": "${DB_NAME}"
    }
  }
}
```

#### 示例 2: API 密钥

```json
{
  "custom-api": {
    "command": "python",
    "args": ["-m", "my_mcp_server"],
    "env": {
      "API_KEY": "${MY_API_KEY}",
      "API_SECRET": "${MY_API_SECRET}",
      "LOG_LEVEL": "info"
    }
  }
}
```

#### 示例 3: 混合配置

```json
{
  "multi-service": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/servers/multi.js"],
    "env": {
      # 数据库配置
      "DB_HOST": "${DB_HOST}",
      "DB_PORT": "${DB_PORT}",
      
      # API 配置
      "STRIPE_KEY": "${STRIPE_SECRET_KEY}",
      "SENDGRID_KEY": "${SENDGRID_API_KEY}",
      
      # 通用配置
      "NODE_ENV": "production",
      "LOG_LEVEL": "warn"
    }
  }
}
```

---

### 环境变量的设置方法

#### 方法 1: Shell 配置文件

**~/.bashrc 或 ~/.zshrc:**
```bash
# 数据库配置
export DB_HOST="localhost"
export DB_PORT="5432"
export DB_USER="myuser"
export DB_PASSWORD="mypassword"
export DB_NAME="mydb"

# API 配置
export STRIPE_SECRET_KEY="sk_xxx"
export SENDGRID_API_KEY="SG.xxx"
```

**应用更改:**
```bash
source ~/.bashrc  # 或 source ~/.zshrc
```

#### 方法 2: 使用 direnv（推荐）

**安装 direnv:**
```bash
# macOS
brew install direnv

# Linux
curl -sfL https://direnv.net/install.sh | bash
```

**配置 .envrc:**
```bash
# 在项目根目录创建 .envrc
export DB_HOST="localhost"
export DB_PASSWORD="secret"
```

**允许加载:**
```bash
direnv allow
```

**优点:**
- ✅ 每个项目独立的环境变量
- ✅ 自动加载和卸载
- ✅ 不会污染全局环境

#### 方法 3: 使用 dotenv-cli

**安装:**
```bash
npm install -g dotenv-cli
```

**创建 .env 文件:**
```bash
DB_HOST=localhost
DB_PASSWORD=secret
```

**使用:**
```bash
dotenv claude
```

---

### 环境变量命名规范

#### 推荐命名约定

```bash
# 数据库相关
DB_HOST          # 数据库主机
DB_PORT          # 数据库端口
DB_USER          # 数据库用户名
DB_PASSWORD      # 数据库密码
DB_NAME          # 数据库名称
DB_URL           # 完整连接 URL

# API 相关
API_KEY          # API 密钥
API_SECRET       # API 机密
API_TOKEN        # API 令牌
API_ENDPOINT     # API 端点

# 服务特定
STRIPE_SECRET_KEY    # Stripe 密钥
AWS_ACCESS_KEY_ID    # AWS 访问密钥
AWS_SECRET_ACCESS_KEY # AWS 密钥
SENDGRID_API_KEY     # SendGrid 密钥
```

#### 命名最佳实践

1. **使用大写字母**
```bash
✅ DB_PASSWORD
❌ db_password
❌ DbPassword
```

2. **使用下划线分隔**
```bash
✅ DATABASE_URL
❌ DATABASEURL
❌ DatabaseUrl
```

3. **有意义的命名**
```bash
✅ API_KEY
❌ KEY123
```

---

## 🔐 安全最佳实践总结

### 1. 最小权限原则

**只授予必要的权限：**
```json
// ❌ 给予所有权限
{
  "scopes": ["*"]
}

// ✅ 只给予需要的权限
{
  "scopes": [
    "read:tasks",
    "write:tasks"
  ]
}
```

### 2. 凭证隔离

**不同环境使用不同凭证：**
```bash
# 开发环境
DEV_DB_PASSWORD="dev_secret"

# 生产环境
PROD_DB_PASSWORD="prod_super_secret"
```

### 3. 定期轮换

**设置提醒定期更换：**
```bash
# 在日历中添加周期性事件
# 每 90 天：更换所有 API Token
# 每 30 天：更换数据库密码
```

### 4. 审计日志

**记录凭证使用情况：**
```bash
# 启用 API 使用日志
# 定期检查异常使用模式
```

### 5. 紧急撤销

**准备应急预案：**
```markdown
## 凭证泄露时的应急步骤

1. 立即撤销泄露的 Token
2. 生成新的 Token
3. 更新所有使用该 Token 的配置
4. 调查泄露原因
5. 加强安全措施
```

---

## 📝 学习检查清单

完成本节后，你应该能够：

- [ ] 解释 OAuth 2.0 的工作原理
- [ ] 配置 OAuth 认证的 MCP 服务器
- [ ] 说出 Token 认证的三种管理方式
- [ ] 正确设置环境变量
- [ ] 列举 5 个 Token 安全最佳实践
- [ ] 排查常见的认证问题

---

## 💡 常见问题

### Q1: OAuth 和 Token 哪个更安全？

**答：** OAuth 更安全，因为：
- Token 由 Claude Code 加密存储
- 自动刷新，无需手动管理
- 可以随时撤销授权
- 支持细粒度的权限控制

### Q2: 环境变量会被其他程序读取吗？

**答：** 有可能。所以：
- 不要用明文存储敏感环境变量
- 使用操作系统的密钥管理服务
- 限制环境变量的可见性

### Q3: Token 过期了怎么办？

**答：** 
- OAuth: 自动刷新，无需担心
- Token: 实现刷新逻辑或手动更新

### Q4: 可以在多个设备上共用一个 Token 吗？

**答：** 技术上可以，但不推荐。最好：
- 每个设备使用独立的 Token
- 便于追踪和撤销

---

## 🎯 实战练习

### 练习 1: 识别认证方式

以下场景应该使用哪种认证方式？

1. 连接本地的 SQLite 数据库
2. 使用 Asana 官方 MCP 服务
3. 调用公司内部的 REST API
4. 连接自己部署的 WebSocket 服务

**答案：**
1. 环境变量（stdio 本地服务）
2. OAuth（云服务 + OAuth 支持）
3. Token（REST API）
4. Token（WebSocket）

### 练习 2: 配置纠错

找出以下配置的错误：

```json
{
  "api": {
    "type": "http",
    "url": "http://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer sk-123456789"
    }
  }
}
```

**答案：**
1. ❌ 使用 HTTP 而不是 HTTPS
2. ❌ 硬编码 Token

**正确配置：**
```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

---

## 🔗 相关资源

### 参考文档
- [authentication.md](../../plugins/plugin-dev/skills/mcp-integration/references/authentication.md) - 完整认证指南

### 下一篇预告
下一节我们将学习 **MCP 工具使用实战**，包括：
- 在 Commands 中使用 MCP 工具
- 在 Agents 中使用 MCP 工具
- 工具调用模式和最佳实践
- 错误处理和重试策略

---

**恭喜你完成了 MCP 认证与鉴权的学习！** 🎉

接下来，让我们学习 [MCP 工具使用实战](./04-MCP 工具使用实战.md)。
