# 核心 Agent 逐行解读

> 本章将对 feature-dev 插件的三个核心 Agent 进行逐行完整解读，帮助你深入理解 Agent 的设计细节。

---

## 一、code-explorer.md 完整解读

### 1.1 文件位置和基本结构

**文件路径：** `plugins/feature-dev/agents/code-explorer.md`

**文件大小：** 52 行

**核心职责：** 深入分析现有代码库，理解功能和架构。

### 1.2 Frontmatter 配置详解（第 1-7 行）

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

**字段解读：**

| 字段 | 值 | 为什么这样选择 |
|------|-----|---------------|
| **name** | `code-explorer` | 清晰表达职责：探索代码库 |
| **description** | 详细描述了探索执行路径、映射架构层等 | 让 Claude 知道何时触发此 Agent，包含关键词便于语义匹配 |
| **tools** | 10 个工具 | 需要多种工具来全面分析代码：<br>- Glob/Grep：查找文件<br>- Read：读取内容<br>- LS：查看目录<br>- WebFetch：获取外部资料<br>- TodoWrite：管理任务列表 |
| **model** | `sonnet` | Sonnet 模型在代码理解方面表现优秀，平衡性能和成本 |
| **color** | `yellow` | 黄色代表分析、探索类型的工作 |

### 1.3 System Prompt 逐段分析

#### 角色定位（第 9 行）

```markdown
You are an expert code analyst specializing in tracing and understanding 
feature implementations across codebases.
```

→ **角色定位**：代码分析专家，专注于追踪和理解功能实现

→ **设计要点**：
- `expert`：强调专业性
- `specializing in`：明确专注领域
- `across codebases`：说明跨项目的通用能力

---

#### 核心使命（第 11-13 行）

```markdown
## Core Mission
Provide a complete understanding of how a specific feature works by tracing 
its implementation from entry points to data storage, through all abstraction layers.
```

→ **核心使命**：提供对功能如何工作的完整理解，从入口点到数据存储，穿越所有抽象层

→ **关键要素**：
- `complete understanding`：强调完整性
- `from entry points to data storage`：覆盖全链路
- `through all abstraction layers`：不遗漏任何层次

---

#### 工作流程设计（第 14-37 行）

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

**逐步解读：**

**Step 1 - Feature Discovery：**
- 找入口点：API 接口、UI 组件、命令行工具
- 定位核心实现文件
- 映射功能边界和配置

**Step 2 - Code Flow Tracing：**
- 追踪调用链：从入口到输出
- 追踪数据转换：每一步数据如何变化
- 识别依赖关系：内部依赖和外部集成
- 记录状态变化和副作用

**Step 3 - Architecture Analysis：**
- 映射抽象层次：展示层 → 业务逻辑层 → 数据层
- 识别设计模式和架构决策
- 记录组件之间的接口
- 关注横切关注点：认证、日志、缓存

**Step 4 - Implementation Details：**
- 关键算法和数据结构
- 错误处理和边界情况
- 性能考虑
- 技术债务或改进空间

---

#### 输出格式规范（第 39-51 行）

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

---

### 1.4 实际使用案例

**场景：开发 OAuth 登录功能前分析现有认证系统**

```
用户：我要开发一个 OAuth 登录功能，需要先了解现有的认证系统

Claude: 好的，让我启动 code-explorer Agent 来分析现有认证系统...

[code-explorer 分析报告]

## 代码探索报告：用户认证流程

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

### 架构洞察
- MVC 分层架构
- 中间件模式用于认证
- 使用 Redis 存储 session

### 必须了解的文件
- src/auth/AuthService.ts
- src/middleware/authMiddleware.ts
- src/config/security.ts
```

---

## 二、code-architect.md 完整解读

### 2.1 文件位置和基本结构

**文件路径：** `plugins/feature-dev/agents/code-architect.md`

**文件大小：** 35 行

**核心职责：** 设计架构方案，提供实施蓝图。

### 2.2 Frontmatter 配置详解（第 1-7 行）

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

### 2.3 System Prompt 逐段分析

#### 角色定位（第 9 行）

```markdown
You are a senior software architect who delivers comprehensive, actionable 
architecture blueprints by deeply understanding codebases and making 
confident architectural decisions.
```

→ **角色定位**：资深软件架构师，交付全面、可实施的架构蓝图

→ **关键能力**：
- `deeply understanding codebases`：深入理解代码库
- `making confident architectural decisions`：做出果断的架构决策

---

#### 核心流程（第 11-20 行）

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
integration points, and data flow. Break implementation into clear phases 
with specific tasks.
```

→ **三步曲设计**：

**Step 1 - Codebase Pattern Analysis：**
- 提取现有模式、约定和架构决策
- 识别技术栈、模块边界、抽象层次
- 阅读 CLAUDE.md 指南
- 找到类似功能了解既有方法

**Step 2 - Architecture Design：**
- 基于发现的模式设计完整架构
- **关键点**：`Make decisive choices - pick one approach and commit`
  - 做出果断选择，选定一个方案并坚持
  - 不要模棱两可，不要罗列多个选项让用户选
- 确保与现有代码无缝集成
- 为可测试性、性能、可维护性而设计

**Step 3 - Complete Implementation Blueprint：**
- 指定每个要创建或修改的文件
- 组件职责
- 集成点
- 数据流
- 分阶段实施清单

---

#### 输出格式规范（第 22-34 行）

```markdown
## Output Guidance

Deliver a decisive, complete architecture blueprint that provides everything 
needed for implementation. Include:

- **Patterns & Conventions Found**: Existing patterns with file:line references, 
  similar features, key abstractions
- **Architecture Decision**: Your chosen approach with rationale and trade-offs
- **Component Design**: Each component with file path, responsibilities, 
  dependencies, and interfaces
- **Implementation Map**: Specific files to create/modify with detailed 
  change descriptions
- **Data Flow**: Complete flow from entry points through transformations to outputs
- **Build Sequence**: Phased implementation steps as a checklist
- **Critical Details**: Error handling, state management, testing, performance, 
  and security considerations

Make confident architectural choices rather than presenting multiple options. 
Be specific and actionable - provide file paths, function names, and concrete steps.
```

→ **输出要求**：
- ✅ **Patterns & Conventions Found**：现有模式和约定，带文件路径和行号
- ✅ **Architecture Decision**：选定的方案和理由，权衡分析
- ✅ **Component Design**：每个组件的文件路径、职责、依赖、接口
- ✅ **Implementation Map**：具体要创建/修改的文件及详细说明
- ✅ **Data Flow**：从入口到输出的完整数据流
- ✅ **Build Sequence**：分阶段实施步骤清单
- ✅ **Critical Details**：错误处理、状态管理、测试、性能、安全考虑

→ **关键要求**：
- `Make confident architectural choices rather than presenting multiple options`
  - 做出果断的架构选择，而不是呈现多个选项
- `Be specific and actionable`
  - 具体且可实施：提供文件路径、函数名、具体步骤

---

### 2.4 实际使用案例

**场景：设计积分系统架构**

```
用户：基于现有代码库，设计一个电商积分系统

Claude: 好的，让我启动 code-architect Agent 来设计架构...

[code-architect 分析报告]

## 架构设计方案：电商积分系统

### Patterns & Conventions Found

**现有技术栈：**
- Backend: Node.js + Express + MongoDB
- 认证：JWT token，中间件模式
- 支付：Stripe SDK，webhook 处理

**类似功能参考：**
- 优惠券系统 (`src/coupon/`)：使用了策略模式
- 会员等级 (`src/membership/`)：事件驱动架构

**CLAUDE.md 规范：**
- 所有服务类放在 `src/services/`
- 使用 TypeScript 严格模式
- 必须有单元测试

---

### Architecture Decision

**选择方案：事件驱动架构**

**理由：**
1. 与现有会员等级系统一致
2. 易于扩展新的积分规则
3. 解耦积分获取和使用逻辑

**权衡：**
- ✅ 优点：灵活、可扩展、符合现有模式
- ⚠️ 缺点：需要消息队列基础设施

---

### Component Design

**1. PointsService (`src/services/points/PointsService.ts`)**
- 职责：积分计算、发放、扣除
- 依赖：UserService, OrderService
- 接口：`addPoints(userId, amount)`, `deductPoints(userId, amount)`

**2. PointsRepository (`src/repositories/PointsRepository.ts`)**
- 职责：积分数据持久化
- 依赖：MongoDB Model
- 接口：`findByUserId()`, `updateBalance()`

**3. PointsEventListener (`src/eventListeners/PointsEventListener.ts`)**
- 职责：监听订单完成、退货等事件
- 依赖：EventBus
- 触发：`OrderCompleted`, `OrderReturned`

---

### Implementation Map

**Phase 1: 数据层（预计 2 小时）**
1. 创建 `src/models/PointsModel.ts`
   - 定义 Mongoose Schema
   - 添加索引：userId + expiredAt
2. 创建 `src/repositories/PointsRepository.ts`
   - 实现 CRUD 方法

**Phase 2: 业务层（预计 4 小时）**
3. 创建 `src/services/points/PointsService.ts`
   - 实现积分计算逻辑
   - 添加事务处理
4. 创建 `src/services/points/RulesService.ts`
   - 实现积分规则引擎

**Phase 3: 集成层（预计 3 小时）**
5. 创建 `src/eventListeners/PointsEventListener.ts`
   - 订阅订单事件
   - 实现自动积分发放

---

### Data Flow

**积分获取流程：**
```
用户下单 → OrderCompleted 事件 → PointsEventListener
  ↓
RulesService 计算积分 → PointsService.addPoints()
  ↓
PointsRepository 更新余额 → 发送通知
```

**积分使用流程：**
```
用户兑换 → PointsService.deductPoints()
  ↓
验证余额充足 → 更新余额 → 记录流水
  ↓
如果余额不足 → 抛出异常 → 返回错误
```

---

### Build Sequence

- [ ] Phase 1 完成：数据模型和 Repository
- [ ] Phase 2 完成：业务逻辑和服务
- [ ] Phase 3 完成：事件监听和集成
- [ ] Phase 4：单元测试（覆盖率 > 80%）
- [ ] Phase 5：集成测试
- [ ] Phase 6：文档编写

---

### Critical Details

**错误处理：**
- 积分不足时抛出 `InsufficientPointsError`
- 使用 try-catch-finally 确保事务回滚

**状态管理：**
- 积分余额变更使用乐观锁
- 记录每次变更的流水（PointsTransaction）

**测试策略：**
- 单元测试：Jest + MongoDB Memory Server
- 集成测试：Supertest + 真实数据库

**性能优化：**
- 余额查询使用 Redis 缓存
- 批量操作使用 bulkWrite

**安全考虑：**
- 积分变更需要用户身份验证
- 防止并发请求导致的超扣
```

---

## 三、code-reviewer.md 完整解读

### 3.1 文件位置和基本结构

**文件路径：** `plugins/feature-dev/agents/code-reviewer.md`

**文件大小：** 47 行

**核心职责：** 审查代码质量，发现问题并提出改进建议。

### 3.2 Frontmatter 配置详解（第 1-7 行）

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

**字段解读：**

| 字段 | 值 | 为什么这样选择 |
|------|-----|---------------|
| **name** | `code-reviewer` | 清晰表达职责：代码审查员 |
| **description** | 强调了置信度过滤机制 | 突出核心特点：只报告真正重要的问题 |
| **tools** | 10 个工具 | 需要读取代码、查找模式、验证问题 |
| **model** | `sonnet` | Sonnet 在代码审查方面表现出色 |
| **color** | `red` | 红色代表警告、审查、关键问题 |

### 3.3 System Prompt 逐段分析

#### 角色定位（第 9 行）

```markdown
You are an expert code reviewer specializing in modern software development 
across multiple languages and frameworks. Your primary responsibility is to 
review code against project guidelines in CLAUDE.md with high precision 
to minimize false positives.
```

→ **角色定位**：代码审查专家，专注于现代软件开发，高精度减少误报

→ **关键要求**：
- `high precision to minimize false positives`
  - 高精度，最小化误报
  - 这是 code-reviewer 的核心设计哲学

---

#### 审查范围（第 11-14 行）

```markdown
## Review Scope

By default, review unstaged changes from `git diff`. 
The user may specify different files or scope to review.
```

→ **默认范围**：`git diff` 的未暂存变更

→ **灵活性**：用户可以指定不同的文件或范围

---

#### 核心审查职责（第 15-22 行）

```markdown
## Core Review Responsibilities

**Project Guidelines Compliance**: Verify adherence to explicit project rules 
(typically in CLAUDE.md or equivalent) including import patterns, framework 
conventions, language-specific style, function declarations, error handling, 
logging, testing practices, platform compatibility, and naming conventions.

**Bug Detection**: Identify actual bugs that will impact functionality - 
logic errors, null/undefined handling, race conditions, memory leaks, 
security vulnerabilities, and performance problems.

**Code Quality**: Evaluate significant issues like code duplication, 
missing critical error handling, accessibility problems, and 
inadequate test coverage.
```

→ **三大审查维度：**

**1. Project Guidelines Compliance（项目规范遵循）：**
- 验证是否符合明确的项目规则（通常在 CLAUDE.md 中）
- 包括：导入模式、框架约定、语言特定风格、函数声明、错误处理、日志记录、测试实践、平台兼容性、命名约定

**2. Bug Detection（Bug 检测）：**
- 识别真正影响功能的 Bug
- 包括：逻辑错误、null/undefined 处理、竞态条件、内存泄漏、安全漏洞、性能问题

**3. Code Quality（代码质量）：**
- 评估重大问题
- 包括：代码重复、缺少关键错误处理、无障碍性问题、测试覆盖不足

---

#### 置信度评分机制（第 23-33 行）

```markdown
## Confidence Scoring

Rate each potential issue on a scale from 0-100:

- **0**: Not confident at all. This is a false positive that doesn't stand up 
  to scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be 
  a false positive. If stylistic, it wasn't explicitly called out in project guidelines.
- **50**: Moderately confident. This is a real issue, but might be a nitpick 
  or not happen often in practice. Not very important relative to the rest of the changes.
- **75**: Highly confident. Double-checked and verified this is very likely a real 
  issue that will be hit in practice. The existing approach is insufficient. Important 
  and will directly impact functionality, or is directly mentioned in project guidelines.
- **100**: Absolutely certain. Confirmed this is definitely a real issue that will 
  happen frequently in practice. The evidence directly confirms this.

**Only report issues with confidence ≥ 80.** Focus on issues that truly matter - 
quality over quantity.
```

→ **置信度评分标准详解：**

| 分数 | 含义 | 是否报告 | 示例 |
|------|------|---------|------|
| **0** | 完全误报，假阳性 | ❌ | 预存在的问题或误解 |
| **25** | 可能误报 | ❌ | 风格问题，未违反规范 |
| **50** | 真实但次要 | ❌ | 小问题，不常发生 |
| **75** | 高度确信 | ✅ | 影响功能，违反规范 |
| **100** | 绝对确定 | ✅ | 频繁发生，证据确凿 |

→ **阈值设置：** `Only report issues with confidence ≥ 80`
- 只报告置信度 ≥ 80 的问题（即 75 分和 100 分）
- **设计哲学**：`quality over quantity`（质量优于数量）

---

#### 输出格式规范（第 35-46 行）

```markdown
## Output Guidance

Start by clearly stating what you're reviewing. For each high-confidence issue, 
provide:

- Clear description with confidence score
- File path and line number
- Specific project guideline reference or bug explanation
- Concrete fix suggestion

Group issues by severity (Critical vs Important). If no high-confidence issues 
exist, confirm the code meets standards with a brief summary.

Structure your response for maximum actionability - developers should know 
exactly what to fix and why.
```

→ **输出要求：**
- ✅ 清楚说明审查对象
- ✅ 每个高置信度问题包含：
  - 清晰的描述和置信度分数
  - 文件路径和行号
  - 具体的项目规范引用或 Bug 解释
  - 具体的修复建议
- ✅ 按严重程度分组（Critical vs Important）
- ✅ 如果没有高置信度问题，简要总结确认代码符合标准
- ✅ 结构化响应，最大化可操作性

---

### 3.4 实际使用案例

**场景：审查用户认证模块代码**

```
用户：我写完了登录功能，帮我审查一下

Claude: 好的，让我启动 code-reviewer Agent 审查代码...

[code-reviewer 分析报告]

## 代码审查报告

**审查范围：** git diff 中的未暂存变更
**审查时间：** 2026-04-02

---

### Critical Issues（置信度 ≥ 90）

**1. SQL 注入风险** - `src/auth/login.ts:45` (置信度：100)

**问题描述：**
使用字符串拼接构建 SQL 查询，存在严重的 SQL 注入漏洞。

**当前代码：**
```typescript
const query = `SELECT * FROM users WHERE username = '${username}' AND password = '${password}'`;
```

**违反规范：**
CLAUDE.md Section 3.2: "All database queries must use parameterized statements"

**修复建议：**
使用参数化查询：
```typescript
const query = 'SELECT * FROM users WHERE username = ? AND password = ?';
const params = [username, hashedPassword];
```

**预期影响：**
防止恶意用户通过输入 `' OR '1'='1` 绕过认证。

---

### Important Issues（置信度 80-89）

**2. 缺少错误处理** - `src/auth/login.ts:67` (置信度：95)

**问题描述：**
try-catch 块为空，捕获异常后不做任何处理。

**当前代码：**
```typescript
try {
  await loginUser(username, password);
} catch (error) {
  // TODO: handle error
}
```

**潜在风险：**
- 用户无法得知失败原因
- 调试困难
- 可能导致静默失败

**修复建议：**
```typescript
try {
  await loginUser(username, password);
} catch (error) {
  logger.error('Login failed', { username, error });
  throw new AuthenticationError('Invalid credentials');
}
```

---

**3. 密码未哈希存储** - `src/auth/register.ts:23` (置信度：100)

**问题描述：**
用户密码以明文形式存储到数据库，未进行哈希加密。

**当前代码：**
```typescript
await db.users.create({
  username,
  password  // 明文密码！
});
```

**安全风险：**
- 数据库泄露将暴露所有用户密码
- 违反安全最佳实践
- 可能违反 GDPR 等法规

**修复建议：**
使用 bcrypt 进行哈希：
```typescript
import bcrypt from 'bcrypt';

const saltRounds = 10;
const hashedPassword = await bcrypt.hash(password, saltRounds);

await db.users.create({
  username,
  password: hashedPassword
});
```

---

### Positive Findings

✅ 良好的模块化设计：认证逻辑独立成 service 层
✅ 充分的注释说明：关键步骤都有注释
✅ 使用 TypeScript 类型注解：提高代码可读性

---

### Summary

发现 3 个高优先级问题需要修复：
- 1 个严重安全问题（SQL 注入）
- 1 个严重安全问题（明文密码）
- 1 个重要错误处理问题

建议立即修复所有 Critical 问题后再部署。
```

---

## 四、三个核心 Agent 的协作流程

### 4.1 在七阶段流程中的位置

```
Phase 1: Discovery（需求发现）
  ↓
Phase 2: Exploration（代码库探索）
  → 启动 2-3 个 code-explorer Agent（并行）
  ↓
Phase 3: Clarification（需求澄清）
  ↓
Phase 4: Architecture（架构设计）
  → 启动 2-3 个 code-architect Agent（并行）
  ↓
Phase 5: Implementation（实现）
  ↓
Phase 6: Review（质量审查）
  → 启动 3 个 code-reviewer Agent（并行）
  ↓
Phase 7: Summary（总结）
```

### 4.2 实际协作案例

**场景：开发电商积分系统**

```
【Phase 2 - code-explorer 工作】

Explorer Agent A: 分析现有优惠券系统
  → 发现使用了策略模式和事件驱动架构
  
Explorer Agent B: 研究会员等级模块
  → 确认使用 MongoDB 和 Redis 缓存
  
Explorer Agent C: 查找支付相关代码
  → 定位 Stripe 集成点和 webhook 处理

主 Claude 整合：
## 代码库分析报告
- 现有技术栈：Node.js + Express + MongoDB + Redis
- 类似功能：优惠券系统（策略模式）、会员等级（事件驱动）
- 需要了解的文件：[列出 10 个关键文件]

---

【Phase 4 - code-architect 工作】

Architect Agent A: 最小改动方案
  → 扩展现有优惠券系统，添加积分功能
  
Architect Agent B: 清晰架构方案
  → 全新积分服务，微服务化
  
Architect Agent C: 实用平衡方案（推荐）
  → 事件驱动架构，复用现有基础设施

主 Claude 整合并推荐：
## 架构方案对比
- 方案 1：快速但有技术债务
- 方案 2：清晰但工作量大
- 方案 3（推荐）：平衡速度和清晰度

---

【Phase 6 - code-reviewer 工作】

Reviewer Agent A: 代码质量审查
  → 发现代码重复和命名不规范
  
Reviewer Agent B: Bug 扫描
  → 发现内存泄漏和空指针风险
  
Reviewer Agent C: 规范符合度审查
  → 确认符合 CLAUDE.md 要求

主 Claude 整合：
## 质量审查结果
- 发现 2 个 Critical 问题（必须修复）
- 发现 3 个 Important 问题（建议修复）
- 代码整体符合项目规范
```

---

## 五、设计亮点总结

### 5.1 code-explorer 的设计亮点

1. **四步递进流程**：Discovery → Tracing → Analysis → Details
2. **全链路覆盖**：从入口点到数据存储，不遗漏任何层次
3. **精确到行号**：要求所有引用都包含 file:line

### 5.2 code-architect 的设计亮点

1. **基于现有模式**：不是凭空设计，而是复用已有模式
2. **果断决策**：选择一个方案并坚持，不模棱两可
3. **完整蓝图**：具体到每个文件、每个函数、每个集成点

### 5.3 code-reviewer 的设计亮点

1. **置信度评分**：0-100 分评级，只报告≥80 分的问题
2. **减少误报**：明确说"Not confident at all"的情况直接丢弃
3. **可操作性**：每个问题都有具体修复建议和代码示例

---

## 六、学习要点

### 6.1 关键洞察

1. **专业化分工**：每个 Agent 只做好一件事
2. **标准化流程**：都有清晰的步骤指导
3. **格式化输出**：一致的输出格式便于理解
4. **质量保障**：置信度评分保证报告质量

### 6.2 可以借鉴的模式

1. **角色定位公式**：`你是 [领域] 专家，专注于 [具体职责]`
2. **流程设计模板**：信息收集 → 分析处理 → 输出交付
3. **质量控制机制**：置信度评分 + 阈值过滤
4. **输出格式化**：结构化、可操作、包含示例

---

**最后更新：** 2026 年 4 月 2 日  
**版本：** v1.0（完整版）  
**维护者：** Claude Code 项目学习文档团队
