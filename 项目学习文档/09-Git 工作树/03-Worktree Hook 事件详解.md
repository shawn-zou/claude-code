# Worktree Hook 事件详解

## 📚 本章概述

Claude Code 提供了两个与 Git 工作树相关的 Hook 事件：`WorktreeCreate` 和 `WorktreeRemove`。这些 Hook 允许你在工作树创建和删除时执行自定义逻辑，实现版本控制系统的自定义设置和清理。

---

## 🎯 学习目标

完成本章学习后，你将能够：

- ✅ 理解 WorktreeCreate 和 WorktreeRemove Hook 事件
- ✅ 掌握 Hook 的触发时机和参数
- ✅ 编写自定义的工作树 Hook
- ✅ 实现工作树的自动化配置和清理

---

## 🔔 一、WorktreeCreate Hook

### 1.1 事件触发时机

`WorktreeCreate` Hook 在以下场景触发：

```
触发场景：
├── 使用 claude --worktree 启动会话
├── Agent 配置了 isolation: worktree
├── 手动创建工作树（通过 Claude 执行 git worktree add）
└── 并行任务需要隔离环境
```

### 1.2 工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│                  WorktreeCreate Hook 触发流程                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  请求创建工作树        │
                  │  (claude --worktree   │
                  │   或 Agent isolation) │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  执行 git worktree add│
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  触发 WorktreeCreate  │
                  │  Hook 事件            │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  执行 Hook 逻辑        │
                  │  (自定义配置)          │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  继续主流程            │
                  │  (启动会话/Agent)      │
                  └───────────────────────┘
```

### 1.3 Hook 参数

```json
{
  "event": "WorktreeCreate",
  "worktreePath": "/path/to/new/worktree",
  "worktreeBranch": "feature-branch",
  "mainRepoPath": "/path/to/main/repo",
  "reason": "agent_isolation" | "cli_flag" | "manual"
}
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `worktreePath` | string | 新工作树的绝对路径 |
| `worktreeBranch` | string | 工作树检出的分支名 |
| `mainRepoPath` | string | 主仓库的路径 |
| `reason` | string | 创建原因 |

### 1.4 Hook 配置示例

#### 示例 1：HTTP 类型 Hook

```json
{
  "hooks": {
    "WorktreeCreate": [
      {
        "type": "http",
        "url": "https://api.example.com/worktree/create",
        "method": "POST",
        "headers": {
          "Authorization": "Bearer ${WORKTREE_API_TOKEN}"
        },
        "body": {
          "path": "${worktreePath}",
          "branch": "${worktreeBranch}",
          "repo": "${mainRepoPath}"
        }
      }
    ]
  }
}
```

#### 示例 2：Command 类型 Hook

```json
{
  "hooks": {
    "WorktreeCreate": [
      {
        "type": "command",
        "script": "./scripts/setup-worktree.sh",
        "env": {
          "WORKTREE_PATH": "${worktreePath}",
          "BRANCH_NAME": "${worktreeBranch}"
        }
      }
    ]
  }
}
```

### 1.5 实用脚本示例

```bash
#!/bin/bash
# scripts/setup-worktree.sh
# 在新工作树中执行初始化配置

WORKTREE_PATH="${WORKTREE_PATH:-}"
BRANCH_NAME="${BRANCH_NAME:-}"

echo "Setting up worktree at: $WORKTREE_PATH"
echo "Branch: $BRANCH_NAME"

# 切换到工作树目录
cd "$WORKTREE_PATH" || exit 1

# 安装依赖
if [ -f "package.json" ]; then
  echo "Installing npm dependencies..."
  npm install --prefer-offline
fi

# 复制环境配置
if [ -f "../.env.example" ]; then
  echo "Copying environment file..."
  cp "../.env.example" ".env"
fi

# 设置 Git 配置
git config user.name "Claude Code Agent"
git config user.email "agent@claude.ai"

echo "Worktree setup complete!"
```

### 1.6 HTTP Hook 响应格式

```json
{
  "status": "success",
  "message": "Worktree configured successfully",
  "hookSpecificOutput": {
    "worktreePath": "/path/to/configured/worktree"
  }
}
```

**重要：** HTTP 类型的 Hook 可以通过 `hookSpecificOutput.worktreePath` 返回工作树路径。

---

## 🔕 二、WorktreeRemove Hook

### 2.1 事件触发时机

`WorktreeRemove` Hook 在以下场景触发：

```
触发场景：
├── Claude Code 会话结束（--worktree 模式）
├── Agent 任务完成（isolation: worktree）
├── 手动删除工作树
├── /clean_gone 命令清理工作树
└── 并行任务清理
```

### 2.2 工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│                  WorktreeRemove Hook 触发流程                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  请求删除工作树        │
                  │  (会话结束/任务完成)   │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  触发 WorktreeRemove  │
                  │  Hook 事件            │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  执行 Hook 逻辑        │
                  │  (清理操作)            │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  执行 git worktree    │
                  │  remove               │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  清理完成              │
                  └───────────────────────┘
```

### 2.3 Hook 参数

```json
{
  "event": "WorktreeRemove",
  "worktreePath": "/path/to/worktree",
  "worktreeBranch": "feature-branch",
  "mainRepoPath": "/path/to/main/repo",
  "reason": "session_end" | "agent_complete" | "manual" | "cleanup"
}
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `worktreePath` | string | 要删除的工作树路径 |
| `worktreeBranch` | string | 工作树的分支名 |
| `mainRepoPath` | string | 主仓库的路径 |
| `reason` | string | 删除原因 |

### 2.4 Hook 配置示例

```json
{
  "hooks": {
    "WorktreeRemove": [
      {
        "type": "command",
        "script": "./scripts/cleanup-worktree.sh",
        "env": {
          "WORKTREE_PATH": "${worktreePath}",
          "BRANCH_NAME": "${worktreeBranch}"
        }
      }
    ]
  }
}
```

### 2.5 实用脚本示例

```bash
#!/bin/bash
# scripts/cleanup-worktree.sh
# 清理工作树相关资源

WORKTREE_PATH="${WORKTREE_PATH:-}"
BRANCH_NAME="${BRANCH_NAME:-}"

echo "Cleaning up worktree: $WORKTREE_PATH"
echo "Branch: $BRANCH_NAME"

# 清理 node_modules（释放磁盘空间）
if [ -d "$WORKTREE_PATH/node_modules" ]; then
  echo "Removing node_modules..."
  rm -rf "$WORKTREE_PATH/node_modules"
fi

# 清理构建产物
if [ -d "$WORKTREE_PATH/dist" ]; then
  echo "Removing build artifacts..."
  rm -rf "$WORKTREE_PATH/dist"
fi

# 清理临时文件
find "$WORKTREE_PATH" -name "*.log" -delete 2>/dev/null
find "$WORKTREE_PATH" -name ".DS_Store" -delete 2>/dev/null

# 记录清理日志
echo "$(date): Cleaned up worktree at $WORKTREE_PATH" >> /tmp/worktree-cleanup.log

echo "Cleanup complete!"
```

---

## 🔄 三、两个 Hook 的协作

### 3.1 完整生命周期

```
┌─────────────────────────────────────────────────────────────┐
│                    工作树完整生命周期                         │
└─────────────────────────────────────────────────────────────┘

                    ┌───────────────┐
                    │   开始请求     │
                    └───────┬───────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    WorktreeCreate Hook                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ • 创建工作树目录                                      │   │
│  │ • 检出代码                                            │   │
│  │ • 安装依赖                                            │   │
│  │ • 配置环境                                            │   │
│  │ • 设置 Git 配置                                       │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  工作树使用中  │
                    │  (执行任务)    │
                    └───────┬───────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    WorktreeRemove Hook                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ • 清理临时文件                                        │   │
│  │ • 归档日志                                            │   │
│  │ • 释放资源                                            │   │
│  │ • 删除工作树                                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │    完成        │
                    └───────────────┘
```

### 3.2 配置文件示例

```json
{
  "hooks": {
    "WorktreeCreate": [
      {
        "type": "command",
        "script": "./scripts/worktree/setup.sh",
        "timeout": 60
      }
    ],
    "WorktreeRemove": [
      {
        "type": "command",
        "script": "./scripts/worktree/cleanup.sh",
        "timeout": 30
      }
    ]
  }
}
```

---

## 🎯 四、实际应用场景

### 场景 1：自动化开发环境配置

```bash
# setup-worktree.sh
#!/bin/bash

cd "$WORKTREE_PATH"

# 1. 安装依赖
npm ci --prefer-offline

# 2. 设置本地配置
cp .env.example .env.local

# 3. 初始化数据库（如果有）
if [ -f "scripts/init-db.sh" ]; then
  ./scripts/init-db.sh --test
fi

# 4. 启动开发服务（后台）
npm run dev &
echo $! > /tmp/worktree-dev.pid
```

### 场景 2：清理和归档

```bash
# cleanup-worktree.sh
#!/bin/bash

cd "$WORKTREE_PATH"

# 1. 收集测试覆盖率报告
if [ -d "coverage" ]; then
  cp -r coverage "$MAIN_REPO_PATH/reports/$BRANCH_NAME-coverage"
fi

# 2. 归档日志
if [ -d "logs" ]; then
  tar -czf "$MAIN_REPO_PATH/archives/$BRANCH_NAME-logs.tar.gz" logs/
fi

# 3. 停止后台服务
if [ -f "/tmp/worktree-dev.pid" ]; then
  kill $(cat /tmp/worktree-dev.pid) 2>/dev/null
  rm /tmp/worktree-dev.pid
fi
```

### 场景 3：通知和监控

```json
{
  "hooks": {
    "WorktreeCreate": [
      {
        "type": "http",
        "url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
        "method": "POST",
        "body": {
          "text": "🌳 New worktree created: ${worktreeBranch}\nPath: ${worktreePath}"
        }
      }
    ],
    "WorktreeRemove": [
      {
        "type": "http",
        "url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
        "method": "POST",
        "body": {
          "text": "🧹 Worktree cleaned up: ${worktreeBranch}"
        }
      }
    ]
  }
}
```

---

## 📊 五、与其他 Hook 的关系

```
Hook 事件时间线：
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  SessionStart ──► WorktreeCreate ──► [工作] ──► WorktreeRemove │
│       │                  │                        │          │
│       │                  │                        │          │
│       ▼                  ▼                        ▼          │
│  会话初始化          工作树配置              工作树清理        │
│                                                              │
└─────────────────────────────────────────────────────────────┘

关联的 Hook 事件：
├── SessionStart：会话开始时触发
├── WorktreeCreate：工作树创建时触发
├── PreToolUse：工具使用前检查
├── PostToolUse：工具使用后处理
├── Stop：任务完成时触发
└── WorktreeRemove：工作树删除时触发
```

---

## ⚠️ 六、注意事项

### 6.1 WorktreeCreate 注意事项

```
✅ 确保 Hook 脚本有执行权限
✅ 设置合理的超时时间
✅ 处理可能的错误情况
✅ 避免长时间阻塞操作

❌ 不要修改主仓库的文件
❌ 不要执行需要用户交互的命令
❌ 不要假设网络一定可用
```

### 6.2 WorktreeRemove 注意事项

```
✅ 确保清理操作是幂等的
✅ 处理工作树不存在的情况
✅ 记录清理日志
✅ 释放所有占用的资源

❌ 不要删除用户数据
❌ 不要影响其他正在运行的工作树
❌ 不要执行耗时过长的操作
```

---

## 📝 七、学习检查清单

完成本章学习后，请确认：

- [ ] 理解 WorktreeCreate Hook 的触发时机
- [ ] 理解 WorktreeRemove Hook 的触发时机
- [ ] 能配置 HTTP 类型的 Hook
- [ ] 能配置 Command 类型的 Hook
- [ ] 能编写实用的 Hook 脚本
- [ ] 理解两个 Hook 的协作关系
- [ ] 了解实际应用场景

---

## 📚 延伸阅读

- [Hook 系统基础](../02-Hook%20系统/01-Hook%20系统基础.md)
- [Agent 协作系统](../04-Agent%20协作系统/01-Agent%20协作系统详解.md)
- 本项目 CHANGELOG.md 中的 WorktreeCreate/WorktreeRemove 更新记录

---

**下一章：** [Git 工作流最佳实践](./04-Git%20工作流最佳实践.md)

---

**最后更新：** 2026 年 4 月 6 日  
**维护者：** Claude Code 项目学习文档团队
