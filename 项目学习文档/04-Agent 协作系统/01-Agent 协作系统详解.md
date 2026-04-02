# Agent 协作系统详解

## 一、什么是 Agent 协作系统

### 1.1 基本概念

**Agent（智能体）** 是一个具有特定能力的 AI 助手。想象一下在一个公司里：
- 有专门做代码审查的工程师
- 有专门做架构设计的设计师
- 有专门做测试的 QA 工程师

每个角色都有自己的专长，他们一起协作完成复杂的项目。

在 Claude Code 中，**Agent 协作系统**就是让多个专门的 AI 助手一起工作，完成复杂的任务。

### 1.2 为什么需要 Agent 协作

**单一 AI 的局限：**
```
用户：帮我开发一个完整的用户认证系统

问题：
- 任务太复杂，容易遗漏细节
- 需要考虑的方面太多（安全、性能、可维护性）
- 一个人工智能难以兼顾所有角度
```

**多 Agent 协作的优势：**
```
用户：帮我开发一个完整的用户认证系统

协作流程：
1. code-explorer Agent → 分析现有代码库
2. code-architect Agent → 设计架构方案  
3. coding Agent → 实现功能
4. code-reviewer Agent → 审查代码质量
5. security-agent Agent → 检查安全性

结果：更全面、更专业、更少遗漏
```

---

## 二、Agent 的类型与职责

项目中有多种专门的 Agent，每种都有明确的职责分工：

### 2.1 code-explorer（代码探索者）

**职责：** 深入分析现有代码库，理解功能和架构。

**触发场景：**
- 需要了解某个功能如何实现
- 需要找到相关代码的位置
- 需要理解代码的执行流程

**典型输出：**
```markdown
## 代码探索报告：用户认证流程

### 入口点
- `src/auth/login.ts:15` - login 函数

### 执行流程
1. 接收用户名密码 → 
2. 验证格式 → 
3. 查询数据库 → 
4. 比对密码哈希 → 
5. 生成 JWT token

### 关键组件
- AuthService: 核心认证逻辑
- JwtService: token 生成和验证
- UserRepository: 数据访问层

### 需要了解的文件
- src/auth/AuthService.ts
- src/middleware/authMiddleware.ts
- src/config/security.ts
```

### 2.2 code-architect（代码架构师）

**职责：** 设计架构方案，提供多种实现思路。

**触发场景：**
- 需要设计新功能
- 需要重构现有代码
- 需要在多个方案中选择

**典型输出：**
```markdown
## 架构设计方案

### 方案 1：最小改动
**思路：** 扩展现有的 AuthService
**优点：** 快速、低风险
**缺点：** 耦合度高

### 方案 2：全新设计
**思路：** 独立的 OAuth 服务
**优点：** 清晰分离、易测试
**缺点：** 工作量大

### 方案 3：折中方案（推荐）
**思路：** OAuthProvider 抽象层
**优点：** 平衡速度和清晰度
**缺点：** 仍有部分耦合

### 推荐方案：方案 3
理由：符合项目现状，既能快速上线又保持良好架构
```

### 2.3 code-reviewer（代码审查员）

**职责：** 审查代码质量，发现问题并提出改进建议。

**触发场景：**
- 写完代码后需要审查
- 提交 PR 前检查
- 发现代码有问题需要诊断

**典型输出：**
```markdown
## 代码审查报告

### 严重问题（必须修复）
1. **SQL 注入风险** - `src/user.ts:45`
   - 问题：字符串拼接 SQL
   - 修复：使用参数化查询

### 重要问题（建议修复）
1. **缺少错误处理** - `src/auth.ts:67`
   - 问题：try-catch 块为空
   - 建议：添加适当的错误处理

### 轻微问题（可选优化）
1. **变量命名不清晰** - `src/utils.ts:23`
   - 问题：变量名 a, b, c 含义不明
   - 建议：使用描述性名称

### 优点
✅ 良好的模块化设计
✅ 充分的注释说明
```

### 2.4 专用领域 Agent

#### pr-test-analyzer（PR 测试分析）
**专长：** 分析 PR 的测试覆盖率和质量

#### silent-failure-hunter（静默失败猎手）
**专长：** 查找错误处理不当的静默失败

#### comment-analyzer（注释分析）
**专长：** 检查代码注释的准确性和完整性

#### type-design-analyzer（类型设计分析）
**专长：** 分析类型设计的质量

---

## 三、Agent 的配置文件结构

### 3.1 Agent 文件的基本结构

每个 Agent 都是一个 `.md` 文件，包含两个主要部分：

```markdown
---
name: agent-name
description: 描述何时使用这个 Agent
model: inherit
color: blue
tools: ["Read", "Grep"]
---

你是 [角色描述]...

**你的核心职责：**
1. ...
2. ...

**工作流程：**
1. ...
2. ...

**输出格式：**
...
```

### 3.2 Frontmatter 字段详解

**name（必需）**
```yaml
name: code-reviewer
```
- 小写字母和连字符
- 3-50 个字符
- 全局唯一

**description（必需）**
```yaml
description: |
  当用户写完代码需要审查时使用此 Agent。
  
  示例：
  <example>
  用户："我写完了代码，帮我检查一下"
  助手："我来用 code-reviewer Agent 审查代码"
  </example>
```
- 包含触发场景
- 包含使用示例
- 示例用 `<example>` 标签包裹

**model（必需）**
```yaml
model: inherit  # 使用继承的模型设置
# 或
model: sonnet   # 指定使用 Sonnet
# 或
model: opus     # 指定使用 Opus
```

**color（必需）**
```yaml
color: blue     # 蓝色
color: green    # 绿色
color: cyan     # 青色
```
- 用于 UI 显示区分不同 Agent

**tools（可选）**
```yaml
tools: ["Read", "Grep", "Glob"]
```
- 授予 Agent 使用的工具权限
- 遵循最小权限原则（只给必要的工具）

### 3.3 System Prompt 编写

System Prompt 是 Agent 的核心，定义了 Agent 的行为模式。

**基本结构：**
```markdown
你是 [角色定位]，专注于 [专业领域]。

**你的核心职责：**
1. 职责 1
2. 职责 2
3. 职责 3

**工作流程：**
1. **第一步**：做什么，怎么做
2. **第二步**：做什么，怎么做
3. **第三步**：做什么，怎么做

**质量标准：**
- 标准 1
- 标准 2

**输出格式：**
按照以下格式输出结果：
## 报告标题

### 摘要
[简要总结]

### 发现的问题
1. [问题 1]
2. [问题 2]

### 建议
[具体建议]
```

**优秀示例：**
```markdown
你是代码质量审查专家，专注于识别代码中的问题并给出改进建议。

**你的核心职责：**
1. 分析代码质量问题（可读性、可维护性、复杂度）
2. 识别安全漏洞（SQL 注入、XSS、认证问题）
3. 检查是否符合项目最佳实践
4. 提供具体、可操作的反馈

**审查流程：**
1. **收集上下文**：使用 Glob 找到最近修改的文件
2. **阅读代码**：使用 Read 工具查看变更内容
3. **质量分析**：
   - 检查代码重复（DRY 原则）
   - 评估复杂度和可读性
   - 验证错误处理
   - 检查日志记录
4. **安全分析**：
   - 扫描注入漏洞
   - 检查认证授权
   - 查找硬编码密钥
5. **分类问题**：按严重程度分组（严重/重要/轻微）
6. **生成报告**：按模板格式化输出

**质量标准：**
- 每个问题都包含文件路径和行号（如 `src/auth.ts:42`）
- 问题按严重程度分类并有明确标准
- 建议具体且可操作
- 包含代码示例说明如何修复

**输出格式：**
## 代码审查报告

### 严重问题（必须修复）
1. **[问题类型]** - `文件路径：行号`
   - 问题描述
   - 修复建议

### 重要问题（建议修复）
...

### 轻微问题（可选优化）
...

### 做得好的地方
✅ [优点 1]
✅ [优点 2]
```

---

## 四、Agent 的触发机制

### 4.1 显式触发

用户明确要求使用某个 Agent：

```
用户："用 code-reviewer Agent 审查这段代码"
→ Claude 启动 code-reviewer Agent
```

### 4.2 隐式触发

Claude 根据对话内容自动判断使用哪个 Agent：

```
用户："我写完了支付功能，帮我检查一下有没有问题"
→ Claude 分析：
  - "写完了代码" → 需要审查
  - "检查问题" → 质量检测
  → 自动启动 code-reviewer Agent
```

### 4.3 主动触发

Claude 主动建议使用 Agent：

```
用户："我实现了用户登录功能"
Claude："很好！我建议用 code-reviewer Agent 审查一下实现，确保没有安全漏洞。要我现在审查吗？"
```

### 4.4 触发条件设计

在 description 中定义清晰的触发条件：

**不好的例子：**
```yaml
description: 用于审查代码
```
❌ 太模糊，不容易触发

**好的例子：**
```yaml
description: |
  当用户写完代码需要质量审查、安全检查或最佳实践验证时使用此 Agent。
  
  示例：
  <example>
  用户："我刚写完了认证模块"
  助手："让我用 code-reviewer 审查一下"
  </example>
  
  <example>
  用户："检查这段代码有没有安全问题"
  助手："我来用 code-reviewer 进行安全分析"
  </example>
```
✅ 具体明确，容易触发

---

## 五、实战案例：七阶段流程中的 Agent 协作

以 feature-dev 插件的七阶段流程为例，看看多个 Agent 如何协作：

### 阶段 1：Discovery（发现）
**参与 Agent：** 无（主 Claude 处理）
**任务：** 理解需求，澄清模糊点

### 阶段 2：Codebase Exploration（代码库探索）
**参与 Agent：** 2-3 个 code-explorer Agent（并行）

```
Agent A: 查找类似功能
  → 找到用户认证的实现
  
Agent B: 分析架构模式
  → 发现使用 MVC 架构
  
Agent C: 研究技术栈
  → 确认使用 Express + MongoDB
```

**主 Claude 整合结果：**
```markdown
## 代码库分析结果

### 类似功能
- 用户认证（src/auth/）：JWT token，中间件模式
- 会话管理（src/session/）：Redis 后端，24 小时过期

### 架构模式
- MVC 分层：Model（数据）、View（展示）、Controller（逻辑）
- 中间件模式：用于认证、日志等横切关注点

### 需要了解的关键文件
- src/auth/AuthService.ts:45 - 核心认证逻辑
- src/middleware/authMiddleware.ts:12 - 请求认证
- src/config/security.ts:8 - 安全配置
```

### 阶段 3：Clarifying Questions（澄清问题）
**参与 Agent：** 无（主 Claude 处理）
**任务：** 提出并回答澄清问题

### 阶段 4：Architecture Design（架构设计）
**参与 Agent：** 2-3 个 code-architect Agent（并行）

```
Agent A（最小改动）:
  → 扩展现有 AuthService
  
Agent B（清晰架构）:
  → 新建独立 OAuth 服务
  
Agent C（实用平衡）:
  → OAuthProvider 抽象层
```

**主 Claude 整合并给出建议：**
```markdown
## 架构方案对比

### 方案 1：最小改动
扩展 AuthService，添加 OAuth 方法
- 优点：快，风险低
- 缺点：耦合

### 方案 2：清晰架构
独立 OAuth 服务
- 优点：清晰，易测试
- 缺点：工作量大

### 方案 3：实用平衡（推荐）
OAuthProvider 抽象
- 优点：平衡速度和清晰度
- 缺点：部分耦合

推荐方案 3，理由是...
```

### 阶段 5：Implementation（实现）
**参与 Agent：** 无（主 Claude 处理）
**任务：** 根据选定的方案实现功能

### 阶段 6：Quality Review（质量审查）
**参与 Agent：** 3 个 code-reviewer Agent（并行）

```
Agent A（简洁/DRY/优雅）:
  → 发现代码重复问题
  
Agent B（Bug/正确性）:
  → 发现内存泄漏风险
  
Agent C（规范/抽象）:
  → 确认符合项目规范
```

**主 Claude 整合结果：**
```markdown
## 质量审查结果

### 高优先级
1. 缺少错误处理（src/oauth.ts:67）
2. 内存泄漏：OAuth state 未清理（src/oauth.ts:89）

### 中优先级
1. 可简化 token 刷新逻辑（src/oauth.ts:120）

所有测试通过，代码符合项目规范。
```

### 阶段 7：Summary（总结）
**参与 Agent：** 无（主 Claude 处理）
**任务：** 总结完成的工作

---

## 六、Agent 协作的优势

### 6.1 并行处理

多个 Agent 可以同时工作，大幅缩短时间：

```
串行方式（传统）:
探索代码 (10 分钟) → 设计架构 (10 分钟) → 审查代码 (10 分钟)
总计：30 分钟

并行方式（多 Agent）:
[Agent A 探索] [Agent B 设计] [Agent C 审查] (同时进行)
总计：10-15 分钟
```

### 6.2 专业分工

每个 Agent 专注于自己最擅长的领域：

```
code-explorer → 擅长理解现有代码
code-architect → 擅长设计架构
code-reviewer → 擅长发现问题

专业的人做专业的事 = 更好的结果
```

### 6.3 多角度审视

同一个问题从不同角度分析：

```
代码审查场景:
- Agent A 关注代码质量（是否简洁、优雅）
- Agent B 关注功能正确性（是否有 Bug）
- Agent C 关注规范符合度（是否遵守规则）

三个角度 = 更全面的检查
```

### 6.4 知识积累

每个 Agent 都可以有自己的专业知识库：

```
security-agent:
  - OWASP Top 10
  - 安全编码规范
  - 加密算法知识

pr-test-analyzer:
  - 测试方法论
  - 覆盖率标准
  - 测试用例设计

专业化 = 更深入的知识
```

---

## 七、创建自定义 Agent

### 7.1 确定 Agent 定位

问自己几个问题：
1. **解决什么问题？** （需求分析）
2. **目标用户是谁？** （使用场景）
3. **有什么专长？** （差异化）

**示例：创建一个"性能优化 Agent"**

```
需求：开发者经常写出性能不佳的代码
用户：需要优化代码性能的开发者
专长：性能分析、瓶颈识别、优化技巧
```

### 7.2 编写 Agent 配置

**步骤 1：创建文件**
```bash
touch agents/performance-optimizer.md
```

**步骤 2：编写 Frontmatter**
```yaml
---
name: performance-optimizer
description: |
  当用户需要分析和优化代码性能时使用此 Agent。
  
  示例：
  <example>
  用户："这段代码运行很慢，帮我优化一下"
  助手："我来用 performance-optimizer 分析性能瓶颈"
  </example>
  
  <example>
  用户："如何提升这个函数的性能？"
  助手："让我用 performance-optimizer 找出优化点"
  </example>
model: inherit
color: orange
tools: ["Read", "Grep", "Glob"]
---
```

**步骤 3：编写 System Prompt**
```markdown
你是性能优化专家，专注于分析和提升代码的执行效率。

**你的核心职责：**
1. 分析代码性能瓶颈（时间复杂度、空间复杂度）
2. 识别低效的编程模式（不必要的循环、重复计算等）
3. 提供具体的性能优化建议
4. 解释优化原理和预期收益

**分析流程：**
1. **收集代码**：获取需要分析的代码段
2. **静态分析**：
   - 识别循环嵌套（O(n²) 及以上）
   - 查找重复计算
   - 发现不必要的内存分配
   - 检查数据结构选择
3. **动态分析**（如果可能）：
   - 热点函数识别
   - 内存使用情况
   - I/O 操作频率
4. **优化设计**：
   - 算法优化（更好的时间复杂度）
   - 数据结构优化（更高效的数据组织）
   - 缓存优化（避免重复计算）
   - 并发优化（利用多核）
5. **生成报告**：按优先级列出优化建议

**质量标准：**
- 每个建议都包含具体的代码示例
- 说明优化的理论依据
- 给出预期的性能提升幅度
- 考虑优化带来的其他影响（如内存换时间）

**输出格式：**
## 性能分析报告

### 当前性能状况
[总体评估]

### 关键瓶颈（优先优化）
1. **[问题名称]** - `文件：行号`
   - **问题描述**：为什么慢
   - **影响程度**：高/中/低
   - **优化建议**：具体怎么做
   - **预期收益**：提升多少

### 次要优化点
...

### 优化示例
[优化前后的代码对比]

### 其他建议
[架构层面、算法层面的建议]
```

### 7.3 测试 Agent

**测试步骤：**

1. **准备测试用例**
```javascript
// 低效代码示例
function findDuplicates(arr) {
    const duplicates = [];
    for (let i = 0; i < arr.length; i++) {
        for (let j = i + 1; j < arr.length; j++) {
            if (arr[i] === arr[j]) {
                duplicates.push(arr[i]);
            }
        }
    }
    return duplicates;
}
```

2. **触发 Agent**
```
用户："分析这个函数的性能"
```

3. **验证输出**
```markdown
## 性能分析报告

### 当前性能状况
时间复杂度：O(n²)，空间复杂度：O(k)（k 为重复元素数量）

### 关键瓶颈
1. **嵌套循环** - 双重循环导致平方级复杂度
   - 影响程度：高
   - 优化建议：使用 HashSet，降至 O(n)
   
### 优化示例
// 优化前：O(n²)
for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
        // ...
    }
}

// 优化后：O(n)
const seen = new Set();
const duplicates = new Set();
for (const item of arr) {
    if (seen.has(item)) {
        duplicates.add(item);
    } else {
        seen.add(item);
    }
}

### 预期收益
从 O(n²) 提升至 O(n)，对于 10000 个元素的数组：
- 优化前：约 1 亿次比较
- 优化后：约 10000 次查找
- 速度提升：约 10000 倍
```

---

## 八、Agent 设计的最佳实践

### 8.1 设计原则

**SOLID 原则在 Agent 设计中的应用：**

**S - Single Responsibility（单一职责）**
```
✅ 好：code-reviewer 只负责代码审查
❌ 坏：一个 Agent 既负责审查又负责测试还负责部署
```

**I - Interface Segregation（接口隔离）**
```
✅ 好：每个 Agent 有清晰的输入输出
❌ 坏：Agent 之间耦合严重
```

### 8.2 命名规范

**好的命名：**
```
performance-optimizer      ✅ 清晰表达用途
code-reviewer              ✅ 一目了然
security-scanner           ✅ 专业领域明确
```

**坏的命名：**
```
agent1                     ❌ 毫无意义
helper                     ❌ 太宽泛
super-agent                ❌ 夸大其词
```

### 8.3 触发词设计

**有效的触发词：**
```
- "审查代码" → code-reviewer
- "性能优化" → performance-optimizer
- "安全检查" → security-agent
- "设计架构" → code-architect
```

**无效的触发词：**
```
- "看一下" → 太模糊
- "处理一下" → 不明确
- "帮我" → 缺少具体内容
```

### 8.4 工具权限管理

**最小权限原则：**
```yaml
# code-reviewer 只需要读权限
tools: ["Read", "Grep", "Glob"]

# 不需要写权限，因为它只审查不修改
# 这样即使被攻击也不会造成破坏
```

---

### 8.5 工具权限管理

**最小权限原则：**
```yaml
# code-reviewer 只需要读权限
tools: ["Read", "Grep", "Glob"]

# 不需要写权限，因为它只审查不修改
# 这样即使被攻击也不会造成破坏
```

---

## 九、核心 Agent 深度解析

> 本节将逐行解析实际项目中的 Agent 文件，帮助你深入理解 Agent 的设计细节。

### 9.1 feature-dev 的三个核心 Agent

#### 9.1.1 code-explorer：代码探索者

**文件位置：** `plugins/feature-dev/agents/code-explorer.md`

**完整配置解读：**

```yaml
---
name: code-explorer
description: Deep analyzes existing codebase features by tracing execution paths, 
             mapping architecture layers, understanding patterns and abstractions, 
             and documenting dependencies to inform new development
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch, KillShell, BashOutput
model: sonnet
color: yellow
---
```

**字段详解：**

| 字段 | 值 | 为什么这样选择 |
|------|-----|---------------|
| **name** | `code-explorer` | 清晰表达职责：探索代码库 |
| **description** | 详细描述了探索执行路径、映射架构层等 | 让 Claude 知道何时触发此 Agent |
| **tools** | 10 个工具 | 需要多种工具来全面分析代码 |
| **model** | `sonnet` | Sonnet 模型在代码理解方面表现优秀 |
| **color** | `yellow` | 黄色代表分析、探索类型的工作 |

**System Prompt 核心结构分析：**

```markdown
You are an expert code analyst specializing in tracing and understanding 
feature implementations across codebases.
```

→ **角色定位**：代码分析专家，专注于追踪和理解功能实现

```markdown
## Core Mission
Provide a complete understanding of how a specific feature works by tracing 
its implementation from entry points to data storage, through all abstraction layers.
```

→ **核心使命**：提供对功能如何工作的完整理解，从入口点到数据存储，穿越所有抽象层

**工作流程设计（第 14-37 行）：**

```markdown
## Analysis Approach

**1. Feature Discovery**        ← 第一步：发现功能
- Find entry points (APIs, UI components, CLI commands)
- Locate core implementation files
- Map feature boundaries and configuration

**2. Code Flow Tracing**        ← 第二步：追踪代码流
- Follow call chains from entry to output
- Trace data transformations at each step
- Identify all dependencies and integrations
- Document state changes and side effects

**3. Architecture Analysis**    ← 第三步：分析架构
- Map abstraction layers (presentation → business logic → data)
- Identify design patterns and architectural decisions
- Document interfaces between components
- Note cross-cutting concerns (auth, logging, caching)

**4. Implementation Details**   ← 第四步：实现细节
- Key algorithms and data structures
- Error handling and edge cases
- Performance considerations
- Technical debt or improvement areas
```

→ **设计亮点**：
1. **四步递进流程**：从发现 → 追踪 → 分析 → 细节，层次清晰
2. **每一步都有具体任务清单**：告诉 Agent 具体要做什么
3. **覆盖全面**：从表层 API 到底层数据结构，无一遗漏

**输出格式设计（第 39-51 行）：**

```markdown
## Output Guidance

Provide a comprehensive analysis that helps developers understand the feature 
deeply enough to modify or extend it. Include:

- Entry points with file:line references          ← 精确到行号
- Step-by-step execution flow with data transformations
- Key components and their responsibilities
- Architecture insights: patterns, layers, design decisions
- Dependencies (external and internal)
- Observations about strengths, issues, or opportunities
- List of files that you think are absolutely essential

Structure your response for maximum clarity and usefulness. 
Always include specific file paths and line numbers.
```

→ **输出要求**：
- ✅ 必须包含文件路径和行号（如 `src/auth.ts:42`）
- ✅ 列出关键组件及其职责
- ✅ 提供架构洞察（模式、层次、设计决策）
- ✅ 给出优势、问题、改进机会的观察
- ✅ 列出必须了解的文件清单

**使用场景示例：**

```
用户：我要开发一个 OAuth 登录功能，需要先了解现有的认证系统

Claude: 好的，让我启动 code-explorer Agent 来分析现有认证系统...

[code-explorer 分析报告]

### 入口点
- `src/auth/login.ts:15` - login 函数
- `src/auth/session.ts:28` - session 管理

### 执行流程
1. 接收用户名密码 → 
2. 验证格式 → 
3. 查询数据库 → 
4. 比对密码哈希 → 
5. 生成 JWT token

### 关键组件
- AuthService: 核心认证逻辑
- JwtService: token 生成和验证
- UserRepository: 数据访问层

### 必须了解的文件
- src/auth/AuthService.ts
- src/middleware/authMiddleware.ts
- src/config/security.ts
```

---

#### 9.1.2 code-architect：代码架构师

**文件位置：** `plugins/feature-dev/agents/code-architect.md`

**完整配置解读：**

```yaml
---
name: code-architect
description: Designs feature architectures by analyzing existing codebase patterns 
             and conventions, then providing comprehensive implementation blueprints 
             with specific files to create/modify, component designs, data flows, 
             and build sequences
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch, KillShell, BashOutput
model: sonnet
color: green
---
```

**与 code-explorer 的对比：**

| 特性 | code-explorer | code-architect |
|------|---------------|----------------|
| **职责** | 理解现有代码 | 设计新架构 |
| **输出** | 分析报告 | 实施蓝图 |
| **时机** | Phase 2（探索） | Phase 4（设计） |
| **颜色** | yellow（分析） | green（创造） |

**System Prompt 核心流程（第 11-34 行）：**

```markdown
## Core Process

**1. Codebase Pattern Analysis**     ← 第一步：分析现有模式
Extract existing patterns, conventions, and architectural decisions. 
Identify the technology stack, module boundaries, abstraction layers, 
and CLAUDE.md guidelines. Find similar features to understand established approaches.

**2. Architecture Design**           ← 第二步：设计架构
Based on patterns found, design the complete feature architecture. 
Make decisive choices - pick one approach and commit. 
Ensure seamless integration with existing code. 
Design for testability, performance, and maintainability.

**3. Complete Implementation Blueprint**  ← 第三步：实施蓝图
Specify every file to create or modify, component responsibilities, 
integration points, and data flow. Break implementation into clear phases with specific tasks.
```

→ **关键设计思想**：
1. **基于现有模式**：不是凭空设计，而是分析现有代码的模式
2. **果断决策**："Make decisive choices - pick one approach and commit"
3. **完整的实施蓝图**：具体到每个文件、每个组件、每个集成点

**输出格式详解（第 22-34 行）：**

```markdown
## Output Guidance

Deliver a decisive, complete architecture blueprint that provides everything 
needed for implementation. Include:

- **Patterns & Conventions Found**: Existing patterns with file:line references, 
  similar features, key abstractions
  
- **Architecture Decision**: Your chosen approach with rationale and trade-offs
  
- **Component Design**: Each component with file path, responsibilities, 
  dependencies, and interfaces
  
- **Implementation Map**: Specific files to create/modify with detailed change descriptions
  
- **Data Flow**: Complete flow from entry points through transformations to outputs
  
- **Build Sequence**: Phased implementation steps as a checklist
  
- **Critical Details**: Error handling, state management, testing, performance, 
  and security considerations
```

→ **输出必须包含的 7 个要素**：
1. 发现的模式和约定（带文件引用）
2. 架构决策及理由和权衡
3. 组件设计（文件路径、职责、依赖、接口）
4. 实施地图（具体要创建/修改的文件）
5. 数据流（从入口到输出的完整流程）
6. 构建序列（分阶段的检查清单）
7. 关键细节（错误处理、状态管理、测试、性能、安全）

**实战输出示例：**

```markdown
## 架构设计方案：OAuth 登录

### Patterns & Conventions Found
- 现有认证使用 JWT token 模式 (`src/auth/AuthService.ts:45`)
- 中间件模式用于请求认证 (`src/middleware/authMiddleware.ts:12`)
- 配置集中在 `src/config/` 目录

### Architecture Decision
**选择方案**：OAuthProvider 抽象层

**理由**：
- 复用现有的 AuthService 基础设施
- 保持清晰的边界便于测试
- 符合项目现有的架构模式

### Component Design

**1. OAuthProvider (新建)**
- 文件：`src/auth/providers/OAuthProvider.ts`
- 职责：定义 OAuth 提供商的通用接口
- 依赖：无
- 接口：`authenticate(code: string): Promise<User>`

**2. GoogleOAuthProvider (新建)**
- 文件：`src/auth/providers/GoogleOAuthProvider.ts`
- 职责：实现 Google OAuth 特定逻辑
- 依赖：OAuthProvider, HttpClient
- 接口：继承 OAuthProvider

**3. AuthController (修改)**
- 文件：`src/controllers/AuthController.ts`
- 变更：添加 `/auth/google/callback` 路由处理

### Implementation Map

| 文件 | 操作 | 描述 |
|------|------|------|
| `src/auth/providers/OAuthProvider.ts` | 新建 | 抽象基类 |
| `src/auth/providers/GoogleOAuthProvider.ts` | 新建 | Google 实现 |
| `src/controllers/AuthController.ts` | 修改 | 添加回调处理 |
| `src/config/oauth.ts` | 新建 | OAuth 配置 |

### Data Flow
```
用户点击"Google 登录"
    ↓
重定向到 Google OAuth
    ↓
用户授权
    ↓
回调 /auth/google/callback?code=xxx
    ↓
GoogleOAuthProvider.exchangeCodeForToken(code)
    ↓
GoogleOAuthProvider.getUserInfo(token)
    ↓
查找或创建本地用户
    ↓
生成 JWT token
    ↓
返回给前端
```

### Build Sequence

- [ ] **Phase 1**: 创建 OAuthProvider 抽象类
- [ ] **Phase 2**: 实现 GoogleOAuthProvider
- [ ] **Phase 3**: 添加 OAuth 配置
- [ ] **Phase 4**: 修改 AuthController 添加路由
- [ ] **Phase 5**: 编写单元测试
- [ ] **Phase 6**: 集成测试

### Critical Details

**错误处理**：
- OAuth 拒绝：抛出 `OAuthError` 并记录日志
- Token 交换失败：重试 3 次后失败
- 用户已存在：直接登录，不创建新用户

**安全性**：
- 验证 state 参数防止 CSRF
- PKCE 增强安全性
- Token 存储在 HttpOnly cookie

**性能**：
- 缓存用户信息 5 分钟
- 异步队列处理用户创建
```

---

#### 9.1.3 code-reviewer：代码审查员

**文件位置：** `plugins/feature-dev/agents/code-reviewer.md`

**核心特色：置信度评分机制**

```yaml
---
name: code-reviewer
description: Reviews code for bugs, logic errors, security vulnerabilities, 
             code quality issues, and adherence to project conventions, 
             using confidence-based filtering to report only high-priority 
             issues that truly matter
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch, KillShell, BashOutput
model: sonnet
color: red
---
```

→ **为什么是红色（red）？** 红色代表关键、安全问题，提醒开发者注意

**置信度评分系统（第 23-33 行）：**

```markdown
## Confidence Scoring

Rate each potential issue on a scale from 0-100:

- **0**: Not confident at all. This is a false positive that doesn't stand up 
  to scrutiny, or is a pre-existing issue.
  
- **25**: Somewhat confident. This might be a real issue, but may also be a 
  false positive. If stylistic, it wasn't explicitly called out in project guidelines.
  
- **50**: Moderately confident. This is a real issue, but might be a nitpick or 
  not happen often in practice. Not very important relative to the rest of the changes.
  
- **75**: Highly confident. Double-checked and verified this is very likely a real 
  issue that will impact functionality. The existing approach is insufficient. 
  Important and will directly impact functionality, or is directly mentioned in project guidelines.
  
- **100**: Absolutely certain. Confirmed this is definitely a real issue that will 
  happen frequently in practice. The evidence directly confirms this.

**Only report issues with confidence ≥ 80.** Focus on quality over quantity.
```

→ **设计原理**：
- **过滤低置信度问题**：避免误报干扰开发者
- **聚焦真正重要的问题**：confidence ≥ 80 才报告
- **质量优于数量**：宁可少报，不可乱报

**审查范围（第 11-21 行）：**

```markdown
## Review Scope

By default, review unstaged changes from `git diff`. 
The user may specify different files or scope to review.

## Core Review Responsibilities

**Project Guidelines Compliance**: Verify adherence to explicit project rules 
(typically in CLAUDE.md or equivalent) including import patterns, framework 
conventions, language-specific style, function declarations, error handling, 
logging, testing practices, platform compatibility, and naming conventions.

**Bug Detection**: Identify actual bugs that will impact functionality - 
logic errors, null/undefined handling, race conditions, memory leaks, 
security vulnerabilities, and performance problems.

**Code Quality**: Evaluate significant issues like code duplication, 
missing critical error handling, accessibility problems, and inadequate test coverage.
```

→ **三大审查维度**：
1. **项目规范遵循**：CLAUDE.md 中的明确规则
2. **Bug 检测**：影响功能的实际问题
3. **代码质量**：重复代码、缺失错误处理等

**输出格式（第 35-46 行）：**

```markdown
## Output Guidance

Start by clearly stating what you're reviewing. For each high-confidence issue, provide:

- Clear description with confidence score
- File path and line number
- Specific project guideline reference or bug explanation
- Concrete fix suggestion

Group issues by severity (Critical vs Important). 
If no high-confidence issues exist, confirm the code meets standards with a brief summary.

Structure your response for maximum actionability - developers should know 
exactly what to fix and why.
```

**实战输出示例：**

```markdown
## 代码审查报告

**审查范围**：本次提交的 3 个文件（git diff）

### 🔴 Critical 问题（confidence ≥ 90）

**1. SQL 注入风险** - `src/user/repository.ts:45` 
   - **Confidence**: 95/100
   - **问题**：字符串拼接 SQL 查询
   ```typescript
   // ❌ 错误写法
   const query = `SELECT * FROM users WHERE id = '${userId}'`;
   ```
   - **修复建议**：使用参数化查询
   ```typescript
   // ✅ 正确写法
   const query = 'SELECT * FROM users WHERE id = ?';
   const params = [userId];
   ```
   - **违反规范**：CLAUDE.md 第 3.2 节 "禁止 SQL 字符串拼接"

### 🟡 Important 问题（confidence 80-89）

**2. 缺少空值检查** - `src/auth/service.ts:67`
   - **Confidence**: 85/100
   - **问题**：可能访问 undefined 对象的属性
   ```typescript
   // ❌ 风险代码
   const token = user.preferences.token;
   ```
   - **修复建议**：添加可选链或默认值
   ```typescript
   // ✅ 安全写法
   const token = user?.preferences?.token ?? null;
   ```

### ✅ 做得好的地方

- 良好的模块化设计
- 充分的错误处理
- 符合项目命名规范

**总结**：发现 2 个需要修复的问题，修复后可达到项目标准。
```

---

### 9.2 pr-review-toolkit 的 6 个专用 Agent

pr-review-toolkit 是一个专门用于 PR 审查的工具包，包含 6 个高度专业化的 Agent。

#### 9.2.1 silent-failure-hunter：静默失败猎手

**文件位置：** `plugins/pr-review-toolkit/agents/silent-failure-hunter.md`

**核心使命（第 8 行）：**

```markdown
You are an elite error handling auditor with zero tolerance for silent failures 
and inadequate error handling.
```

→ **角色定位**：精英级错误处理审计员，对静默失败零容忍

**五大核心原则（第 10-18 行）：**

```markdown
## Core Principles

1. **Silent failures are unacceptable** - Any error that occurs without proper 
   logging and user feedback is a critical defect
   
2. **Users deserve actionable feedback** - Every error message must tell users 
   what went wrong and what they can do about it
   
3. **Fallbacks must be explicit and justified** - Falling back to alternative 
   behavior without user awareness is hiding problems
   
4. **Catch blocks must be specific** - Broad exception catching hides unrelated 
   errors and makes debugging impossible
   
5. **Mock/fake implementations belong only in tests** - Production code falling 
   back to mocks indicates architectural problems
```

→ **每一条都是铁律**，违反任何一条都是严重问题

**审查范围（第 24-32 行）：**

```markdown
### 1. Identify All Error Handling Code

Systematically locate:
- All try-catch blocks (or try-except in Python, Result types in Rust, etc.)
- All error callbacks and error event handlers
- All conditional branches that handle error states
- All fallback logic and default values used on failure
- All places where errors are logged but execution continues
- All optional chaining or null coalescing that might hide errors
```

→ **审查的全面性**：不放过任何可能的错误处理点

**审查问题清单（第 36-66 行）：**

这个 Agent 会问自己一系列问题，确保错误处理的质量：

```markdown
### Logging Quality（日志质量）
- Is the error logged with appropriate severity?
- Does the log include sufficient context?
- Would this log help someone debug the issue 6 months from now?

### User Feedback（用户反馈）
- Does the user receive clear, actionable feedback?
- Is the error message specific enough to be useful?

### Catch Block Specificity（catch 块特异性）
- Does the catch block catch only the expected error types?
- Could this catch block accidentally suppress unrelated errors?

### Fallback Behavior（回退行为）
- Is there fallback logic that executes when an error occurs?
- Does the fallback behavior mask the underlying problem?
- Would the user be confused about why they're seeing fallback behavior?
```

**输出格式（第 99-109 行）：**

```markdown
## Your Output Format

For each issue you find, provide:

1. **Location**: File path and line number(s)
2. **Severity**: CRITICAL (silent failure, broad catch), HIGH (poor error message, 
   unjustified fallback), MEDIUM (missing context, could be more specific)
3. **Issue Description**: What's wrong and why it's problematic
4. **Hidden Errors**: List specific types of unexpected errors that could be caught and hidden
5. **User Impact**: How this affects the user experience and debugging
6. **Recommendation**: Specific code changes needed to fix the issue
7. **Example**: Show what the corrected code should look like
```

**实战输出示例：**

```markdown
## Silent Failure Hunter 报告

### 🔴 CRITICAL: 静默失败

**位置**：`src/api/client.ts:89-92`

**问题描述**：
```typescript
try {
  await fetchUserData(userId);
} catch (error) {
  console.log('Failed to fetch user');  // ❌ 只打印日志，没有进一步处理
  return null;  // ❌ 静默返回 null，调用方不知道出错了
}
```

**严重性**：CRITICAL

**隐藏的错误类型**：
- 网络超时错误
- 用户 ID 不存在
- 服务器内部错误

**用户影响**：
- 用户看到空白页面，不知道为什么
- 开发者无法调试，没有错误堆栈
- Sentry 等监控系统无法捕获

**修复建议**：
```typescript
try {
  await fetchUserData(userId);
} catch (error) {
  logError(error, { userId });  // ✅ 记录完整错误
  throw new ApiError('USER_FETCH_FAILED', userId);  // ✅ 抛出明确的错误
}
```

### 🟡 HIGH: 宽泛的 catch 块

**位置**：`src/utils/parser.ts:45`

**问题描述**：
```typescript
try {
  JSON.parse(data);
} catch (e) {  // ❌ 捕获所有错误，没有区分类型
  return defaultValue;
}
```

**可能隐藏的错误**：
- JSON 格式错误
- 数据为 null/undefined
- 内存不足

**修复建议**：
```typescript
if (!data) {
  logError('No data provided');
  throw new ValidationError('DATA_REQUIRED');
}

try {
  return JSON.parse(data);
} catch (error) {
  if (error instanceof SyntaxError) {
    logError(error, { data });
    throw new ParseError('INVALID_JSON', error);
  }
  throw error;  // 重新抛出非预期错误
}
```
```

---

#### 9.2.2 comment-analyzer：注释分析器

**文件位置：** `plugins/pr-review-toolkit/agents/comment-analyzer.md`

**核心使命（第 8 行）：**

```markdown
You are a meticulous code comment analyzer with deep expertise in technical 
documentation and long-term code maintainability.
```

→ **角色定位**：一丝不苟的注释分析员，专注长期可维护性

**五大审查维度：**

```markdown
1. **Verify Factual Accuracy**（验怔事实准确性）
   - Function signatures match documented parameters and return types
   - Described behavior aligns with actual code logic
   - Referenced types, functions, and variables exist

2. **Assess Completeness**（评估完整性）
   - Critical assumptions or preconditions are documented
   - Non-obvious side effects are mentioned
   - Important error conditions are described

3. **Evaluate Long-term Value**（评估长期价值）
   - Comments that merely restate obvious code should be flagged for removal
   - Comments explaining 'why' are more valuable than those explaining 'what'
   - Comments should be written for the least experienced future maintainer

4. **Identify Misleading Elements**（识别误导元素）
   - Ambiguous language that could have multiple meanings
   - Outdated references to refactored code
   - Assumptions that may no longer hold true

5. **Suggest Improvements**（提出改进建议）
   - Rewrite suggestions for unclear or inaccurate portions
   - Recommendations for additional context where needed
```

**输出格式（第 48-66 行）：**

```markdown
## Your Analysis Output Structure

**Summary**: Brief overview of the comment analysis scope and findings

**Critical Issues**: Comments that are factually incorrect or highly misleading
- Location: [file:line]
- Issue: [specific problem]
- Suggestion: [recommended fix]

**Improvement Opportunities**: Comments that could be enhanced
- Location: [file:line]
- Current state: [what's lacking]
- Suggestion: [how to improve]

**Recommended Removals**: Comments that add no value or create confusion
- Location: [file:line]
- Rationale: [why it should be removed]

**Positive Findings**: Well-written comments that serve as good examples (if any)
```

**实战输出示例：**

```markdown
## Comment Analyzer 报告

### Summary
审查了 5 个文件的注释修改，发现 2 个严重问题，3 个改进机会。

### 🔴 Critical Issues

**位置**：`src/auth/token.ts:23-25`

**原注释**：
```typescript
/**
 * Generates a JWT token that never expires
 * @param userId - The user ID
 * @returns The token string
 */
function generateToken(userId: string): string {
  return jwt.sign({ userId }, SECRET, { expiresIn: '24h' });  // ← 实际 24 小时过期
}
```

**问题**：注释说 "never expires"（永不过期），但代码设置 `expiresIn: '24h'`

**建议修改**：
```typescript
/**
 * Generates a JWT token that expires after 24 hours
 * @param userId - The user ID to encode in the token
 * @returns The encoded JWT token string
 */
```

### 🟡 Improvement Opportunities

**位置**：`src/utils/cache.ts:45`

**当前注释**：
```typescript
// Check if cache exists
if (cache.has(key)) {
  return cache.get(key);
}
```

**问题**：注释只是重复代码，没有解释为什么

**建议修改**：
```typescript
// Return cached value if available (avoids expensive database query)
if (cache.has(key)) {
  return cache.get(key);
}
```

### ♻️ Recommended Removals

**位置**：`src/controllers/user.ts:12`

**注释**：
```typescript
// Loop through all users
for (const user of users) {
  processUser(user);
}
```

**移除理由**：注释完全重复代码，没有任何额外价值

### ✅ Positive Findings

**位置**：`src/middleware/auth.ts:67-72`

**注释**：
```typescript
/**
 * Validates the JWT token and attaches user to request.
 * 
 * Why this approach:
 * - Centralizes authentication logic
 * Allows easy reuse across routes
 * - Makes testing easier (can mock middleware)
 * 
 * @throws AuthenticationError if token is invalid or expired
 */
```

**优点**：解释了"为什么"而不仅仅是"做什么"，提到了设计理由
```

---

#### 9.2.3 type-design-analyzer：类型设计分析器

**文件位置：** `plugins/pr-review-toolkit/agents/type-design-analyzer.md`

**核心专长（第 8 行）：**

```markdown
Your specialty is analyzing and improving type designs to ensure they have 
strong, clearly expressed, and well-encapsulated invariants.
```

→ **专长**：分析类型设计，确保有强大、清晰、封装良好的不变式

**什么是不变式（Invariant）？**

> **通俗解释**：不变式就是"必须始终为真的条件"，无论代码如何执行，这些条件都不能被破坏。

**例子：**
```typescript
// 示例 1：Email 类型的不变式
interface Email {
  value: string;  // 不变式：必须是有效的 email 格式
}

// 示例 2：正整数类型的不变式
interface PositiveInteger {
  value: number;  // 不变式：必须 > 0
}

// 示例 3：用户账户的不变式
class UserAccount {
  balance: number;  // 不变式：不能为负数
  status: 'active' | 'suspended' | 'closed';  // 不变式：只能是这三个值之一
}
```

**五大评估维度（第 24-46 行）：**

```markdown
### 2. Evaluate Encapsulation（封装性） (Rate 1-10)
- Are internal implementation details properly hidden?
- Can the type's invariants be violated from outside?
- Are there appropriate access modifiers?
- Is the interface minimal and complete?

### 3. Assess Invariant Expression（不变式表达） (Rate 1-10)
- How clearly are invariants communicated through the type's structure?
- Are invariants enforced at compile-time where possible?
- Is the type self-documenting through its design?

### 4. Judge Invariant Usefulness（不变式有用性） (Rate 1-10)
- Do the invariants prevent real bugs?
- Are they aligned with business requirements?
- Do they make the code easier to reason about?

### 5. Examine Invariant Enforcement（不变式执行） (Rate 1-10)
- Are invariants checked at construction time?
- Are all mutation points guarded?
- Is it impossible to create invalid instances?
```

**输出格式（第 48-78 行）：**

```markdown
## Type: [TypeName]

### Invariants Identified
- [List each invariant with a brief description]

### Ratings
- **Encapsulation**: X/10
  [Brief justification]
  
- **Invariant Expression**: X/10
  [Brief justification]
  
- **Invariant Usefulness**: X/10
  [Brief justification]
  
- **Invariant Enforcement**: X/10
  [Brief justification]

### Strengths
[What the type does well]

### Concerns
[Specific issues that need attention]

### Recommended Improvements
[Concrete, actionable suggestions]
```

**实战输出示例：**

```markdown
## Type: UserAccount

### Invariants Identified
1. **余额非负**：balance 字段不能小于 0
2. **状态枚举限制**：status 只能是 'active'、'suspended'、'closed' 之一
3. **邮箱唯一性**：email 必须在整个系统中唯一
4. **状态转换规则**：只能 active→suspended→closed，不能反向

### Ratings

- **Encapsulation**: 6/10
  ⚠️ 问题：balance 是 public 可读写的，外部可以直接修改
  ```typescript
  // ❌ 可以直接设置为负数
  account.balance = -1000;
  ```

- **Invariant Expression**: 5/10
  ⚠️ 问题：不变式只在注释中说明，类型本身没有强制
  ```typescript
  interface UserAccount {
    balance: number;  // 注释说不能为负，但类型允许任何数字
  }
  ```

- **Invariant Usefulness**: 8/10
  ✅ 优点：识别出的不变式都是业务必需的
  
- **Invariant Enforcement**: 4/10
  ❌ 严重问题：没有在构造函数中验证不变式
  ```typescript
  // ❌ 可以创建无效实例
  const account = new UserAccount(-100, 'invalid@email');
  ```

### Strengths
✅ 使用了枚举类型限制 status 的取值范围
✅ 有 TypeScript 类型注解

### Concerns
❌ balance 可以直接修改，绕过了不变式检查
❌ 没有在构造时验证 email 格式
❌ 缺少状态转换的方法，外部可以直接设置 status

### Recommended Improvements

**1. 使用私有字段 + getter**
```typescript
class UserAccount {
  private _balance: number;
  
  constructor(initialBalance: number) {
    if (initialBalance < 0) {
      throw new Error('Balance cannot be negative');
    }
    this._balance = initialBalance;
  }
  
  get balance(): number {
    return this._balance;
  }
  
  deposit(amount: number): void {
    if (amount <= 0) throw new Error('Invalid amount');
    this._balance += amount;
  }
}
```

**2. 使用品牌类型（Branded Type）增强类型安全**
```typescript
// 使用品牌类型确保 email 经过验证
type VerifiedEmail = string & { readonly brand: unique symbol };

function createEmail(email: string): VerifiedEmail | null {
  if (isValidEmail(email)) {
    return email as VerifiedEmail;
  }
  return null;
}

interface UserAccount {
  email: VerifiedEmail;  // 现在类型系统确保 email 已验证
}
```

**3. 使用 discriminated union 限制状态转换**
```typescript
type ActiveAccount = { status: 'active'; activatedAt: Date };
type SuspendedAccount = { status: 'suspended'; suspendedAt: Date; reason: string };
type ClosedAccount = { status: 'closed'; closedAt: Date };

type UserAccount = ActiveAccount | SuspendedAccount | ClosedAccount;

// 现在 TypeScript 会在编译期阻止非法状态
```
```

---

#### 9.2.4 code-simplifier：代码简化器

**文件位置：** `plugins/pr-review-toolkit/agents/code-simplifier.md`

**核心特点：使用 Opus 模型**

```yaml
model: opus  # ← 这是少数指定使用 Opus 而非 Sonnet 的 Agent
```

→ **为什么用 Opus？** Opus 是 Claude 系列中最强大的模型，在代码简化和重构方面表现最佳

**核心使命（第 38 行）：**

```markdown
You are an expert code simplification specialist focused on enhancing code clarity, 
consistency, and maintainability while preserving exact functionality.
```

→ **关键约束**："preserving exact functionality" - 只改变"如何做"，不改变"做什么"

**四大优化方向（第 44-68 行）：**

```markdown
### 2. Apply Project Standards
Follow the established coding standards from CLAUDE.md including:
- Use ES modules with proper import sorting and extensions
- Prefer `function` keyword over arrow functions
- Use explicit return type annotations for top-level functions
- Follow proper React component patterns with explicit Props types
- Use proper error handling patterns (avoid try/catch when possible)
- Maintain consistent naming conventions

### 3. Enhance Clarity
Simplify code structure by:
- Reducing unnecessary complexity and nesting
- Eliminating redundant code and abstractions
- Improving readability through clear variable and function names
- Consolidating related logic
- Removing unnecessary comments that describe obvious code
- IMPORTANT: Avoid nested ternary operators - prefer switch statements or if/else chains
- Choose clarity over brevity - explicit code is often better than overly compact code

### 4. Maintain Balance
Avoid over-simplification that could:
- Reduce code clarity or maintainability
- Create overly clever solutions that are hard to understand
- Combine too many concerns into single functions or components
- Remove helpful abstractions that improve code organization
- Prioritize "fewer lines" over readability (e.g., nested ternaries, dense one-liners)
```

→ **平衡的艺术**：既要简化，又不能过度简化；既要简洁，又要清晰

**实战示例：简化前 vs 简化后**

```typescript
// ===== 简化前 =====
const processData = (data, options) => {
  // Check if data is valid
  if (!data || data.length === 0) {
    return options.defaultValue !== undefined ? options.defaultValue : [];
  }
  
  // Process the data based on type
  return data.map(item => {
    if (item.type === 'A') {
      return { ...item, processed: true, timestamp: Date.now() };
    } else if (item.type === 'B') {
      return { ...item, processed: true, timestamp: Date.now(), category: 'B' };
    } else {
      return { ...item, processed: true, timestamp: Date.now(), category: 'default' };
    }
  }).filter(x => x !== null);
};

// ===== 简化后 =====
interface DataItem {
  type: 'A' | 'B' | string;
  processed?: boolean;
  timestamp?: number;
  category?: string;
}

interface ProcessOptions {
  defaultValue?: DataItem[];
}

function processData(data: DataItem[], options: ProcessOptions = {}): DataItem[] {
  if (!data || data.length === 0) {
    return options.defaultValue ?? [];
  }
  
  return data.map(processItem);
}

function processItem(item: DataItem): DataItem {
  const baseResult = {
    ...item,
    processed: true,
    timestamp: Date.now()
  };
  
  switch (item.type) {
    case 'A':
      return baseResult;
    case 'B':
      return { ...baseResult, category: 'B' };
    default:
      return { ...baseResult, category: 'default' };
  }
}
```

**简化要点：**
1. ✅ 添加了明确的类型定义
2. ✅ 提取了独立的处理函数
3. ✅ 用 switch 替代嵌套 if-else
4. ✅ 使用解构减少重复代码
5. ✅ 用 ?? 替代三元运算符
6. ✅ 函数有了清晰的签名和文档

---

#### 9.2.5 pr-test-analyzer：PR 测试分析器

**核心职责：** 分析 PR 的测试覆盖率和质量

**关注重点：**
- 新增代码是否有对应的测试
- 测试是否覆盖了边界情况
- 测试用例设计是否合理
- 是否存在测试盲区

#### 9.2.6 code-reviewer：综合代码审查

**注意**：这与 feature-dev 的 code-reviewer 不同，这个版本专门针对 PR 审查场景优化。

---

### 9.3 plugin-dev 的 3 个辅助 Agent

plugin-dev 是用于开发插件的工具包，包含 3 个帮助创建和验证插件的 Agent。

#### 9.3.1 agent-creator：Agent 创建器

**文件位置：** `plugins/plugin-dev/agents/agent-creator.md`

**核心能力（第 41 行）：**

```markdown
When a user describes what they want an agent to do, you will:

1. **Extract Core Intent**: Identify the fundamental purpose, key responsibilities, 
   and success criteria for the agent.

2. **Design Expert Persona**: Create a compelling expert identity that embodies 
   deep domain knowledge relevant to the task.

3. **Architect Comprehensive Instructions**: Develop a system prompt that:
   - Establishes clear behavioral boundaries
   - Provides specific methodologies and best practices
   - Anticipates edge cases
   - Defines output format expectations

4. **Optimize for Performance**: Include decision-making frameworks, quality control

5. **Create Identifier**: Design a concise, descriptive name

6. **Craft Triggering Examples**: Create 2-4 `<example>` blocks
```

→ **自动化程度**：用户只需描述需求，agent-creator 自动生成完整的 Agent 配置文件

**使用示例：**

```
用户：创建一个帮我审查代码的 Agent

agent-creator：我来为你创建一个代码审查 Agent...

[自动生成的文件]
agents/code-quality-reviewer.md

---
name: code-quality-reviewer
description: |
  Use this agent when you need to review code for quality issues, 
  maintainability problems, and best practice violations.
  
  Examples:
  <example>
  Context: User has finished writing a function
  user: "I've written this function, can you check it?"
  assistant: "I'll use the code-quality-reviewer agent to analyze the code quality."
  </example>
model: inherit
color: blue
---

你是代码质量审查专家...

[完整的 System Prompt]
```

→ **生成内容包括**：
- ✅ 唯一的标识符名称
- ✅ 详细的触发描述
- ✅ 2-4 个触发示例
- ✅ 完整的 System Prompt
- ✅ 合适的模型和颜色选择

---

#### 9.3.2 plugin-validator：插件验证器

**核心职责：** 验证插件的结构完整性、配置正确性、文件规范性

**检查清单：**
- `.claude-plugin/plugin.json` 是否存在且格式正确
- `agents/*.md` 文件是否符合规范
- `hooks/hooks.json` 配置是否正确
- 必需的文件是否齐全

---

#### 9.3.3 skill-reviewer：技能文档审查员

**核心职责：** 审查 `skills/*/SKILL.md` 文档的质量

**审查维度：**
- 文档结构是否清晰
- 示例是否充分
- 说明是否准确
- 是否易于理解和跟随

---

### 9.4 其他专用 Agent

#### 9.4.1 claude-opus-4-5-migration 的 Verifiers

这两个 Agent 用于验证迁移脚本的正确性：

- **agent-sdk-verifier-py.md**：Python 版本验证
- **agent-sdk-verifier-ts.md**：TypeScript 版本验证

#### 9.4.2 hookify 的 conversation-analyzer

**特殊用途：** 分析对话模式，识别潜在的风险行为

---

## 十、Agent vs Skill 机制对比

> **重要区别**：Agent 和 Skill 是两种不同的扩展机制，虽然都是 Markdown 文件，但触发方式和使用场景完全不同。

### 10.1 触发方式的根本区别

| 特性 | Agent | Skill |
|------|-------|-------|
| **触发方式** | 显式/隐式调用 | 自动触发 |
| **文件格式** | `agents/*.md` | `skills/*/SKILL.md` |
| **交互模式** | 主动执行任务 | 被动提供知识 |
| **典型场景** | "用 code-reviewer 审查代码" | 写前端代码时自动应用 |

**图解：**

```
【Agent 的使用流程】
用户需求 → Claude 判断 → 启动 Agent → 执行任务 → 返回结果
              ↓
         可以是显式（用户指定）或隐式（Claude 推荐）

【Skill 的使用流程】
用户开始写前端代码
    ↓
frontend-design Skill 自动激活
    ↓
在后台提供领域知识
    ↓
Claude 的回答包含了 Skill 的知识，但用户感知不到 Skill 的存在
```

### 10.2 frontend-design Skill 示例

**文件位置：** `plugins/frontend-design/skills/frontend-design/SKILL.md`

**自动触发场景：**

```
用户：帮我创建一个登录表单

Claude: （检测到"登录表单"是前端 UI 任务）
    ↓
frontend-design Skill 自动激活
    ↓
提供以下知识：
- 大胆的设计选择（不要用平庸的设计）
- 排版层次和对比度
- 动画和微交互建议
- 视觉细节注意事项
    ↓
Claude 的输出包含了这些指导，但不会说"根据 Skill..."
```

**Skill 的内容片段：**

```markdown
## When to Auto-Invoke

Automatically invoke this skill whenever the user requests or implies:
- Frontend UI development (forms, pages, layouts, components)
- Styling work (CSS, Tailwind, styled-components)
- Interactive elements (buttons, modals, animations)
- Visual design decisions

## What You Provide

When invoked, provide guidance on:
1. **Bold Design Choices**: Avoid generic AI-looking designs
2. **Typography Hierarchy**: Clear visual hierarchy with proper contrast
3. **Animation & Micro-interactions**: Hover states, transitions, feedback
4. **Visual Details**: Spacing, colors, shadows, border radius
```

→ **关键点**：Skill 不会主动说"我要做什么"，而是在 Claude 回答时默默提供知识支持

### 10.3 何时使用 Agent，何时使用 Skill？

**使用 Agent 的场景：**
- ✅ 需要主动执行某个任务
- ✅ 需要专门的分析和输出
- ✅ 需要明确的交付物（报告、方案等）

**使用 Skill 的场景：**
- ✅ 需要领域知识注入
- ✅ 希望在日常工作中自动获得指导
- ✅ 不需要额外的交互，只要背景知识

**简单记忆：**
```
Agent = 主动干活的专家
Skill = 默默提供知识的顾问
```

---

## 十一、常见问题解答

### Q1: 如何选择合适的模型？

**模型选择指南：**

| 模型 | 适用场景 | 成本 | 速度 |
|------|---------|------|------|
| **Sonnet** | 大多数 Agent（平衡性能和成本） | 中等 | 快 |
| **Opus** | 复杂任务（code-simplifier） | 高 | 中等 |
| **inherit** | 不确定时使用主 Claude 的模型 | - | - |

### Q2: tools 应该给哪些？

**最小权限原则：**

```yaml
# code-explorer 需要广泛读取代码
tools: Glob, Grep, LS, Read, NotebookRead, ...

# code-reviewer 只需要读权限
tools: Glob, Grep, LS, Read

# agent-creator 需要写文件
tools: Write, Read
```

### Q3: 多个 Agent 同时工作会冲突吗？

**不会冲突，因为：**
1. 每个 Agent 独立运行
2. 主 Claude 负责整合结果
3. 有明确的职责分工

**示例：**
```
Phase 2: Exploration
├── code-explorer-A → 分析认证模块
├── code-explorer-B → 分析支付模块  ← 并行执行
└── code-explorer-C → 分析架构模式
    ↓
主 Claude 整合三份报告 → 统一的代码库分析结果
```

### Q4: 如何让 Agent 更容易触发？

**在 description 中添加更多触发示例：**

```yaml
description: |
  Use this agent when reviewing code for bugs.
  
  Examples:
  <example>
  user: "检查这段代码"
  assistant: "我用 code-reviewer 审查"
  </example>
  <example>
  user: "有没有什么问题？"
  assistant: "让我启动 code-reviewer"
  </example>
  <example>
  user: "代码写得怎么样？"
  assistant: "我来用 code-reviewer 评估一下"
  </example>
```

---

## 十二、总结

### Q1: Agent 不触发怎么办？

**可能原因：**
1. description 不够清晰
2. 缺少足够的触发示例
3. 用户的表达方式不符合预期

**解决方案：**
```yaml
# 增加更多触发示例
description: |
  示例：
  <example>
  用户："帮我检查代码"
  </example>
  <example>
  用户："审查一下这个实现"
  </example>
  <example>
  用户："看看有没有问题"
  </example>
```

### Q2: Agent 输出质量不稳定？

**可能原因：**
1. System Prompt 不够详细
2. 缺少明确的流程指导
3. 没有定义清晰的输出格式

**解决方案：**
```markdown
# 在 System Prompt 中添加详细流程
**工作流程：**
1. 第一步：具体做什么
2. 第二步：具体做什么
3. 第三步：具体做什么

**输出格式：**
严格按照以下格式：
## 标题
### 小节
- 列表项
```

### Q3: 多个 Agent 冲突怎么办？

**场景：**
```
Agent A 说应该这样做
Agent B 说应该那样做
用户不知道该听谁的
```

**解决方案：**

**方案 1：定义优先级**
```markdown
如果与其他 Agent 建议冲突：
1. 优先考虑安全性（security-agent > 其他）
2. 优先考虑正确性（code-reviewer 的 Bug 报告 > 性能优化）
3. 说明权衡取舍，让用户决定
```

**方案 2：协调机制**
```markdown
当发现冲突时：
1. 明确指出冲突点
2. 分析各自的理由
3. 给出你的建议
4. 最终由用户决定
```

---

## 十、总结

Agent 系统和 Hook 系统不是孤立的，它们可以紧密配合，形成更强大的工作流。

### 9.1 Agent 如何使用 Hook

#### 场景 1：code-reviewer + Stop Hook

**工作流程：**
```
用户：帮我开发一个登录功能
    ↓
feature-dev 启动七阶段流程
    ↓
Phase 6: Quality Review → 激活 code-reviewer Agent
    ↓
code-reviewer 审查代码
    ↓
发现 SQL 注入漏洞
    ↓
触发 Stop Hook（如果配置了质量门禁）
    ↓
阻止任务完成，要求修复
```

**Stop Hook 配置示例：**
```json
{
  "Stop": [
    {
      "type": "prompt",
      "prompt": "检查 code-reviewer Agent 的审查报告。\n如果报告中包含以下问题，阻止停止：\n1. 安全漏洞（SQL 注入、XSS、命令注入）\n2. 严重 Bug（空指针、资源泄漏）\n3. 测试覆盖率低于 80%\n\n返回格式：{\"decision\": \"approve|block\", \"reason\": \"详细说明\"}",
      "timeout": 30
    }
  ]
}
```

**效果：**
```
code-reviewer: 📋 代码审查完成

发现严重问题：
❌ SQL 注入风险 - src/auth/login.ts:45
❌ 缺少密码强度验证 - src/auth/register.ts:67

安全门（Stop Hook）：🔍 基于审查报告验证...

❌ 阻止停止！

原因：code-reviewer 发现了 2 个严重问题未修复。

必须修复后才能完成：
1. 使用参数化查询替代字符串拼接
2. 添加密码复杂度验证

Claude: 明白了，我立即修复这些问题。
```

---

#### 场景 2：code-explorer + SessionStart Hook

**工作流程：**
```
新会话开始
    ↓
SessionStart Hook 触发
    ↓
加载项目上下文信息
    ↓
调用 code-explorer Agent 分析项目结构
    ↓
生成项目知识图谱
    ↓
后续对话中可以使用这些上下文
```

**SessionStart Hook 配置：**
```json
{
  "SessionStart": [
    {
      "type": "command",
      "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/load-project-context.sh"
    }
  ]
}
```

**脚本实现：** `load-project-context.sh`
```bash
#!/bin/bash
# 加载项目上下文

cd "$CLAUDE_PROJECT_DIR" || exit 1

# 检测项目类型
if [ -f "package.json" ]; then
  echo "📦 Node.js 项目"
  echo "export PROJECT_TYPE=nodejs" >> "$CLAUDE_ENV_FILE"
  
  # 调用 code-explorer 分析入口文件
  if [ -f "src/index.ts" ]; then
    echo "主入口：src/index.ts"
    # 可以在这里调用 Agent 分析
  fi
fi

# 加载项目特定的配置
if [ -f ".claude/project-config.md" ]; then
  cat .claude/project-config.md >> "$TRANSCRIPT_PATH"
fi
```

---

#### 场景 3：feature-dev + PreToolUse Hook

**feature-dev 插件**使用了完整的七阶段流程，每个阶段都可以与 Hook 配合。

**PreToolUse Hook 增强：**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "检查当前是否在 feature-dev 的七阶段流程中。\n如果是 Phase 5 (Implementation)，允许写入代码。\n如果不是，提醒用户按照流程执行。\n\n当前阶段可以通过对话历史判断。",
          "timeout": 20
        }
      ]
    }
  ]
}
```

**效果：**
```
用户：直接帮我写代码

Claude: 好的，我准备写入代码...

安全门（PreToolUse）：⚠️ 流程检查...

检测到未遵循七阶段流程！

建议步骤：
1. 先运行 /feature-dev 命令启动流程
2. Phase 1-2: 需求和代码库探索
3. Phase 3: 澄清需求细节
4. Phase 4: 架构设计评审
5. Phase 5: 才开始实现

跳过前期步骤可能导致：
- 需求理解不准确
- 架构设计不合理
- 返工风险高

是否现在启动 feature-dev 流程？
```

---

### 9.2 Hook 如何增强 Agent

#### 增强 1：为 Agent 添加质量门禁

**问题：** Agent 给出的建议可能不够严谨

**解决：使用 Hook 验证 Agent 输出**

```json
{
  "PostToolUse": [
    {
      "matcher": "Read",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "检查 Agent 输出的置信度：\n\n1. 是否有明确的证据支持结论\n2. 是否考虑了边界情况\n3. 是否有过度推测\n\n如果发现以下问题，发出警告：\n- 缺乏依据的断言\n- 忽略重要因素\n- 逻辑推理错误",
          "timeout": 20
        }
      ]
    }
  ]
}
```

**效果：**
```
code-architect: 建议使用微服务架构

安全门（PostToolUse）：🔍 建议质量验证...

⚠️ 置信度评分：60/100

扣分项：
1. ❌ 未分析团队规模（微服务需要足够人手）
2. ❌ 未考虑运维成本
3. ❌ 未对比单体架构的优劣

建议补充：
- 团队现状分析
- 运维能力评估
- ROI 计算

请完善分析后再给出建议。
```

---

#### 增强 2：确保 Agent 遵循规范

**配置：**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "检查 Agent 写入的代码是否符合项目规范：\n\n1. 命名风格（驼峰 vs 下划线）\n2. 文件格式（类名= 文件名）\n3. 注释要求（JSDoc/TSDoc）\n4. 错误处理模式\n\n参考项目的 CLAUDE.md 或 CONTRIBUTING.md",
          "timeout": 20
        }
      ]
    }
  ]
}
```

---

#### 增强 3：多 Agent 协作的流程控制

**复杂场景：** 多个 Agent 需要按顺序执行

**解决方案：** 使用状态跟踪 + Hook 协调

```json
{
  "Stop": [
    {
      "type": "prompt",
      "prompt": "检查七阶段流程的完整性：\n\n阶段清单：\n- [ ] Phase 1: Discovery（需求发现）\n- [ ] Phase 2: Exploration（代码探索）\n- [ ] Phase 3: Clarification（需求澄清）\n- [ ] Phase 4: Architecture（架构设计）\n- [ ] Phase 5: Implementation（实现）\n- [ ] Phase 6: Review（质量审查）\n- [ ] Phase 7: Summary（总结）\n\n如果有跳过的阶段，阻止停止并说明。",
      "timeout": 30
    }
  ]
}
```

**效果：**
```
Claude: 功能已完成，准备提交

安全门（Stop）：📋 七阶段流程完整性检查...

❌ 发现跳过的阶段：

已完成：
✅ Phase 1: Discovery
✅ Phase 2: Exploration
❌ Phase 3: Clarification（跳过）
✅ Phase 4: Architecture
✅ Phase 5: Implementation
✅ Phase 6: Review

缺失 Phase 3 可能导致：
- 需求理解不准确
- 实现偏离预期
- 返工风险

建议：
1. 回顾 Phase 3 的产出物
2. 确认需求已澄清
3. 补充遗漏的分析
```

---

### 9.3 实战案例：完整的集成的工作流

#### 案例：开发用户积分系统

**步骤 1：启动 feature-dev 流程**
```
用户：/feature-dev 开发用户积分系统

feature-dev Agent 启动七阶段流程
```

**步骤 2：Phase 1-2 - Discovery & Exploration**
```
SessionStart Hook 自动加载项目上下文
    ↓
code-explorer Agent 分析现有用户系统
    ↓
输出：用户模型、数据库结构、API 端点
```

**步骤 3：Phase 3 - Clarification**
```
PreToolUse Hook 确保需求澄清：
- 积分获取规则？
- 积分使用场景？
- 积分有效期？
```

**步骤 4：Phase 4 - Architecture**
```
code-architect Agent 设计架构方案
    ↓
PostToolUse Hook 验证方案质量：
- ✅ 考虑了并发性能
- ⚠️ 未提及数据一致性（需补充）
```

**步骤 5：Phase 5 - Implementation**
```
编码阶段
    ↓
security-guidance Hook 实时监控：
- ⚠️ 发现积分计算的精度问题（使用 float 而非 decimal）
- ✅ 及时修正
```

**步骤 6：Phase 6 - Review**
```
code-reviewer Agent 审查代码
    ↓
Stop Hook 验证审查结果：
- ❌ 发现 3 个问题未修复
- 阻止完成，要求修复
```

**步骤 7：Phase 7 - Summary**
```
所有问题修复完成
    ↓
Stop Hook 最终检查：
- ✅ 七阶段流程完整
- ✅ 代码审查通过
- ✅ 安全检查通过
- ✅ 测试运行通过

批准完成任务
```

---

### 9.4 最佳实践

#### ✅ 推荐做法

1. **明确分工**
   ```
   Agent → 负责创造性工作（分析、设计、编码）
   Hook → 负责规范性工作（检查、验证、门禁）
   ```

2. **适度集成**
   ```
   不要：每个 Agent 都配一堆 Hook（过于复杂）
   要：关键节点设置 Hook（质量门禁、安全检查）
   ```

3. **透明提示**
   ```markdown
   好的提示：
   🔍 Hook 检查：发现 2 个安全问题，详见下方报告
   
   不好的提示：
   ❌ 操作被阻止（不说原因）
   ```

#### ❌ 避免的错误

1. **Hook 过多影响体验**
   ```
   每写一行代码都被检查 → 开发效率极低
   ```

2. **Agent 和 Hook 职责混乱**
   ```
   Agent 做检查，Hook 做分析 → 角色错位
   ```

3. **冲突的处理不当**
   ```
   Agent A 说可以，Hook 说不行 → 用户困惑
   正确：明确说明谁有最终决定权
   ```

---

## 十、总结

### 10.1 核心要点

1. **Agent 是什么？**
   - 专门的 AI 助手，有特定专长
   - 可以独立工作，也可以协作

2. **为什么需要？**
   - 专业分工，提高质量
   - 并行处理，提升效率
   - 多角度审视，减少遗漏

3. **如何工作？**
   - 通过配置文件定义行为
   - 通过触发词激活
   - 通过 System Prompt 指导

4. **如何设计？**
   - 单一职责
   - 清晰的触发条件
   - 详细的流程指导
   - 明确的输出格式

### 10.2 与其他系统的关系

```
Hook 系统 → 保护机制（防御）
    ↓
Agent 系统 → 执行机制（进攻）
    ↓
插件系统 → 打包形式（产品）
```

### 10.3 下一步学习

- 📚 **七阶段流程**：Agent 如何在完整流程中协作
- 📚 **插件系统**：如何将 Agent 打包成插件
- 📚 **提示词工程**：如何编写更好的 System Prompt

---

## 十二、总结

### 12.1 核心要点回顾

**1. Agent 是什么？**
- 专门的 AI 助手，有特定专长
- 通过配置文件定义行为
- 可以独立工作，也可以协作

**2. 为什么需要 Agent 协作？**
- **专业分工**：每个 Agent 专注于自己最擅长的领域
- **并行处理**：多个 Agent 同时工作，大幅缩短时间
- **多角度审视**：同一个问题从不同角度分析，更全面

**3. Agent 如何工作？**
- **触发机制**：显式调用、隐式判断、主动推荐
- **配置文件**：Frontmatter（元数据）+ System Prompt（行为指导）
- **工具权限**：遵循最小权限原则，只给必要的工具

**4. 如何设计 Agent？**
- **单一职责**：一个 Agent 只做一件事，做到极致
- **清晰的触发条件**：在 description 中定义明确的触发场景
- **详细的流程指导**：告诉 Agent 第一步做什么、第二步做什么
- **明确的输出格式**：规定输出的结构和内容要求

---

### 12.2 关键 Agent 矩阵

| Agent | 所属插件 | 核心职责 | 触发时机 |
|-------|---------|---------|----------|
| **code-explorer** | feature-dev | 分析现有代码 | Phase 2: Exploration |
| **code-architect** | feature-dev | 设计架构方案 | Phase 4: Architecture |
| **code-reviewer** | feature-dev | 审查代码质量 | Phase 6: Review |
| **silent-failure-hunter** | pr-review-toolkit | 查找静默失败 | PR 审查 |
| **comment-analyzer** | pr-review-toolkit | 检查注释质量 | 添加/修改注释后 |
| **type-design-analyzer** | pr-review-toolkit | 分析类型设计 | 创建新类型时 |
| **code-simplifier** | pr-review-toolkit | 简化代码 | 完成编码后自动 |
| **pr-test-analyzer** | pr-review-toolkit | 分析测试覆盖 | PR 审查 |
| **agent-creator** | plugin-dev | 创建新 Agent | 需要新 Agent 时 |
| **plugin-validator** | plugin-dev | 验证插件结构 | 插件开发完成 |
| **skill-reviewer** | plugin-dev | 审查 Skill 文档 | 编写技能文档后 |

---

### 12.3 Agent 设计最佳实践清单

✅ **命名规范：**
- 使用小写字母和连字符（如 `code-reviewer`）
- 清晰表达用途（避免 `helper`、`assistant` 等泛用词）
- 长度 3-50 个字符

✅ **Description 设计：**
- 以 "Use this agent when..." 开头
- 包含 2-4 个 `<example>` 触发示例
- 示例要展示显式和隐式两种触发方式

✅ **System Prompt 结构：**
- 明确的角色定位（你是 XXX 专家）
- 核心职责列表（1、2、3...）
- 详细的工作流程（第一步、第二步...）
- 质量标准（必须做到什么程度）
- 输出格式（按照 XXX 结构输出）

✅ **模型选择：**
- 默认使用 `inherit`（继承主 Claude 的模型）
- 复杂分析任务使用 `sonnet`
- 超复杂任务（如 code-simplifier）使用 `opus`

✅ **颜色选择：**
- blue/cyan：分析、审查类
- green：生成、创造类
- yellow：探索、调查类
- red：安全、关键类
- magenta：转换、创意类

✅ **工具权限：**
- 遵循最小权限原则
- 只读 Agent 不给写权限
- 审查 Agent 不需要执行权限

---

### 12.4 与其他系统的关系

```
┌─────────────────┐
│   Hook 系统     │ → 保护机制（防御）
│  (Module 02)    │    - PreToolUse 检查
└────────┬────────┘    - Stop 门禁
         │
         ↓ 配合
┌─────────────────┐
│  Agent 系统     │ → 执行机制（进攻）
│  (Module 04)    │    - 代码分析
└────────┬────────┘    - 架构设计
         │             - 代码审查
         ↓ 在...中应用
┌─────────────────┐
│ 七阶段流程      │ → 完整工作流
│  (Module 05)    │    - Phase 2 用 code-explorer
└────────┬────────┘    - Phase 4 用 code-architect
         │             - Phase 6 用 code-reviewer
         ↓ 打包成
┌─────────────────┐
│  插件系统       │ → 产品化
│  (Module 07)    │    - feature-dev 插件
└─────────────────┘    - pr-review-toolkit 插件
```

---

### 12.5 学习路线建议

**初级阶段（1-2 周）：**
1. 理解 Agent 的基本概念
2. 学会阅读 Agent 配置文件
3. 能够使用现有的 Agent

**中级阶段（2-3 周）：**
1. 创建一个简单的 Agent（如专项检查）
2. 理解 System Prompt 的设计技巧
3. 掌握触发机制的设计方法

**高级阶段（3-4 周）：**
1. 设计多 Agent 协作系统
2. 优化 Agent 的性能和质量
3. 将 Agent 打包成插件分享

---

### 12.6 下一步学习方向

📚 **深入学习：**
- 【Module 05-七阶段流程】了解 Agent 如何在完整流程中协作
- 【Module 07-插件系统】学习如何将 Agent 打包发布
- 【Module 10-提示词工程】深入掌握 System Prompt 编写技巧

🔧 **实践项目：**
1. 分析一个现有 Agent 的完整配置
2. 创建一个解决你实际问题的 Agent
3. 为创建的 Agent 编写测试用例
4. 尝试将多个 Agent 组合使用

💡 **进阶挑战：**
- 设计一个支持并行执行的 Agent 系统
- 实现 Agent 之间的冲突检测和解决机制
- 创建自适应的 Agent（根据反馈调整行为）

---

## 十三、附录：参考资源

### 官方示例仓库

**feature-dev Agents:**
- 位置：`plugins/feature-dev/agents/`
- 包含：code-explorer, code-architect, code-reviewer
- 特点：完整的七阶段流程实现

**pr-review-toolkit Agents:**
- 位置：`plugins/pr-review-toolkit/agents/`
- 包含：6 个专用审查 Agent
- 特点：高度专业化，每个 Agent 只做好一件事

**plugin-dev Agents:**
- 位置：`plugins/plugin-dev/agents/`
- 包含：agent-creator, plugin-validator, skill-reviewer
- 特点：用于开发其他 Agent 的元工具

### 开发工具

**Agent 验证脚本:**
```bash
# 验证单个 Agent
./scripts/validate-agent.sh agents/code-reviewer.md

# 验证整个插件
./scripts/validate-plugin.sh plugins/feature-dev/
```

**Agent 开发技能:**
- 位置：`plugins/plugin-dev/skills/agent-development/SKILL.md`
- 内容：完整的 Agent 开发指南

### 调试技巧

**测试 Agent 触发:**
```
1. 直接调用："用 [agent-name] 做 XXX"
2. 隐式触发：描述 Agent 应该处理的场景
3. 查看日志：观察 Agent 是否被正确激活
```

**检查 Agent 输出质量:**
```
1. 是否符合输出格式要求？
2. 是否包含了所有必需的信息？
3. 置信度评分是否合理？
4. 建议是否具体可操作？
```

**优化 Agent 性能:**
```
1. 减少不必要的工具调用
2. 优化 System Prompt 的长度（500-3000 字为宜）
3. 调整触发条件的敏感度
4. 收集用户反馈持续改进
```

---

**最后更新：** 2026 年 4 月 2 日  
**版本：** v2.0（基于实际 Agent 代码全面更新）  
**维护者：** Claude Code 项目学习文档团队
