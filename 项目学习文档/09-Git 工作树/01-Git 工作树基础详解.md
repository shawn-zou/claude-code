# Git 工作树基础详解

## 📚 本章概述

Git 工作树（Worktree）是 Git 2.5 版本引入的一个强大功能，它允许你在同一个仓库中同时检出多个分支到不同的目录。本章将深入讲解 Git 工作树的概念、使用方法和在 Claude Code 项目中的应用。

---

## 🎯 学习目标

完成本章学习后，你将能够：

- ✅ 理解 Git 工作树的核心概念
- ✅ 掌握 `git worktree` 命令的使用
- ✅ 理解工作树与分支的关系
- ✅ 在 Claude Code 中使用 `--worktree` 参数
- ✅ 理解 Agent 隔离机制中的工作树应用

---

## 🌳 一、什么是 Git 工作树？

### 1.1 传统 Git 工作方式的问题

在传统的 Git 工作流中，你一次只能在一个分支上工作：

```
问题场景：
├── 你正在 feature-A 分支开发新功能
├── 突然需要紧急修复 main 分支的 bug
├── 你必须：
│   ├── 暂存当前修改（git stash）
│   ├── 切换分支（git checkout main）
│   ├── 修复 bug 并提交
│   ├── 切换回 feature-A
│   └── 恢复暂存的修改（git stash pop）
└── 这个过程繁琐且容易出错
```

### 1.2 工作树的解决方案

Git 工作树允许你**同时**在多个分支上工作：

```
工作树解决方案：
项目根目录/
├── .git/                    # 主仓库的 Git 目录
├── src/                     # 主工作目录（main 分支）
├── feature-A/               # 工作树 1（feature-A 分支）
│   └── src/                 # feature-A 的代码
└── hotfix-123/              # 工作树 2（hotfix-123 分支）
    └── src/                 # hotfix-123 的代码

优势：
✅ 无需切换分支
✅ 无需暂存修改
✅ 可以同时运行多个开发环境
✅ 每个工作树独立的 IDE 窗口
```

### 1.3 核心概念对比

| 概念 | 说明 | 类比 |
|------|------|------|
| **仓库（Repository）** | 包含所有版本历史的数据库 | 图书馆的书库 |
| **工作树（Worktree）** | 检出特定分支的目录 | 图书馆的阅览室 |
| **分支（Branch）** | 一系列提交的指针 | 书的某个版本 |
| **HEAD** | 当前工作树指向的分支 | 你正在读的那本书 |

---

## 📖 二、Git Worktree 命令详解

### 2.1 创建工作树

#### 基本语法

```bash
# 基于现有分支创建工作树
git worktree add <路径> <分支名>

# 基于新分支创建工作树
git worktree add -b <新分支名> <路径> <起点分支>
```

#### 实际示例

```bash
# 示例 1：基于现有分支创建工作树
git worktree add ../my-project-feature feature-login

# 示例 2：创建新分支并同时创建工作树
git worktree add -b feature-payment ../my-project-payment main

# 示例 3：基于远程分支创建工作树
git worktree add ../my-project-hotfix origin/hotfix-123
```

#### 创建后的目录结构

```
创建前：
my-project/
├── .git/
└── src/

创建后：
my-project/                    # 主工作树（main 分支）
├── .git/
└── src/

my-project-feature/            # 新工作树（feature-login 分支）
├── .git                       # 指向主仓库的文件
└── src/

my-project-payment/            # 新工作树（feature-payment 分支）
├── .git                       # 指向主仓库的文件
└── src/
```

### 2.2 列出工作树

```bash
# 列出所有工作树
git worktree list

# 输出示例：
# /path/to/main          abc1234 [main]
# /path/to/feature       def5678 [feature-login]
# /path/to/hotfix        ghi9012 [hotfix-123]
```

#### 输出字段说明

| 字段 | 说明 |
|------|------|
| 路径 | 工作树的目录路径 |
| 提交哈希 | 当前 HEAD 指向的提交 |
| 分支名 | 当前检出的分支（方括号内） |

### 2.3 删除工作树

```bash
# 基本删除
git worktree remove <路径>

# 强制删除（有未提交的修改时）
git worktree remove --force <路径>

# 删除已合并的工作树
git worktree prune
```

#### 删除流程图

```
┌─────────────────────────────────────────┐
│           删除工作树流程                  │
└─────────────────────────────────────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  检查是否有未提交修改   │
        └───────────────────────┘
                    │
           ┌────────┴────────┐
           │                 │
           ▼                 ▼
    ┌────────────┐    ┌────────────┐
    │  有修改     │    │  无修改     │
    └────────────┘    └────────────┘
           │                 │
           ▼                 ▼
    ┌────────────┐    ┌────────────┐
    │ 使用 --force│    │ 直接删除    │
    │ 或先提交    │    │            │
    └────────────┘    └────────────┘
```

### 2.4 清理陈旧工作树

```bash
# 清理已删除分支的工作树引用
git worktree prune

# 清理并显示详细信息
git worktree prune -v

# 设置过期时间（默认 3 个月）
git worktree prune --expire=2.weeks.ago
```

---

## 🔧 三、Claude Code 中的工作树应用

### 3.1 `--worktree` 命令行参数

Claude Code 提供了 `--worktree`（或 `-w`）参数，用于在隔离的 Git 工作树中启动会话。

#### 基本用法

```bash
# 在新工作树中启动 Claude Code
claude --worktree

# 简写形式
claude -w

# 指定工作树名称
claude --worktree --name my-feature
```

#### 工作原理

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code --worktree 流程               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  检查当前是否在 Git 仓库 │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  创建新的 Git 工作树    │
                  │  （基于当前分支）       │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  触发 WorktreeCreate   │
                  │  Hook 事件             │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  在隔离环境中启动       │
                  │  Claude Code 会话      │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  会话结束后清理工作树   │
                  │  触发 WorktreeRemove   │
                  └───────────────────────┘
```

### 3.2 Agent 隔离机制

Claude Code 的 Agent 可以配置为在隔离的工作树中运行。

#### 配置示例

```yaml
# agent 定义文件中的配置
---
name: my-isolated-agent
description: 在隔离工作树中运行的 Agent
isolation: worktree    # 关键配置：启用工作树隔离
tools: Glob, Read, Bash
---

# Agent 的 System Prompt
你是一个在隔离环境中运行的 Agent...
```

#### 隔离机制的优势

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent 工作树隔离机制                       │
└─────────────────────────────────────────────────────────────┘

主工作树                          隔离工作树（Agent 1）
┌─────────────────┐              ┌─────────────────┐
│  main 分支      │              │  临时分支        │
│  ┌───────────┐  │              │  ┌───────────┐  │
│  │ 用户代码   │  │    隔离      │  │ Agent 修改 │  │
│  │ 不受影响   │◄─┼──────────────┼──│ 不会影响   │  │
│  └───────────┘  │              │  │ 主分支     │  │
└─────────────────┘              └─────────────────┘

优势：
✅ Agent 的修改不会影响主工作树
✅ 多个 Agent 可以并行工作
✅ Agent 完成后自动清理
✅ 支持回滚和重试
```

### 3.3 大型 Monorepo 的稀疏检出

对于大型 Monorepo，Claude Code 支持 `worktree.sparsePaths` 设置：

```json
// settings.json
{
  "worktree": {
    "sparsePaths": [
      "packages/frontend/",
      "packages/shared/",
      "configs/"
    ]
  }
}
```

#### 稀疏检出流程

```
┌─────────────────────────────────────────────────────────────┐
│                    稀疏检出流程                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  创建工作树            │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  初始化稀疏检出        │
                  │  git sparse-checkout  │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  只检出指定目录        │
                  │  （减少磁盘占用）      │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  加快启动速度          │
                  └───────────────────────┘
```

---

## 📊 四、工作树与分支的关系

### 4.1 一对一关系

每个工作树对应一个分支：

```
工作树 1 ─────────────► 分支 A
工作树 2 ─────────────► 分支 B
工作树 3 ─────────────► 分支 C

注意：同一个分支不能被多个工作树同时检出
```

### 4.2 分支切换限制

```bash
# ❌ 错误：分支已被其他工作树检出
$ git checkout feature-login
fatal: 'feature-login' is already checked out at '/path/to/other-worktree'

# ✅ 正确：在该工作树中操作
$ cd /path/to/other-worktree
$ git status
On branch feature-login
```

### 4.3 工作树状态同步

```
┌─────────────────────────────────────────────────────────────┐
│                    工作树状态同步机制                         │
└─────────────────────────────────────────────────────────────┘

主工作树                           工作树 A
┌─────────────────┐              ┌─────────────────┐
│  git commit     │              │                 │
│  新提交 X       │              │                 │
└────────┬────────┘              └─────────────────┘
         │
         │ git push
         ▼
┌─────────────────┐
│   远程仓库       │
│   包含提交 X     │
└────────┬────────┘
         │
         │ git fetch（在工作树 A 中）
         ▼
┌─────────────────┐              ┌─────────────────┐
│                 │              │  git merge      │
│                 │              │  获取提交 X      │
└─────────────────┘              └─────────────────┘

关键点：
- 所有工作树共享同一个 .git 目录
- git fetch/pull 在任何工作树中都会更新远程跟踪分支
- 但本地分支的更新需要在对应工作树中操作
```

---

## 🛠️ 五、实战案例

### 5.1 场景：并行开发多个功能

```bash
# 步骤 1：创建主工作树（假设已存在）
cd /projects/my-app

# 步骤 2：为功能 A 创建工作树
git worktree add ../my-app-feature-a feature-a

# 步骤 3：为功能 B 创建工作树
git worktree add ../my-app-feature-b feature-b

# 步骤 4：在各自的 IDE 中打开
code ../my-app-feature-a    # VS Code 打开功能 A
code ../my-app-feature-b    # VS Code 打开功能 B

# 步骤 5：并行开发
# 在功能 A 工作树中开发...
# 在功能 B 工作树中开发...

# 步骤 6：完成后清理
git worktree remove ../my-app-feature-a
git worktree remove ../my-app-feature-b
```

### 5.2 场景：紧急修复

```bash
# 当前正在开发功能，突然需要修复 bug

# 步骤 1：创建热修复工作树
git worktree add ../my-app-hotfix -b hotfix-123 main

# 步骤 2：在热修复工作树中修复
cd ../my-app-hotfix
# 修复代码...
git add .
git commit -m "fix: resolve issue #123"

# 步骤 3：推送修复
git push origin hotfix-123

# 步骤 4：返回原工作树继续开发
cd ../my-app
# 继续开发，无需 stash

# 步骤 5：清理热修复工作树
git worktree remove ../my-app-hotfix
```

### 5.3 场景：使用 Claude Code 的 --worktree

```bash
# 场景：让 Claude 在隔离环境中执行可能有大改动的任务

# 步骤 1：使用 --worktree 启动
claude --worktree

# Claude 会：
# 1. 创建新的工作树
# 2. 在隔离环境中执行任务
# 3. 完成后清理工作树

# 步骤 2：如果满意，合并修改
# Claude 会引导你完成合并流程
```

---

## 📋 六、常用命令速查表

| 命令 | 说明 | 示例 |
|------|------|------|
| `git worktree add` | 创建新工作树 | `git worktree add ../path branch` |
| `git worktree list` | 列出所有工作树 | `git worktree list` |
| `git worktree remove` | 删除工作树 | `git worktree remove ../path` |
| `git worktree prune` | 清理陈旧引用 | `git worktree prune -v` |
| `git worktree lock` | 锁定工作树 | `git worktree lock ../path` |
| `git worktree unlock` | 解锁工作树 | `git worktree unlock ../path` |
| `claude --worktree` | Claude 隔离模式 | `claude -w` |
| `claude -w --name` | 指定工作树名称 | `claude -w --name my-feature` |

---

## ⚠️ 七、注意事项与最佳实践

### 7.1 注意事项

1. **分支独占**：同一分支不能被多个工作树同时检出
2. **磁盘空间**：每个工作树都会占用磁盘空间
3. **路径管理**：建议使用一致的命名规范
4. **清理习惯**：完成工作后及时清理工作树

### 7.2 最佳实践

```
✅ 推荐做法：

1. 命名规范
   ├── 使用描述性名称：my-app-feature-login
   └── 统一前缀/后缀

2. 目录组织
   ├── 所有工作树放在同一父目录
   └── 例如：~/worktrees/my-app-feature-a

3. 定期清理
   ├── 每周运行 git worktree prune
   └── 删除已完成的工作树

4. IDE 配置
   ├── 每个工作树独立的 IDE 窗口
   └── 避免在同一个 IDE 中打开多个工作树
```

---

## 🔗 八、与项目其他模块的关联

```
Git 工作树
    │
    ├──► Hook 系统
    │    ├── WorktreeCreate Hook
    │    └── WorktreeRemove Hook
    │
    ├──► Agent 协作系统
    │    └── isolation: worktree 配置
    │
    ├──► commit-commands 插件
    │    └── /clean_gone 命令
    │
    └──► 插件系统
         └── 工作树相关的 Hook 事件
```

---

## 📝 九、学习检查清单

完成本章学习后，请确认：

- [ ] 理解 Git 工作树的基本概念
- [ ] 能使用 `git worktree add` 创建工作树
- [ ] 能使用 `git worktree list` 查看工作树
- [ ] 能使用 `git worktree remove` 删除工作树
- [ ] 理解 Claude Code 的 `--worktree` 参数
- [ ] 理解 Agent 的 `isolation: worktree` 配置
- [ ] 了解工作树与分支的关系
- [ ] 掌握工作树的最佳实践

---

## 📚 延伸阅读

- [Git 官方文档 - git-worktree](https://git-scm.com/docs/git-worktree)
- [Pro Git - Git 工具 - 工作区](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%B7%A5%E4%BD%9C%E5%8C%BA)
- 本项目 CHANGELOG.md 中的 worktree 相关更新

---

**下一章：** [commit-commands 插件详解](./02-commit-commands%20插件详解.md)

---

**最后更新：** 2026 年 4 月 6 日  
**维护者：** Claude Code 项目学习文档团队
