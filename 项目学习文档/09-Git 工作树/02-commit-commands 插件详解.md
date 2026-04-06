# commit-commands 插件详解

## 📚 本章概述

`commit-commands` 是 Claude Code 项目中的一个实用插件，它提供了三个核心命令来简化 Git 工作流：`/commit`、`/commit-push-pr` 和 `/clean_gone`。本章将深入讲解这个插件的使用方法和内部实现。

---

## 🎯 学习目标

完成本章学习后，你将能够：

- ✅ 理解 commit-commands 插件的功能
- ✅ 掌握三个核心命令的使用方法
- ✅ 理解命令的内部实现原理
- ✅ 在实际项目中应用这些命令
- ✅ 结合工作树实现高效的 Git 工作流

---

## 📦 一、插件概述

### 1.1 插件位置

```
plugins/commit-commands/
├── .claude-plugin/
│   └── plugin.json          # 插件元信息
├── commands/
│   ├── commit.md            # /commit 命令定义
│   ├── commit-push-pr.md    # /commit-push-pr 命令定义
│   └── clean_gone.md        # /clean_gone 命令定义
└── README.md                # 插件说明文档
```

### 1.2 插件功能概览

| 命令 | 功能 | 适用场景 |
|------|------|----------|
| `/commit` | 自动生成提交信息并提交 | 日常开发提交 |
| `/commit-push-pr` | 提交、推送、创建 PR 一条龙 | 功能开发完成 |
| `/clean_gone` | 清理已删除的远程分支 | 仓库维护 |

---

## 🔧 二、/commit 命令详解

### 2.1 命令功能

`/commit` 命令会自动分析你的代码变更，生成符合项目风格的提交信息，然后执行提交。

### 2.2 命令定义文件

**文件位置：** `plugins/commit-commands/commands/commit.md`

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
description: Create a git commit
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your task

Based on the above changes, create a single git commit.

You have the capability to call multiple tools in a single response. Stage and create the commit using a single message. Do not use any other tools or do anything else. Do not send any other text or messages besides these tool calls.
```

### 2.3 工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    /commit 命令工作流程                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  获取当前 Git 状态     │
                  │  git status           │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  获取代码变更内容      │
                  │  git diff HEAD        │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  获取当前分支名        │
                  │  git branch --show-current │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  获取最近提交记录      │
                  │  git log --oneline -10│
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  分析提交风格          │
                  │  （从历史提交中学习）   │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  生成提交信息          │
                  │  （匹配项目风格）       │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  暂存相关文件          │
                  │  git add              │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  执行提交              │
                  │  git commit           │
                  └───────────────────────┘
```

### 2.4 核心特性

#### 特性 1：自动匹配提交风格

```bash
# 假设项目历史提交风格为：
# feat: add user authentication
# fix: resolve login timeout issue
# docs: update API documentation

# /commit 会自动生成类似风格的提交信息
# 例如：feat: add password reset functionality
```

#### 特性 2：遵循约定式提交

```
约定式提交格式：
<类型>(<范围>): <描述>

类型包括：
- feat: 新功能
- fix: Bug 修复
- docs: 文档更新
- style: 代码格式（不影响功能）
- refactor: 重构
- test: 测试相关
- chore: 构建/工具相关
```

#### 特性 3：安全检查

```
自动避免提交：
❌ .env 文件
❌ credentials.json
❌ 包含密钥的配置文件
❌ 其他敏感信息文件
```

### 2.5 使用示例

```bash
# 场景：完成了一个新功能的开发

# 步骤 1：查看当前状态
$ git status
On branch feature-login
Changes not staged for commit:
  modified:   src/auth/login.ts
  modified:   src/auth/types.ts

# 步骤 2：执行 /commit
$ /commit

# Claude 会：
# 1. 分析变更内容
# 2. 生成提交信息：feat(auth): add login functionality
# 3. 暂存文件
# 4. 执行提交

# 步骤 3：查看结果
$ git log -1
commit abc1234...
Author: You <you@example.com>
Date:   Mon Apr 6 2026

    feat(auth): add login functionality
    
    🤖 Generated with [Claude Code](https://claude.ai/code)
```

### 2.6 allowed-tools 配置解析

```yaml
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
```

| 配置 | 说明 |
|------|------|
| `Bash(git add:*)` | 允许执行 `git add` 命令（任意参数） |
| `Bash(git status:*)` | 允许执行 `git status` 命令 |
| `Bash(git commit:*)` | 允许执行 `git commit` 命令 |

**安全设计：** 只允许执行特定的 Git 命令，防止误操作。

---

## 🚀 三、/commit-push-pr 命令详解

### 3.1 命令功能

`/commit-push-pr` 是一个完整的 Git 工作流命令，它会：
1. 创建新分支（如果当前在 main）
2. 提交代码变更
3. 推送到远程仓库
4. 创建 Pull Request

### 3.2 命令定义文件

**文件位置：** `plugins/commit-commands/commands/commit-push-pr.md`

```markdown
---
allowed-tools: Bash(git checkout --branch:*), Bash(git add:*), Bash(git status:*), Bash(git push:*), Bash(git commit:*), Bash(gh pr create:*)
description: Commit, push, and open a PR
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`

## Your task

Based on the above changes:

1. Create a new branch if on main
2. Create a single commit with an appropriate message
3. Push the branch to origin
4. Create a pull request using `gh pr create`
5. You have the capability to call multiple tools in a single response. You MUST do all of the above in a single message. Do not use any other tools or do anything else. Do not send any other text or messages besides these tool calls.
```

### 3.3 工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│                /commit-push-pr 命令工作流程                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  检查当前分支          │
                  └───────────────────────┘
                              │
               ┌──────────────┴──────────────┐
               │                             │
               ▼                             ▼
        ┌────────────┐                ┌────────────┐
        │  在 main   │                │ 不在 main  │
        └────────────┘                └────────────┘
               │                             │
               ▼                             │
        ┌────────────┐                       │
        │ 创建新分支 │                       │
        └────────────┘                       │
               │                             │
               └──────────────┬──────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  分析代码变更          │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  生成提交信息          │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  暂存并提交            │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  推送到远程            │
                  │  git push origin      │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  创建 Pull Request    │
                  │  gh pr create         │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  返回 PR URL          │
                  └───────────────────────┘
```

### 3.4 PR 描述生成

```
PR 描述包含：
├── 变更摘要（1-3 个要点）
├── 测试计划清单
└── Claude Code 标识

示例：
## Summary
- Add user authentication with JWT tokens
- Implement password hashing with bcrypt
- Add login/logout API endpoints

## Test plan
- [ ] Test login with valid credentials
- [ ] Test login with invalid credentials
- [ ] Test token refresh flow
- [ ] Test logout functionality

🤖 Generated with [Claude Code](https://claude.ai/code)
```

### 3.5 使用示例

```bash
# 场景：完成功能开发，准备创建 PR

# 步骤 1：查看当前状态
$ git status
On branch main
Changes not staged for commit:
  modified:   src/features/payment.ts
  new file:   src/features/refund.ts

# 步骤 2：执行 /commit-push-pr
$ /commit-push-pr

# Claude 会：
# 1. 创建新分支：feature-payment-refund
# 2. 提交变更
# 3. 推送到远程
# 4. 创建 PR

# 步骤 3：查看结果
# Claude 返回 PR URL：
# https://github.com/owner/repo/pull/42
```

### 3.6 前置要求

```bash
# 必须安装 GitHub CLI
$ gh --version
gh version 2.40.0

# 必须已登录
$ gh auth status
✓ Logged in to github.com as your-username

# 仓库必须有 origin 远程
$ git remote -v
origin  https://github.com/owner/repo.git (fetch)
origin  https://github.com/owner/repo.git (push)
```

---

## 🧹 四、/clean_gone 命令详解

### 4.1 命令功能

`/clean_gone` 命令用于清理本地已删除的远程分支（标记为 `[gone]` 的分支），包括删除关联的工作树。

### 4.2 什么是 [gone] 分支？

```
当远程分支被删除后，本地分支会显示为 [gone]：

$ git branch -v
  feature-a     abc1234 [origin/feature-a] Some commit
  feature-b     def5678 [gone] Old feature        ← 远程已删除
* main          ghi9012 [origin/main] Latest
  hotfix-123    jkl3456 [gone] Merged hotfix      ← 远程已删除
```

### 4.3 命令定义文件

**文件位置：** `plugins/commit-commands/commands/clean_gone.md`

```markdown
---
description: Cleans up all git branches marked as [gone] (branches that have been deleted on the remote but still exist locally), including removing associated worktrees.
---

## Your Task

You need to execute the following bash commands to clean up stale local branches that have been deleted from the remote repository.

## Commands to Execute

1. **First, list branches to identify any with [gone] status**
   Execute this command:
   ```bash
   git branch -v
   ```
   
   Note: Branches with a '+' prefix have associated worktrees and must have their worktrees removed before deletion.

2. **Next, identify worktrees that need to be removed for [gone] branches**
   Execute this command:
   ```bash
   git worktree list
   ```

3. **Finally, remove worktrees and delete [gone] branches (handles both regular and worktree branches)**
   Execute this command:
   ```bash
   # Process all [gone] branches, removing '+' prefix if present
   git branch -v | grep '\[gone\]' | sed 's/^[+* ]//' | awk '{print $1}' | while read branch; do
     echo "Processing branch: $branch"
     # Find and remove worktree if it exists
     worktree=$(git worktree list | grep "\\[$branch\\]" | awk '{print $1}')
     if [ ! -z "$worktree" ] && [ "$worktree" != "$(git rev-parse --show-toplevel)" ]; then
       echo "  Removing worktree: $worktree"
       git worktree remove --force "$worktree"
     fi
     # Delete the branch
     echo "  Deleting branch: $branch"
     git branch -D "$branch"
   done
   ```

## Expected Behavior

After executing these commands, you will:

- See a list of all local branches with their status
- Identify and remove any worktrees associated with [gone] branches
- Delete all branches marked as [gone]
- Provide feedback on which worktrees and branches were removed

If no branches are marked as [gone], report that no cleanup was needed.
```

### 4.4 工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│                  /clean_gone 命令工作流程                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  列出所有本地分支      │
                  │  git branch -v        │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  识别 [gone] 分支      │
                  └───────────────────────┘
                              │
               ┌──────────────┴──────────────┐
               │                             │
               ▼                             ▼
        ┌────────────┐                ┌────────────┐
        │ 有 [gone]  │                │ 无 [gone]  │
        └────────────┘                └────────────┘
               │                             │
               ▼                             ▼
        ┌────────────┐                ┌────────────┐
        │ 列出工作树 │                │ 报告无需清理│
        └────────────┘                └────────────┘
               │
               ▼
        ┌────────────┐
        │ 检查关联   │
        │ 工作树     │
        └────────────┘
               │
       ┌───────┴───────┐
       │               │
       ▼               ▼
┌────────────┐  ┌────────────┐
│ 有工作树   │  │ 无工作树   │
└────────────┘  └────────────┘
       │               │
       ▼               │
┌────────────┐         │
│ 删除工作树 │         │
└────────────┘         │
       │               │
       └───────┬───────┘
               │
               ▼
        ┌────────────┐
        │ 删除分支   │
        │ git branch -D │
        └────────────┘
               │
               ▼
        ┌────────────┐
        │ 报告清理结果│
        └────────────┘
```

### 4.5 核心脚本解析

```bash
# 步骤 1：获取所有 [gone] 分支
git branch -v | grep '\[gone\]'

# 输出示例：
#   feature-b     def5678 [gone] Old feature
# + hotfix-123    jkl3456 [gone] Merged hotfix  ← '+' 表示有工作树

# 步骤 2：提取分支名
sed 's/^[+* ]//' | awk '{print $1}'

# 步骤 3：检查并删除工作树
worktree=$(git worktree list | grep "\\[$branch\\]" | awk '{print $1}')
if [ ! -z "$worktree" ]; then
  git worktree remove --force "$worktree"
fi

# 步骤 4：删除分支
git branch -D "$branch"
```

### 4.6 使用示例

```bash
# 场景：多个 PR 已合并，远程分支已删除

# 步骤 1：查看当前状态
$ git branch -v
  feature-a     abc1234 [origin/feature-a] Active feature
  feature-b     def5678 [gone] Merged feature    ← 需要清理
* main          ghi9012 [origin/main] Main branch
  hotfix-123    jkl3456 [gone] Merged hotfix     ← 需要清理

# 步骤 2：执行 /clean_gone
$ /clean_gone

# Claude 会执行：
# Processing branch: feature-b
#   Deleting branch: feature-b
# Processing branch: hotfix-123
#   Removing worktree: /path/to/hotfix-123
#   Deleting branch: hotfix-123

# 步骤 3：验证清理结果
$ git branch -v
  feature-a     abc1234 [origin/feature-a] Active feature
* main          ghi9012 [origin/main] Main branch
```

### 4.7 与工作树的配合

```
/clean_gone 命令特别处理有工作树的分支：

分支状态识别：
┌─────────────────────────────────────────┐
│  git branch -v 输出                      │
├─────────────────────────────────────────┤
│  feature-a   abc1234 [gone] ...         │ ← 普通分支
│ +feature-b  def5678 [gone] ...          │ ← 有工作树（'+' 前缀）
└─────────────────────────────────────────┘

处理流程：
1. 识别 '+' 前缀
2. 查找对应的工作树路径
3. 先删除工作树
4. 再删除分支

这样可以避免：
❌ 直接删除分支导致工作树残留
❌ 工作树占用磁盘空间
❌ Git 引用混乱
```

---

## 📊 五、三个命令对比

| 特性 | /commit | /commit-push-pr | /clean_gone |
|------|---------|-----------------|-------------|
| **主要功能** | 提交代码 | 完整工作流 | 清理分支 |
| **适用阶段** | 开发中 | 功能完成 | 维护阶段 |
| **涉及远程** | 否 | 是 | 否 |
| **创建 PR** | 否 | 是 | 否 |
| **处理工作树** | 否 | 否 | 是 |
| **前置条件** | 有变更 | gh CLI + origin | 有 [gone] 分支 |

---

## 🔗 六、与其他模块的关联

```
commit-commands 插件
    │
    ├──► Git 工作树
    │    └── /clean_gone 处理工作树
    │
    ├──► Hook 系统
    │    └── PreToolUse 检查敏感文件
    │
    ├──► Agent 协作系统
    │    └── Agent 完成任务后自动提交
    │
    └──► 七阶段流程
         └── Phase 5 实现阶段的提交
```

---

## 🛠️ 七、实战场景

### 场景 1：日常开发流程

```bash
# 1. 开始开发
git checkout -b feature-new

# 2. 编写代码...

# 3. 提交变更
/commit

# 4. 继续开发...

# 5. 再次提交
/commit

# 6. 功能完成，创建 PR
/commit-push-pr
```

### 场景 2：紧急修复流程

```bash
# 1. 在工作树中创建热修复
git worktree add ../hotfix -b hotfix-123 main

# 2. 修复代码...

# 3. 提交并创建 PR
/commit-push-pr

# 4. PR 合并后，清理工作树
/clean_gone
```

### 场景 3：定期维护

```bash
# 每周执行一次清理
# 1. 更新远程信息
git fetch --prune

# 2. 清理本地分支
/clean_gone

# 输出示例：
# Processing branch: feature-completed
#   Deleting branch: feature-completed
# Processing branch: hotfix-merged
#   Deleting branch: hotfix-merged
# Cleaned up 2 branches.
```

---

## ⚠️ 八、注意事项

### 8.1 /commit 注意事项

```
✅ 确保有变更需要提交
✅ 检查提交信息是否准确
✅ 避免提交敏感文件

❌ 不要在空仓库中使用
❌ 不要在 detached HEAD 状态使用
```

### 8.2 /commit-push-pr 注意事项

```
✅ 确保 gh CLI 已安装并登录
✅ 确保有 origin 远程
✅ 检查分支命名是否合适

❌ 不要在 main 分支直接推送
❌ 不要跳过代码审查
```

### 8.3 /clean_gone 注意事项

```
✅ 先执行 git fetch --prune 更新状态
✅ 确认 [gone] 分支确实已合并
✅ 检查是否有未保存的工作树修改

❌ 不要在有待推送提交的分支上使用
❌ 不要清理正在使用的分支
```

---

## 📝 九、学习检查清单

完成本章学习后，请确认：

- [ ] 理解 commit-commands 插件的三个命令
- [ ] 能使用 `/commit` 自动提交代码
- [ ] 能使用 `/commit-push-pr` 创建 PR
- [ ] 能使用 `/clean_gone` 清理分支
- [ ] 理解命令的内部实现原理
- [ ] 了解命令的前置要求
- [ ] 掌握命令的使用时机

---

## 📚 延伸阅读

- [GitHub CLI 官方文档](https://cli.github.com/manual/)
- [约定式提交规范](https://www.conventionalcommits.org/)
- 本项目 `plugins/commit-commands/README.md`

---

**下一章：** [Worktree Hook 事件详解](./03-Worktree%20Hook%20事件详解.md)

---

**最后更新：** 2026 年 4 月 6 日  
**维护者：** Claude Code 项目学习文档团队
