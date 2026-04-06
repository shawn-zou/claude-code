# Git 工作流最佳实践

## 📚 本章概述

本章将结合 Claude Code 项目的实际代码，总结 Git 工作树和工作流的最佳实践，帮助你在实际项目中高效地使用这些功能。

---

## 🎯 学习目标

完成本章学习后，你将能够：

- ✅ 掌握高效的 Git 工作流模式
- ✅ 理解工作树在团队协作中的应用
- ✅ 学会结合 Claude Code 优化 Git 操作
- ✅ 避免 Git 工作流中的常见陷阱

---

## 🌊 一、推荐的工作流模式

### 1.1 功能分支工作流 + 工作树

```
┌─────────────────────────────────────────────────────────────┐
│              功能分支工作流 + 工作树模式                       │
└─────────────────────────────────────────────────────────────┘

主仓库 (main)
    │
    ├──► 工作树 1: feature-login
    │    └── 开发登录功能
    │
    ├──► 工作树 2: feature-payment
    │    └── 开发支付功能
    │
    └──► 工作树 3: hotfix-123
         └── 紧急修复

每个工作树：
├── 独立的开发环境
├── 独立的 IDE 窗口
├── 独立的依赖安装
└── 完成后合并并清理
```

### 1.2 完整工作流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    完整 Git 工作流程                          │
└─────────────────────────────────────────────────────────────┘

开始
  │
  ▼
┌─────────────────┐
│  从 main 创建    │
│  功能分支        │
│  git checkout -b │
│  feature-xxx     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  创建工作树      │
│  git worktree   │
│  add ../feature │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  在工作树中开发  │
│  • 编写代码      │
│  • 运行测试      │
│  • 本地验证      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  使用 /commit   │
│  提交变更        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  功能完成？      │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
   否        是
    │         │
    │         ▼
    │  ┌─────────────────┐
    │  │  使用            │
    │  │  /commit-push-pr │
    │  └────────┬────────┘
    │           │
    │           ▼
    │  ┌─────────────────┐
    │  │  PR 审查通过     │
    │  │  合并到 main     │
    │  └────────┬────────┘
    │           │
    │           ▼
    │  ┌─────────────────┐
    │  │  清理工作树      │
    │  │  /clean_gone     │
    │  └────────┬────────┘
    │           │
    └───────────┤
                │
                ▼
           继续开发
```

---

## 🔧 二、Claude Code 集成最佳实践

### 2.1 使用 --worktree 进行隔离开发

```bash
# 场景：需要执行可能有大改动的任务

# 步骤 1：使用 --worktree 启动
claude --worktree

# Claude 会：
# 1. 创建隔离的工作树
# 2. 在隔离环境中执行任务
# 3. 完成后自动清理

# 步骤 2：审查变更
# Claude 会展示所有变更

# 步骤 3：确认合并
# 如果满意，Claude 会引导你合并到主分支
```

### 2.2 Agent 隔离配置

```yaml
# 在 Agent 定义中启用工作树隔离
---
name: risky-refactor-agent
description: 执行可能有风险的重构任务
isolation: worktree    # 启用工作树隔离
tools: Glob, Read, Edit, Bash
---

你的任务是执行代码重构...

## 安全措施
- 你在隔离的工作树中运行
- 所有修改不会影响主分支
- 完成后需要用户确认才能合并
```

### 2.3 结合七阶段流程

```
七阶段流程 + Git 工作流集成：

Phase 1: Discovery
├── 在主工作树中进行需求分析
└── 创建功能分支

Phase 2: Exploration
├── 使用 code-explorer Agent
└── 可在隔离工作树中运行

Phase 3: Clarification
├── 与用户确认需求
└── 更新功能分支

Phase 4: Architecture
├── 使用 code-architect Agent
└── 设计架构方案

Phase 5: Implementation
├── 使用 --worktree 隔离开发
├── 定期 /commit 提交
└── 保持提交历史整洁

Phase 6: Review
├── 使用 code-reviewer Agent
└── 修复问题后提交

Phase 7: Summary
├── 使用 /commit-push-pr 创建 PR
└── 清理工作树
```

---

## 📋 三、日常工作流清单

### 3.1 开始新功能

```bash
# 清单：开始新功能开发

# 1. 确保主分支是最新的
git checkout main
git pull origin main

# 2. 创建功能分支
git checkout -b feature-xxx

# 3. 创建工作树（可选，推荐用于大功能）
git worktree add ../project-feature-xxx feature-xxx

# 4. 在工作树中安装依赖
cd ../project-feature-xxx
npm install  # 或其他包管理器

# 5. 启动 Claude Code
claude
```

### 3.2 日常提交

```bash
# 清单：日常提交

# 1. 查看当前状态
git status

# 2. 使用 /commit 提交
/commit

# 3. 验证提交
git log -1

# 4. 推送到远程（可选）
git push origin feature-xxx
```

### 3.3 完成功能

```bash
# 清单：完成功能开发

# 1. 确保所有测试通过
npm test

# 2. 使用 /commit-push-pr
/commit-push-pr

# 3. 等待 PR 审查

# 4. PR 合并后清理
git fetch --prune
/clean_gone
```

### 3.4 定期维护

```bash
# 清单：每周维护

# 1. 更新主分支
git checkout main
git pull origin main

# 2. 清理陈旧分支
git fetch --prune
/clean_gone

# 3. 清理工作树
git worktree prune -v

# 4. 清理本地缓存
npm cache clean --force  # 或其他清理命令
```

---

## ⚠️ 四、常见陷阱与解决方案

### 4.1 陷阱：忘记清理工作树

```
问题：
├── 创建了很多工作树
├── 完成后忘记删除
├── 磁盘空间被占用
└── 分支列表混乱

解决方案：
# 定期检查工作树
git worktree list

# 定期清理
/clean_gone

# 设置提醒
# 在 .claude/rules/ 中添加规则：
---
description: 提醒清理工作树
triggers:
  - pattern: "工作树|worktree"
    action: remind
    message: "记得清理不再使用的工作树！使用 /clean_gone 命令"
---
```

### 4.2 陷阱：在错误的分支工作

```
问题：
├── 在 main 分支直接修改
├── 忘记切换到功能分支
└── 提交到错误的分支

解决方案：
# 使用 Hook 防止
{
  "hooks": {
    "PreToolUse": [
      {
        "type": "prompt",
        "tool": "Bash",
        "command_contains": ["git commit", "git push"],
        "prompt": "请确认当前分支是否正确。当前分支：${currentBranch}"
      }
    ]
  }
}

# 或使用 CLAUDE.md 规范
## Git 规范
- 禁止直接在 main 分支提交
- 所有功能开发必须在功能分支进行
- 使用 /commit 命令自动提交
```

### 4.3 陷阱：工作树之间的依赖冲突

```
问题：
├── 多个工作树共享 node_modules
├── 版本冲突
└── 运行时错误

解决方案：
# 每个工作树独立安装依赖
cd /path/to/worktree
npm install

# 使用不同的包管理器缓存
npm config set cache /tmp/npm-cache-worktree-xxx

# 或使用 pnpm 的隔离模式
pnpm install --no-hoist
```

### 4.4 陷阱：敏感信息泄露

```
问题：
├── 提交了 .env 文件
├── 提交了密钥
└── 敏感信息进入版本历史

解决方案：
# 1. 使用 .gitignore
# .gitignore
.env
.env.local
*.pem
credentials.json

# 2. 使用 Hook 检查
{
  "hooks": {
    "PreToolUse": [
      {
        "type": "prompt",
        "tool": "Write",
        "file_pattern": ["**/.env", "**/credentials.*", "**/*.pem"],
        "prompt": "警告：你正在创建可能包含敏感信息的文件。请确认这是否必要。"
      }
    ]
  }
}

# 3. 使用 /commit 时自动检查
# commit.md 中已内置敏感文件检查
```

---

## 🏆 五、高级技巧

### 5.1 并行开发多个功能

```bash
# 场景：同时开发多个独立功能

# 1. 创建主工作树（已存在）
cd /projects/my-app

# 2. 为每个功能创建工作树
git worktree add ../my-app-auth feature-auth
git worktree add ../my-app-api feature-api
git worktree add ../my-app-ui feature-ui

# 3. 在各自的 IDE 中打开
code ../my-app-auth
code ../my-app-api
code ../my-app-ui

# 4. 并行开发
# 在 auth 工作树中开发认证功能
# 在 api 工作树中开发 API
# 在 ui 工作树中开发界面

# 5. 分别提交和创建 PR
cd ../my-app-auth && /commit-push-pr
cd ../my-app-api && /commit-push-pr
cd ../my-app-ui && /commit-push-pr

# 6. 清理
/clean_gone
```

### 5.2 快速切换上下文

```bash
# 场景：正在开发功能，突然需要修复紧急 bug

# 1. 当前在功能开发中
cd /projects/my-app-feature

# 2. 创建热修复工作树
git worktree add ../my-app-hotfix -b hotfix-999 main

# 3. 切换到热修复
cd ../my-app-hotfix

# 4. 修复 bug
# ... 编写修复代码 ...

# 5. 提交并创建 PR
/commit-push-pr

# 6. 返回功能开发
cd ../my-app-feature

# 7. 继续开发，无需 stash
```

### 5.3 大型 Monorepo 优化

```json
// settings.json
{
  "worktree": {
    "sparsePaths": [
      "packages/core/",
      "packages/utils/",
      "shared/types/"
    ]
  }
}
```

```
优势：
├── 只检出需要的目录
├── 减少磁盘占用
├── 加快工作树创建速度
└── 提高搜索效率
```

### 5.4 自动化工作流脚本

```bash
#!/bin/bash
# scripts/start-feature.sh
# 快速开始新功能开发

FEATURE_NAME="${1:-}"
if [ -z "$FEATURE_NAME" ]; then
  echo "Usage: ./start-feature.sh <feature-name>"
  exit 1
fi

BRANCH_NAME="feature-$FEATURE_NAME"
WORKTREE_PATH="../$(basename $(pwd))-$FEATURE_NAME"

echo "Creating feature: $FEATURE_NAME"

# 1. 更新主分支
git checkout main
git pull origin main

# 2. 创建分支和工作树
git worktree add -b "$BRANCH_NAME" "$WORKTREE_PATH" main

# 3. 在工作树中初始化
cd "$WORKTREE_PATH"
npm install

# 4. 打开 IDE
code .

echo "Feature $FEATURE_NAME is ready!"
echo "Worktree: $WORKTREE_PATH"
echo "Branch: $BRANCH_NAME"
```

---

## 📊 六、团队协作建议

### 6.1 命名规范

```
分支命名：
├── feature/xxx    - 新功能
├── fix/xxx        - Bug 修复
├── hotfix/xxx     - 紧急修复
├── refactor/xxx   - 重构
└── docs/xxx       - 文档更新

工作树命名：
├── project-feature-xxx
├── project-hotfix-xxx
└── project-refactor-xxx

提交信息：
├── feat: 添加新功能
├── fix: 修复 Bug
├── docs: 更新文档
├── refactor: 重构代码
└── test: 添加测试
```

### 6.2 工作树共享规则

```
团队规则：
├── 工作树放在统一目录（如 ~/worktrees/）
├── 使用描述性名称
├── 定期清理（每周）
├── 不提交工作树目录到版本控制
└── 共享 CLAUDE.md 规范
```

### 6.3 代码审查流程

```
PR 审查流程：
┌─────────────────────────────────────────┐
│  1. 开发者创建 PR                        │
│     /commit-push-pr                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  2. 审查者检出 PR 分支                    │
│     gh pr checkout 123                   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  3. 在工作树中测试                        │
│     git worktree add ../pr-123 pr-branch │
│     cd ../pr-123 && npm test             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  4. 审查通过，合并                        │
│     gh pr merge 123                      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  5. 清理                                 │
│     /clean_gone                          │
└─────────────────────────────────────────┘
```

---

## 📝 七、学习检查清单

完成本章学习后，请确认：

- [ ] 掌握功能分支工作流 + 工作树模式
- [ ] 能使用 --worktree 进行隔离开发
- [ ] 能配置 Agent 的工作树隔离
- [ ] 掌握日常工作流清单
- [ ] 了解常见陷阱及解决方案
- [ ] 掌握高级技巧
- [ ] 了解团队协作建议

---

## 📚 八、模块总结

### 8.1 Git 工作树模块完整知识体系

```
09-Git 工作树模块
│
├── 01-Git 工作树基础详解
│   ├── 工作树概念
│   ├── git worktree 命令
│   ├── Claude Code --worktree 参数
│   └── Agent isolation 配置
│
├── 02-commit-commands 插件详解
│   ├── /commit 命令
│   ├── /commit-push-pr 命令
│   └── /clean_gone 命令
│
├── 03-Worktree Hook 事件详解
│   ├── WorktreeCreate Hook
│   └── WorktreeRemove Hook
│
└── 04-Git 工作流最佳实践
    ├── 推荐工作流模式
    ├── Claude Code 集成
    ├── 常见陷阱
    └── 团队协作建议
```

### 8.2 核心要点回顾

| 知识点 | 核心内容 |
|--------|----------|
| **工作树概念** | 同时在多个分支工作，无需切换 |
| **核心命令** | `git worktree add/list/remove/prune` |
| **Claude 集成** | `--worktree` 参数，`isolation: worktree` |
| **自动化提交** | `/commit`，`/commit-push-pr`，`/clean_gone` |
| **Hook 事件** | `WorktreeCreate`，`WorktreeRemove` |
| **最佳实践** | 功能分支 + 工作树，定期清理 |

---

## 🔗 九、与其他模块的关联

```
Git 工作树模块
    │
    ├──► 02-Hook 系统
    │    └── WorktreeCreate/Remove Hook
    │
    ├──► 04-Agent 协作系统
    │    └── isolation: worktree 配置
    │
    ├──► 05-七阶段流程
    │    └── Phase 5 实现阶段的提交
    │
    ├──► 07-插件系统
    │    └── commit-commands 插件
    │
    └──► 12-学习次序指南
         └── Week 8: Git 工作树
```

---

## 📚 延伸阅读

- [Git 官方文档](https://git-scm.com/doc)
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
- [约定式提交](https://www.conventionalcommits.org/)
- 本项目 `plugins/commit-commands/` 目录

---

**恭喜你完成了 Git 工作树模块的学习！** 🎉

**建议下一步：** 学习 [提示词工程与质量保障](../10-提示词工程与质量保障/) 模块

---

**最后更新：** 2026 年 4 月 6 日  
**维护者：** Claude Code 项目学习文档团队
