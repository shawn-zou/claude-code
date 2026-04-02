# 03-MCP 服务器集成指南

## 🎯 本章导读

MCP（Model Context Protocol）是 Claude Code 插件系统与外部世界连接的桥梁。通过 MCP，你的插件可以访问数据库、调用 API、操作文件系统等。

**你将学到**：
- ✅ MCP 的基本概念和工作原理
- ✅ 4 种 MCP 服务器类型详解
- ✅ .mcp.json 配置文件格式
- ✅ 环境变量管理与安全
- ✅ OAuth 认证流程
- ✅ 实战：集成 3 个常见服务

**前置知识**：
- 了解插件基本结构（参考 01-插件系统详解.md）
- 基本的 JSON 和 HTTP 知识

**预计耗时**：2-2.5 小时

---

## 一、MCP 基础

### 1.1 什么是 MCP？

**MCP（Model Context Protocol）** 是一种标准化的协议，允许 AI 模型（如 Claude）与外部工具和服务进行交互。

**类比理解**：

```
手机 App 生态系统              MCP 生态系统
    ├── iOS 系统                    ├── Claude Code
    ├── App Store                   ├── MCP Servers
    ├── 第三方 App                  ├── 外部服务（GitHub、数据库等）
    └── API/SDK                     └── MCP 协议
```

**核心价值**：
- 🔗 **连接外部服务** - 数据库、API、文件系统
- 🛠️ **扩展能力边界** - 超越 Claude 原生功能
- 🔌 **即插即用** - 标准化接口，快速集成

---

### 1.2 为什么需要 MCP？

#### 场景对比

**没有 MCP 时**：

```markdown
用户：帮我查询 GitHub 上 claude-code 项目的 star 数

Claude：抱歉，我无法访问实时数据。
        我可以告诉你如何查看：
        1. 打开 github.com
        2. 搜索 claude-code
        3. 查看 star 计数
```

**有 MCP 后**：

```markdown
用户：帮我查询 GitHub 上 claude-code 项目的 star 数

Claude：好的，我来查询一下。
        [使用 mcp__github__get_repository_info]
        
        查询结果：
        ⭐ Star 数：15,234
        🍴 Fork 数：1,876
        👁️ Watchers: 234
        
        需要我帮你做什么吗？
```

**对比结果**：
- ✅ MCP 让 Claude 能够**访问实时数据**
- ✅ MCP 让 Claude 能够**执行实际操作**
- ✅ MCP 让 Claude 能够**与外部系统交互**

---

### 1.3 MCP 工作原理

**简化流程图**：

```mermaid
graph LR
    User[用户请求] --> Claude[Claude Code]
    Claude --> MCP_Client[MCP 客户端]
    MCP_Client --> MCP_Server[MCP Server]
    MCP_Server --> External_Service[外部服务]
    
    External_Service -->|返回数据 | MCP_Server
    MCP_Server -->|格式化结果 | MCP_Client
    MCP_Client -->|工具调用结果 | Claude
    Claude -->|响应 | User
```

**详细步骤**：

```
1. 用户提问
   ↓
2. Claude 分析需要什么工具
   ↓
3. 发现需要 MCP 提供的工具
   ↓
4. MCP 客户端调用对应的 MCP Server
   ↓
5. MCP Server 与外部服务通信
   ↓
6. 获取结果并返回给 Claude
   ↓
7. Claude 整理结果回答用户
```

---

## 二、MCP 服务器配置

### 2.1 两种配置方式

#### 方式 1：独立的 .mcp.json（推荐）

**位置**：插件根目录

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json          ← MCP 配置文件
├── servers/
│   └── my-server.js
└── README.md
```

**.mcp.json 示例**：

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"],
    "env": {
      "LOG_LEVEL": "info"
    }
  },
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  }
}
```

**优点**：
- ✅ 配置清晰，易于维护
- ✅ 支持多个服务器
- ✅ 与 plugin.json 分离，职责明确

---

#### 方式 2：内联在 plugin.json

**位置**：`.claude-plugin/plugin.json`

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My awesome plugin",
  
  "mcpServers": {
    "database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--port", "5432"]
    }
  }
}
```

**优点**：
- ✅ 单一配置文件
- ✅ 适合简单插件

**缺点**：
- ❌ 配置复杂时显得臃肿
- ❌ 不利于管理多个服务器

**建议**：优先使用 **.mcp.json** 方式

---

### 2.2 四种服务器类型详解

#### 类型 1：stdio（本地进程）

**适用场景**：
- 本地工具和服务
- 自定义 MCP 服务器
- NPM 包形式的服务器

**配置格式**：

```json
{
  "server-name": {
    "command": "可执行命令",
    "args": ["参数 1", "参数 2"],
    "env": {
      "环境变量名": "值"
    }
  }
}
```

**完整示例**：

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/Projects"],
    "env": {
      "NODE_ENV": "production",
      "LOG_LEVEL": "info"
    }
  },
  "postgres": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
  }
}
```

**实际案例**：

```json
// 使用本地 Node.js 脚本
{
  "custom-api": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/servers/custom-api.js"],
    "env": {
      "PORT": "3000",
      "API_KEY": "${MY_API_KEY}"
    }
  }
}
```

```python
# servers/custom-api.py (Python 示例)
#!/usr/bin/env python3
import sys
import json

def main():
    # 从 stdin 读取 MCP 请求
    request = json.loads(sys.stdin.readline())
    
    # 处理请求
    result = process_request(request)
    
    # 写入 stdout 返回结果
    print(json.dumps(result), flush=True)

if __name__ == "__main__":
    main()
```

---

#### 类型 2：SSE（Server-Sent Events）

**适用场景**：
- 官方托管 MCP 服务
- 需要 OAuth 认证的服务
- 云端 SaaS 应用

**配置格式**：

```json
{
  "server-name": {
    "type": "sse",
    "url": "https://mcp.example.com/sse"
  }
}
```

**完整示例**：

```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  },
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  }
}
```

**OAuth 认证流程**：

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as Claude Code
    participant M as MCP Server
    participant A as 认证服务 (OAuth)
    
    U->>C: 使用 GitHub 功能
    C->>M: 请求 GitHub 数据
    M->>A: 检查认证状态
    A-->>M: 未认证
    M->>U: 弹出 OAuth 授权页面
    U->>A: 登录并授权
    A->>M: 返回 access_token
    M->>C: 提供 GitHub 工具
    C->>M: 调用工具获取数据
    M->>C: 返回数据
    C->>U: 展示结果
```

**用户体验**：

```
1. 用户首次使用 GitHub 相关功能
   ↓
2. Claude 提示："需要授权访问 GitHub"
   ↓
3. 弹出浏览器窗口，跳转到 GitHub 登录
   ↓
4. 用户登录并点击"授权"
   ↓
5. 自动返回，功能立即可用
   ↓
6. 后续使用无需重复授权（token 已保存）
```

---

#### 类型 3：HTTP（REST API）

**适用场景**：
- RESTful API 服务
- Token 认证的 API
- 自定义后端服务

**配置格式**：

```json
{
  "server-name": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}",
      "Content-Type": "application/json"
    }
  }
}
```

**完整示例**：

```json
{
  "notion": {
    "type": "http",
    "url": "https://api.notion.com/v1/mcp",
    "headers": {
      "Authorization": "Bearer ${NOTION_TOKEN}",
      "Notion-Version": "2022-06-28"
    }
  },
  "linear": {
    "type": "http",
    "url": "https://api.linear.app/mcp",
    "headers": {
      "Authorization": "${LINEAR_API_KEY}"
    }
  }
}
```

**自定义后端示例**：

```javascript
// servers/http-mcp-server.js
const express = require('express');
const app = express();

app.post('/mcp', async (req, res) => {
  const { tool, arguments } = req.body;
  
  switch (tool) {
    case 'get_data':
      const data = await fetchData(arguments.id);
      res.json({ result: data });
      break;
      
    case 'create_item':
      const item = await createItem(arguments.name);
      res.json({ result: item });
      break;
      
    default:
      res.status(404).json({ error: 'Tool not found' });
  }
});

app.listen(3000);
```

---

#### 类型 4：WebSocket（实时通信）

**适用场景**：
- 实时数据流
- 推送通知
- 低延迟双向通信

**配置格式**：

```json
{
  "server-name": {
    "type": "ws",
    "url": "wss://realtime.example.com/ws",
    "headers": {
      "Authorization": "Bearer ${TOKEN}"
    }
  }
}
```

**完整示例**：

```json
{
  "slack-realtime": {
    "type": "ws",
    "url": "wss://slack.com/realtime",
    "headers": {
      "Authorization": "Bearer ${SLACK_BOT_TOKEN}"
    }
  },
  "firebase": {
    "type": "ws",
    "url": "wss://your-app.firebaseio.com/.ws",
    "headers": {
      "Authorization": "Bearer ${FIREBASE_TOKEN}"
    }
  }
}
```

---

### 2.3 环境变量管理

#### ${CLAUDE_PLUGIN_ROOT} 特殊变量

**作用**：指向插件根目录的绝对路径

**为什么重要**：
- ✅ **跨平台兼容** - Windows/Mac/Linux 路径格式不同
- ✅ **便携性** - 插件可以移动到任意位置
- ✅ **安全性** - 避免硬编码路径

**示例对比**：

```json
// ❌ 错误：硬编码路径（不可移植）
{
  "command": "/Users/david/plugins/my-plugin/server.js"
}

// ✅ 正确：使用环境变量（可移植）
{
  "command": "node",
  "args": ["${CLAUDE_PLUGIN_ROOT}/servers/server.js"]
}
```

**实际使用**：

```json
{
  "local-server": {
    "command": "node",
    "args": [
      "${CLAUDE_PLUGIN_ROOT}/servers/api-server.js",
      "--config",
      "${CLAUDE_PLUGIN_ROOT}/config/default.json"
    ],
    "env": {
      "PLUGIN_PATH": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```

---

#### 用户环境变量

**作用**：从用户的 shell 环境读取变量

**语法**：`${VARIABLE_NAME}`

**示例**：

```json
{
  "database": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres"],
    "env": {
      "DATABASE_URL": "${DATABASE_URL}",
      "DB_USER": "${DB_USER}",
      "DB_PASSWORD": "${DB_PASSWORD}"
    }
  },
  "api-service": {
    "type": "http",
    "url": "https://api.example.com",
    "headers": {
      "Authorization": "Bearer ${GITHUB_TOKEN}"
    }
  }
}
```

**用户如何设置**：

```bash
# ~/.bashrc 或 ~/.zshrc
export DATABASE_URL="postgresql://user:pass@localhost/mydb"
export DB_USER="myuser"
export DB_PASSWORD="mypassword"
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
```

**最佳实践**：

1. **在 README 中明确列出所有需要的环境变量**

```markdown
## 环境变量要求

使用前请确保设置以下环境变量：

```bash
# .env 文件
DATABASE_URL=postgresql://...
API_KEY=sk-...
SECRET_KEY=your-secret-key
```

2. **提供 .env.example 模板**

```bash
# .env.example
DATABASE_URL=postgresql://user:pass@localhost/dbname
API_KEY=your-api-key-here
SECRET_KEY=generate-a-secure-key
```

3. **在代码中验证环境变量存在**

```javascript
// servers/server.js
const required = ['DATABASE_URL', 'API_KEY'];
for (const env of required) {
  if (!process.env[env]) {
    console.error(`Missing required environment variable: ${env}`);
    process.exit(1);
  }
}
```

---

## 三、MCP 工具使用

### 3.1 工具命名规则

**自动命名格式**：

```
mcp__plugin_<plugin-name>_<server-name>__<tool-name>
     │            │           │              │
     │            │           │              └── MCP 服务器提供的工具名
     │            │           └── 你在配置中定义的服务器名
     │            └── 插件名称
     └── 固定前缀
```

**示例**：

```json
// .mcp.json
{
  "github-tools": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  }
}
```

如果 GitHub MCP 服务器提供了这些工具：
- `get_repository`
- `create_issue`
- `list_pull_requests`

那么在 Claude Code 中的完整工具名是：
- `mcp__plugin_github_github-tools__get_repository`
- `mcp__plugin_github_github-tools__create_issue`
- `mcp__plugin_github_github-tools__list_pull_requests`

---

### 3.2 在命令中使用 MCP 工具

**示例命令**：

```markdown
<!-- commands/check-github-status.md -->
---
description: 检查 GitHub 仓库状态
argument-hint: [repository]
---

检查仓库状态：!`curl -s https://api.github.com/repos/$1`

使用 GitHub MCP 工具获取详细信息：
- mcp__plugin_github_github-tools__get_repository $1
- mcp__plugin_github_github-tools__list_issues $1

生成报告：
## GitHub 仓库报告：$1

[在此填写 MCP 返回的详细信息]
```

**实际效果**：

```
用户：/check-github-status anthropics/claude-code

Claude: 
正在获取仓库信息...
[使用 mcp__plugin_github_github-tools__get_repository]

✅ 获取成功！

## GitHub 仓库报告：anthropics/claude-code

- ⭐ Stars: 15,234
- 🍴 Forks: 1,876
- 📦 Issues: 234 open
- 🔀 Pull Requests: 45 open
```

---

### 3.3 在 Agent 中使用 MCP 工具

**示例 Agent**：

```markdown
<!-- agents/github-assistant.md -->
---
name: github-assistant
description: GitHub 操作助手，帮助管理仓库、Issues 和 Pull Requests
model: inherit
color: blue
tools: ["Read", "Grep"]
---

你是专业的 GitHub 操作助手。

**你可以使用以下 MCP 工具：**
- mcp__plugin_github_github-tools__get_repository - 获取仓库信息
- mcp__plugin_github_github-tools__list_issues - 列出 Issues
- mcp__plugin_github_github-tools__create_issue - 创建 Issue
- mcp__plugin_github_github-tools__list_pull_requests - 列出 PRs

**工作流程：**
1. 询问用户需要做什么操作
2. 使用相应的 MCP 工具
3. 整理结果并展示给用户

**示例对话：**

用户：帮我看看 claude-code 仓库有多少个开放的 issue

你：好的，我来查询一下。
   [使用 mcp__plugin_github_github-tools__list_issues anthropics/claude-code]
   
   查询结果：
   当前有 234 个开放的 issue，其中：
   - 🔴 高优先级：12 个
   - 🟡 中优先级：89 个
   - 🟢 低优先级：133 个
   
   需要我帮你做什么吗？
```

---

## 四、实战案例

### 实战 1：集成 PostgreSQL 数据库

**目标**：创建一个可以查询数据库的插件

**步骤 1：创建插件结构**

```bash
mkdir -p database-plugin/{.claude-plugin,servers}
cd database-plugin
```

**步骤 2：创建 plugin.json**

```json
{
  "name": "database-plugin",
  "version": "1.0.0",
  "description": "PostgreSQL 数据库查询插件"
}
```

**步骤 3：创建 .mcp.json**

```json
{
  "postgres": {
    "command": "npx",
    "args": [
      "-y",
      "@modelcontextprotocol/server-postgres",
      "${DATABASE_URL}"
    ]
  }
}
```

**步骤 4：创建 README.md**

```markdown
# Database Plugin

PostgreSQL 数据库查询插件。

## 安装

```bash
claude plugins install /path/to/database-plugin
```

## 配置

设置环境变量：

```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

## 使用

启动 Claude 后，可以使用以下工具：
- mcp__plugin_database_postgres__query - 执行 SQL 查询
- mcp__plugin_database_postgres__list_tables - 列出所有表
- mcp__plugin_database_postgres__describe_table - 查看表结构
```

**步骤 5：测试**

```sql
-- 让用户测试
用户：查询 users 表的前 10 条记录

Claude: 好的，执行查询。
       [使用 mcp__plugin_database_postgres__query]
       
       查询结果：
       SELECT * FROM users LIMIT 10;
       
       返回 10 条记录...
```

---

### 实战 2：集成 GitHub API（OAuth）

**目标**：创建一个可以管理 GitHub 的插件

**步骤 1：创建 .mcp.json**

```json
{
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  }
}
```

**步骤 2：创建命令**

```markdown
<!-- commands/create-issue.md -->
---
description: 创建 GitHub Issue
argument-hint: [repo] [title]
---

创建 Issue：
仓库：$1
标题：$2

使用 GitHub MCP：
mcp__plugin_github_github-tools__create_issue $1 "$2"

生成确认消息：
✅ Issue 已创建！
- 仓库：$1
- 标题：$2
- 链接：https://github.com/$1/issues/xxx
```

**步骤 3：OAuth 流程体验**

```
用户：/create-issue anthropics/claude-code "Bug: 某个功能不工作"

Claude: 检测到需要使用 GitHub 功能。
        请点击以下链接授权：
        [GitHub OAuth 授权按钮]
        
用户：点击按钮 → 跳转到 GitHub 登录 → 授权

Claude: ✅ 授权成功！
        正在创建 Issue...
        [使用 mcp__plugin_github_github-tools__create_issue]
        
        ✅ Issue #1234 已创建！
        https://github.com/anthropics/claude-code/issues/1234
```

---

### 实战 3：集成本地文件系统

**目标**：允许 Claude 访问特定目录

**步骤 1：创建 .mcp.json**

```json
{
  "filesystem": {
    "command": "npx",
    "args": [
      "-y",
      "@modelcontextprotocol/server-filesystem",
      "/Users/username/Projects/my-project"
    ]
  }
}
```

**步骤 2：创建使用文档**

```markdown
<!-- commands/list-files.md -->
---
description: 列出项目文件
---

列出项目目录：
mcp__plugin_filesystem_filesystem__list_files /Users/username/Projects/my-project

格式化输出：
📁 项目文件列表：
[文件列表...]
```

---

## 五、安全最佳实践

### 5.1 最小权限原则

**❌ 错误示例**：

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/"]
  }
}
```

**✅ 正确示例**：

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/Projects"]
  }
}
```

**为什么**：
- 🔒 只授予必要的访问权限
- 🔒 避免访问敏感文件（~/.ssh, ~/.aws 等）
- 🔒 限制在项目范围内

---

### 5.2 凭证安全

**❌ 错误**：

```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com",
    "headers": {
      "Authorization": "Bearer sk-1234567890abcdef"  // 硬编码密钥！
    }
  }
}
```

**✅ 正确**：

```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com",
    "headers": {
      "Authorization": "Bearer ${API_KEY}"  // 使用环境变量
    }
  }
}
```

**额外建议**：
- 📝 在 README 中说明需要的环境变量
- 📝 提供 .env.example 文件
- 📝 永远不要提交 .env 到 Git
- 📝 定期轮换密钥

---

### 5.3 输入验证

**在 MCP 服务器代码中验证输入**：

```javascript
// servers/custom-api.js
async function handleRequest(request) {
  const { tool, arguments } = request;
  
  // 验证必填参数
  if (tool === 'query_database') {
    if (!arguments.sql || typeof arguments.sql !== 'string') {
      throw new Error('Missing or invalid SQL query');
    }
    
    // 防止危险操作
    if (/DROP|DELETE|TRUNCATE/i.test(arguments.sql)) {
      throw new Error('Dangerous operations are not allowed');
    }
  }
  
  // 验证通过后执行
  return executeTool(tool, arguments);
}
```

---

## 六、调试技巧

### 6.1 启用调试日志

```json
{
  "my-server": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"],
    "env": {
      "DEBUG": "true",
      "LOG_LEVEL": "debug"
    }
  }
}
```

### 6.2 常见问题排查

**问题 1：MCP 服务器无法启动**

```bash
# 手动测试服务器
cd /path/to/plugin
node servers/server.js

# 检查依赖
npm install
```

**问题 2：环境变量未生效**

```bash
# 检查环境变量是否设置
echo $MY_ENV_VAR

# 在 Claude Code 中验证
claude --debug
```

**问题 3：OAuth 认证失败**

```
1. 清除旧的授权
2. 重新授权
3. 检查网络代理设置
```

---

## 七、总结

### 📝 知识点回顾

**MCP 服务器类型**：
- stdio - 本地进程，适合自定义工具
- SSE - 托管服务，支持 OAuth
- HTTP - REST API，Token 认证
- WebSocket - 实时双向通信

**配置要点**：
- ✅ 优先使用 .mcp.json
- ✅ 始终使用 ${CLAUDE_PLUGIN_ROOT}
- ✅ 通过环境变量管理密钥
- ✅ 遵循最小权限原则

**工具使用**：
- 命名格式：`mcp__plugin_<plugin>_<server>__<tool>`
- 可在 commands、agents 中使用
- 自动被 Claude 发现和调用

---

### 🎯 下一步

1. **实战练习**：选择一个你常用的服务，尝试集成 MCP
2. **参考实现**：研究 `plugins/plugin-dev/skills/mcp-integration/` 的完整示例
3. **社区资源**：查看 [MCP 官方服务器列表](https://modelcontextprotocol.io/servers)

---

**恭喜！** 🎉 你已经掌握了 MCP 集成的核心技能，开始构建你的第一个 MCP 插件吧！
