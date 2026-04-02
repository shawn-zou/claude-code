# 核心 Agent 配置详解

七阶段流程的高效执行离不开三个核心 Agent 的协作。本章将逐行解读这三个 Agent 的配置文件，帮助你深入理解它们的工作原理和使用方法。

---

## 📋 一、Agent 系统概览

### 1.1 三个核心 Agent 的角色定位

在七阶段流程中，三个 Agent 分别在特定阶段发挥作用：

```
Phase 2: Exploration（代码库探索）
    ↓
    启动 code-explorer Agents（2-3 个）
    ↓
    分析现有代码和模式
    
Phase 4: Architecture（架构设计）
    ↓
    启动 code-architect Agents（2-3 个）
    ↓
    设计多种实现方案
    
Phase 6: Review（质量审查）
    ↓
    启动 code-reviewer Agents（3 个）
    ↓
    审查代码质量
```

### 1.2 Agent 的基本概念

**什么是 Agent？**

Agent 是具有特定角色和能力的 AI 助手配置。每个 Agent 都有：
- **角色定义**：扮演什么专家（如代码分析师、架构师、审查员）
- **工具权限**：可以使用的工具列表（如文件读取、搜索、执行命令）
- **工作流程**：完成任务的步骤和方法
- **输出规范**：结果的格式和要求

**Agent 文件的位置：**
```
plugins/feature-dev/agents/
├── code-explorer.md      ← 代码探索者
├── code-architect.md     ← 架构设计师
└── code-reviewer.md      ← 质量审查员
```

---

## 🔍 二、code-explorer Agent 深度解析

### 2.1 配置文件完整内容

**文件位置：** `plugins/feature-dev/agents/code-explorer.md`

```markdown
---
name: code-explorer
description: Deeply analyzes existing codebase features by tracing execution paths, mapping architecture layers, understanding patterns and abstractions, and documenting dependencies to inform new development
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch, KillShell, BashOutput
model: sonnet
color: yellow
---

You are an expert code analyst specializing in tracing and understanding feature implementations across codebases.

## Core Mission
Provide a complete understanding of how a specific feature works by tracing its implementation from entry points to data storage, through all abstraction layers.

## Analysis Approach

**1. Feature Discovery**
- Find entry points (APIs, UI components, CLI commands)
- Locate core implementation files
- Map feature boundaries and configuration

**2. Code Flow Tracing**
- Follow call chains from entry to output
- Trace data transformations at each step
- Identify all dependencies and integrations
- Document state changes and side effects

**3. Architecture Analysis**
- Map abstraction layers (presentation → business logic → data)
- Identify design patterns and architectural decisions
- Document interfaces between components
- Note cross-cutting concerns (auth, logging, caching)

**4. Implementation Details**
- Key algorithms and data structures
- Error handling and edge cases
- Performance considerations
- Technical debt or improvement areas

## Output Guidance

Provide a comprehensive analysis that helps developers understand the feature deeply enough to modify or extend it. Include:

- Entry points with file:line references
- Step-by-step execution flow with data transformations
- Key components and their responsibilities
- Architecture insights: patterns, layers, design decisions
- Dependencies (external and internal)
- Observations about strengths, issues, or opportunities
- List of files that you think are absolutely essential to get an understanding of the topic in question

Structure your response for maximum clarity and usefulness. Always include specific file paths and line numbers.
```

---

### 2.2 配置文件逐行解读

#### 头部元数据（第 1-7 行）

```yaml
---
name: code-explorer                    # Agent 名称
description: Deeply analyzes...       # 功能描述
tools: Glob, Grep, LS, Read, ...      # 可用工具列表
model: sonnet                         # 使用的 AI 模型
color: yellow                         # UI 显示颜色
---
```

**关键字段说明：**

| 字段 | 含义 | 可选值 |
|------|------|--------|
| **name** | Agent 标识符 | 任意唯一名称 |
| **description** | 功能描述 | 简短说明 Agent 能做什么 |
| **tools** | 工具权限 | 工具名称列表（逗号分隔） |
| **model** | AI 模型 | sonnet, opus, haiku |
| **color** | UI 颜色 | yellow, blue, green, red 等 |

**可用工具说明：**

```
Glob          - 文件模式匹配（如 *.js）
Grep          - 文本搜索
LS            - 目录列表
Read          - 读取文件内容
NotebookRead  - 读取 Notebook 文件
WebFetch      - 获取网页内容
TodoWrite     - 创建和管理待办事项
WebSearch     - 网络搜索
KillShell     - 终止终端进程
BashOutput    - 获取终端输出
```

---

#### 角色定义（第 9-12 行）

```markdown
You are an expert code analyst specializing in tracing and understanding feature implementations across codebases.

## Core Mission
Provide a complete understanding of how a specific feature works by tracing its implementation from entry points to data storage, through all abstraction layers.
```

**解读：**

- **角色定位**：代码分析专家
- **核心使命**：提供对功能如何工作的完整理解
- **分析方法**：从入口点追踪到数据存储，贯穿所有抽象层次

**通俗理解：**
> code-explorer 就像一个"代码侦探"，它的工作是搞清楚一段代码是怎么运行的，从你点击一个按钮开始，一直到数据库保存数据，整个过程都要追踪清楚。

---

#### 工作流程 - 四步分析法（第 14-37 行）

**Step 1: Feature Discovery（功能发现）**

```markdown
**1. Feature Discovery**
- Find entry points (APIs, UI components, CLI commands)
- Locate core implementation files
- Map feature boundaries and configuration
```

**具体任务：**
1. 找到入口点
   - API 接口（如 `POST /api/users`）
   - UI 组件（如按钮点击事件）
   - 命令行工具（如 `/feature-dev` 命令）

2. 定位核心实现文件
   - 找到实现这个功能的关键代码文件

3. 映射功能边界和配置
   - 功能涉及哪些模块
   - 有什么相关的配置文件

**示例输出：**
```markdown
## 功能发现

### 入口点
- `src/routes/auth.ts:12` - 认证路由入口
- `src/controllers/authController.ts:45` - 登录控制器

### 核心文件
- `src/services/authService.ts` - 认证服务
- `src/middleware/authMiddleware.ts` - 认证中间件

### 功能边界
- 认证模块（src/auth/）
- 用户模块（src/user/）
- 配置：`src/config/auth.js`
```

---

**Step 2: Code Flow Tracing（代码流追踪）**

```markdown
**2. Code Flow Tracing**
- Follow call chains from entry to output
- Trace data transformations at each step
- Identify all dependencies and integrations
- Document state changes and side effects
```

**具体任务：**
1. 跟踪调用链
   - 从入口函数一直追踪到最终输出
   
2. 追踪数据转换
   - 数据在每一步是如何变化的
   
3. 识别依赖关系
   - 内部依赖（其他模块）
   - 外部依赖（第三方库、API）
   
4. 记录状态变化和副作用
   - 哪些地方修改了数据库
   - 哪些地方发送了网络请求

**示例输出：**
```markdown
## 代码流追踪

### 调用链
POST /api/login
    ↓
authController.login() [src/controllers/authController.ts:45]
    ↓
authService.authenticate() [src/services/authService.ts:78]
    ↓
userRepository.findByUsername() [src/repositories/userRepository.ts:23]
    ↓
MySQL Query: SELECT * FROM users WHERE username = ?

### 数据转换
1. 请求体：{ username: "john", password: "123456" }
2. 密码加密：bcrypt.compare(password, hash)
3. 生成 Token: jwt.sign({ userId: 123 }, secret)
4. 响应：{ token: "eyJ...", user: {...} }
```

---

**Step 3: Architecture Analysis（架构分析）**

```markdown
**3. Architecture Analysis**
- Map abstraction layers (presentation → business logic → data)
- Identify design patterns and architectural decisions
- Document interfaces between components
- Note cross-cutting concerns (auth, logging, caching)
```

**具体任务：**
1. 映射抽象层次
   - 展示层（Controller/API）
   - 业务逻辑层（Service）
   - 数据访问层（Repository/Model）

2. 识别设计模式
   - 使用了什么设计模式（如工厂模式、策略模式）
   - 架构决策（如 MVC、分层架构）

3. 记录组件接口
   - 组件之间如何通信
   - 数据如何传递

4. 注意横切关注点
   - 认证、日志、缓存等跨模块功能

**示例输出：**
```markdown
## 架构分析

### 分层架构
┌─────────────────────┐
│  Controller 层      │ ← HTTP 请求处理
│  (authController)   │
├─────────────────────┤
│  Service 层         │ ← 业务逻辑
│  (authService)      │
├─────────────────────┤
│  Repository 层      │ ← 数据访问
│  (userRepository)   │
├─────────────────────┤
│  Database 层        │ ← 数据存储
│  (MySQL)            │
└─────────────────────┘

### 设计模式
- 仓库模式（Repository Pattern）
- 依赖注入（Dependency Injection）
- 中间件模式（Middleware Pattern）

### 横切关注点
- 认证：JWT Token 验证
- 日志：winston 日志记录
- 缓存：Redis Session 缓存
```

---

**Step 4: Implementation Details（实现细节）**

```markdown
**4. Implementation Details**
- Key algorithms and data structures
- Error handling and edge cases
- Performance considerations
- Technical debt or improvement areas
```

**具体任务：**
1. 关键算法和数据结构
   - 核心算法是什么
   - 使用了什么数据结构

2. 错误处理和边界情况
   - 如何处理异常
   - 边界情况如何处理

3. 性能考虑
   - 有没有性能优化
   - 是否存在性能瓶颈

4. 技术债务或改进空间
   - 哪些地方可以改进
   - 存在什么技术债务

**示例输出：**
```markdown
## 实现细节

### 关键算法
- 密码验证：bcrypt 比较算法（时间复杂度 O(n)）
- Token 生成：JWT HS256 签名

### 错误处理
- 用户不存在：抛出 UserNotFoundError
- 密码错误：抛出 InvalidCredentialsError
- 数据库异常：全局异常处理器捕获

### 性能优化
- ✅ 使用 Redis 缓存 Session（减少数据库查询）
- ⚠️ 未使用连接池（可能导致性能问题）

### 技术债务
- TODO: 添加密码强度验证
- TODO: 实现登录失败限流
- TODO: 添加双因素认证支持
```

---

#### 输出规范（第 39-51 行）

```markdown
## Output Guidance

Provide a comprehensive analysis that helps developers understand the feature deeply enough to modify or extend it. Include:

- Entry points with file:line references
- Step-by-step execution flow with data transformations
- Key components and their responsibilities
- Architecture insights: patterns, layers, design decisions
- Dependencies (external and internal)
- Observations about strengths, issues, or opportunities
- List of files that you think are absolutely essential to get an understanding of the topic in question

Structure your response for maximum clarity and usefulness. Always include specific file paths and line numbers.
```

**关键要求：**

1. **必须包含文件路径和行号**
   - ✅ 正确：`src/auth.ts:42`
   - ❌ 错误：`在 auth.ts 文件中`

2. **必须列出关键文件清单**
   - 帮助开发者快速定位核心代码

3. **结构清晰、易于理解**
   - 使用标题、列表、代码块
   - 必要时使用图表

**完整输出示例：**

```markdown
# code-explorer 分析报告：用户认证功能

## 一、入口点
- `src/routes/auth.ts:12` - POST /api/login
- `src/routes/auth.ts:28` - POST /api/logout

## 二、执行流程
[详细的数据流图]

## 三、关键组件
1. authController (src/controllers/authController.ts:45)
   职责：处理登录请求
   
2. authService (src/services/authService.ts:78)
   职责：认证逻辑

## 四、架构洞察
[分层架构图]

## 五、依赖关系
- 内部：userRepository, tokenService
- 外部：bcrypt, jsonwebtoken

## 六、优势与问题
### 优势
- 使用成熟的 JWT 方案
- 分层清晰

### 问题
- 缺少登录失败限流
- 密码强度验证不足

## 七、必读文件清单
1. src/services/authService.ts (L1-L234)
2. src/middleware/authMiddleware.ts (L1-L156)
3. src/config/auth.js (L1-L45)
```

---

### 2.3 实战使用指南

#### 何时使用 code-explorer？

**场景 1：新功能开发前**
```
用户：我要开发一个积分系统
Claude: 好的，让我先启动 code-explorer 分析现有的奖励功能
        → 启动 code-explorer 分析类似功能
```

**场景 2：理解复杂代码**
```
用户：这段代码我看不懂，帮我分析一下
Claude: 我来启动 code-explorer 追踪这段代码的执行流程
        → 启动 code-explorer 详细分析
```

**场景 3：Bug 排查**
```
用户：登录功能出问题了，帮我看看
Claude: 让我启动 code-explorer 追踪登录流程
        → 启动 code-explorer 找出问题所在
```

---

#### 如何启动 code-explorer？

**方式 1：在 feature-dev 流程中自动启动**
```bash
/feature-dev 开发用户积分系统
# Phase 2 会自动启动 2-3 个 code-explorer agents
```

**方式 2：手动启动**
```
"请启动 code-explorer 分析这个项目的认证功能"
```

**方式 3：指定分析重点**
```
"启动 code-explorer，重点关注安全方面的实现"
```

---

#### 输出质量评估

**优秀的分析报告特征：**
- ✅ 包含具体的文件路径和行号
- ✅ 有清晰的流程图或架构图
- ✅ 列出了必读文件清单
- ✅ 指出了优势和问题
- ✅ 提供了改进建议

**低质量的报告特征：**
- ❌ 只有泛泛而谈的描述
- ❌ 没有具体的文件引用
- ❌ 缺少实际代码示例
- ❌ 没有指出问题和改进方向

---

## 🏗️ 三、code-architect Agent 深度解析

### 3.1 配置文件完整内容

**文件位置：** `plugins/feature-dev/agents/code-architect.md`

```markdown
---
name: code-architect
description: Designs feature architectures by analyzing existing codebase patterns and conventions, then providing comprehensive implementation blueprints with specific files to create/modify, component designs, data flows, and build sequences
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch, KillShell, BashOutput
model: sonnet
color: green
---

You are a senior software architect who delivers comprehensive, actionable architecture blueprints by deeply understanding codebases and making confident architectural decisions.

## Core Process

**1. Codebase Pattern Analysis**
Extract existing patterns, conventions, and architectural decisions. Identify the technology stack, module boundaries, abstraction layers, and CLAUDE.md guidelines. Find similar features to understand established approaches.

**2. Architecture Design**
Based on patterns found, design the complete feature architecture. Make decisive choices - pick one approach and commit. Ensure seamless integration with existing code. Design for testability, performance, and maintainability.

**3. Complete Implementation Blueprint**
Specify every file to create or modify, component responsibilities, integration points, and data flow. Break implementation into clear phases with specific tasks.

## Output Guidance

Deliver a decisive, complete architecture blueprint that provides everything needed for implementation. Include:

- **Patterns & Conventions Found**: Existing patterns with file:line references, similar features, key abstractions
- **Architecture Decision**: Your chosen approach with rationale and trade-offs
- **Component Design**: Each component with file path, responsibilities, dependencies, and interfaces
- **Implementation Map**: Specific files to create/modify with detailed change descriptions
- **Data Flow**: Complete flow from entry points through transformations to outputs
- **Build Sequence**: Phased implementation steps as a checklist
- **Critical Details**: Error handling, state management, testing, performance, and security considerations

Make confident architectural choices rather than presenting multiple options. Be specific and actionable - provide file paths, function names, and concrete steps.
```

---

### 3.2 配置文件逐行解读

#### 头部元数据（第 1-7 行）

```yaml
---
name: code-architect                   # Agent 名称
description: Designs feature...       # 功能描述
tools: Glob, Grep, LS, Read, ...      # 可用工具列表
model: sonnet                         # 使用的 AI 模型
color: green                          # UI 显示颜色
---
```

**与 code-explorer 的对比：**

| 特性 | code-explorer | code-architect |
|------|---------------|----------------|
| **角色** | 代码分析师 | 软件架构师 |
| **目标** | 理解现有代码 | 设计新架构 |
| **颜色** | yellow | green |
| **使用阶段** | Phase 2 | Phase 4 |

---

#### 角色定义（第 9-10 行）

```markdown
You are a senior software architect who delivers comprehensive, actionable architecture blueprints by deeply understanding codebases and making confident architectural decisions.
```

**解读：**

- **角色定位**：资深软件架构师
- **核心能力**：
  - 深入理解代码库
  - 做出果断的架构决策
- **产出物**：全面、可执行的架构蓝图

**与 code-explorer 的区别：**
> code-explorer 是"分析过去"（现有代码如何工作）
> 
> code-architect 是"设计未来"（新代码应该如何写）

---

#### 工作流程 - 三步曲（第 11-20 行）

**Step 1: Codebase Pattern Analysis（代码库模式分析）**

```markdown
**1. Codebase Pattern Analysis**
Extract existing patterns, conventions, and architectural decisions. Identify the technology stack, module boundaries, abstraction layers, and CLAUDE.md guidelines. Find similar features to understand established approaches.
```

**具体任务：**

1. **提取现有模式**
   - 代码组织模式（如 MVC、分层架构）
   - 命名约定（如驼峰命名、下划线命名）
   - 文件组织方式（按功能 or 按类型）

2. **识别技术栈**
   - 使用的框架（Express、React 等）
   - 数据库（MySQL、MongoDB 等）
   - 第三方库

3. **确定模块边界**
   - 各模块的职责划分
   - 模块之间的依赖关系

4. **阅读 CLAUDE.md**
   - 项目特定的编码规范
   - 架构原则

5. **寻找类似功能**
   - 参考已有的实现方式

**示例输出：**
```markdown
## 代码库模式分析

### 技术栈
- 后端：Node.js + Express.js
- 数据库：MySQL + Sequelize ORM
- 缓存：Redis

### 架构模式
- 分层架构：Controller → Service → Repository
- 中间件模式：用于认证、日志

### 命名约定
- 文件名：camelCase（如 authService.js）
- 类名：PascalCase（如 UserService）
- 常量：UPPER_SNAKE_CASE（如 MAX_RETRY）

### 项目规范（CLAUDE.md）
- 必须编写单元测试
- 使用 async/await 而非回调
- 错误统一处理

### 类似功能参考
- 用户认证功能（src/auth/）
  - 使用 JWT Token
  - bcrypt 密码加密
  - Redis Session 存储
```

---

**Step 2: Architecture Design（架构设计）**

```markdown
**2. Architecture Design**
Based on patterns found, design the complete feature architecture. Make decisive choices - pick one approach and commit. Ensure seamless integration with existing code. Design for testability, performance, and maintainability.
```

**关键原则：**

⚠️ **Make decisive choices - pick one approach and commit**
> 做出果断选择，选定一个方案并坚持

**这是 code-architect 最重要的原则！**

**错误做法 ❌：**
```markdown
有三种方案：
方案 1：...
方案 2：...
方案 3：...
你选一个吧。
```

**正确做法 ✅：**
```markdown
基于分析，我推荐方案 2：

理由：
1. 符合项目现有架构模式
2. 可维护性最好
3. 团队熟悉相关技术

实施方案 2 的具体步骤：
...
```

**设计要求：**

1. **无缝集成现有代码**
   - 不要推翻重来
   - 遵循现有模式

2. **为可测试性设计**
   - 依赖注入
   - 单一职责

3. **为性能设计**
   - 缓存策略
   - 数据库索引

4. **为可维护性设计**
   - 清晰的代码结构
   - 完善的注释

**示例输出：**
```markdown
## 架构设计

### 设计方案
采用务实平衡方案：

1. 新建 OAuthProvider 抽象类
2. 集成到现有 AuthService
3. 最小化重构

### 架构图
```
┌──────────────┐
│   Client     │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ AuthController │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ AuthService  │←── OAuthProvider (新增)
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ UserRepository│
└──────┬───────┘
       │
       ↓
┌──────────────┐
│   MySQL      │
└──────────────┘
```

### 推荐理由
1. 符合现有分层架构
2. OAuth 逻辑独立封装
3. 不影响现有认证流程
```

---

**Step 3: Complete Implementation Blueprint（实施蓝图）**

```markdown
**3. Complete Implementation Blueprint**
Specify every file to create or modify, component responsibilities, integration points, and data flow. Break implementation into clear phases with specific tasks.
```

**具体内容：**

1. **文件清单**
   - 要创建的文件
   - 要修改的文件

2. **组件职责**
   - 每个组件负责什么
   - 组件之间的关系

3. **集成点**
   - 如何与现有代码集成
   - 需要修改的接口

4. **数据流**
   - 数据如何流动
   - 在哪里存储

5. **分阶段实施**
   - Phase 1: ...
   - Phase 2: ...

**示例输出：**
```markdown
## 实施蓝图

### 文件清单

#### 新建文件
1. `src/auth/OAuthProvider.ts`
   - OAuth 提供者基类
   - 约 150 行代码

2. `src/auth/providers/GoogleOAuthProvider.ts`
   - Google OAuth 实现
   - 约 80 行代码

3. `src/auth/providers/GitHubOAuthProvider.ts`
   - GitHub OAuth 实现
   - 约 80 行代码

#### 修改文件
4. `src/auth/AuthService.ts`
   - 添加 OAuth 方法
   - 修改约 30 行

5. `src/routes/auth.ts`
   - 添加 OAuth 路由
   - 修改约 20 行

### 组件职责

#### OAuthProvider (抽象类)
职责：定义 OAuth 提供者接口
依赖：无
被依赖：AuthService

#### GoogleOAuthProvider
职责：实现 Google OAuth 逻辑
依赖：OAuthProvider, axios
被依赖：AuthService

### 集成点

1. AuthService 集成 OAuthProvider
   ```typescript
   class AuthService {
     async loginWithOAuth(provider: string, code: string) {
       const providerImpl = this.getProvider(provider);
       const userInfo = await providerImpl.getUserInfo(code);
       // ...
     }
   }
   ```

2. 路由集成
   ```typescript
   router.get('/oauth/:provider', oauthController.callback);
   ```

### 数据流

1. 用户点击"使用 Google 登录"
2. 跳转到 Google OAuth 授权页
3. 用户授权后返回 authorization code
4. 后端用 code 换取 access_token
5. 用 access_token 获取用户信息
6. 创建或更新本地用户
7. 生成 JWT Token 返回

### 实施阶段

#### Phase 1: 基础架构（预计 2 小时）
- [ ] 创建 OAuthProvider 抽象类
- [ ] 定义 Provider 接口
- [ ] 编写单元测试

#### Phase 2: Google 实现（预计 3 小时）
- [ ] 创建 GoogleOAuthProvider
- [ ] 实现 getUserInfo 方法
- [ ] 集成到 AuthService
- [ ] 编写端到端测试

#### Phase 3: GitHub 实现（预计 2 小时）
- [ ] 创建 GitHubOAuthProvider
- [ ] 复用测试用例
- [ ] 集成测试

#### Phase 4: 前端集成（预计 2 小时）
- [ ] 添加 OAuth 登录按钮
- [ ] 处理回调
- [ ] UI 测试
```

---

#### 输出规范（第 22-34 行）

```markdown
## Output Guidance

Deliver a decisive, complete architecture blueprint that provides everything needed for implementation. Include:

- **Patterns & Conventions Found**: Existing patterns with file:line references, similar features, key abstractions
- **Architecture Decision**: Your chosen approach with rationale and trade-offs
- **Component Design**: Each component with file path, responsibilities, dependencies, and interfaces
- **Implementation Map**: Specific files to create/modify with detailed change descriptions
- **Data Flow**: Complete flow from entry points through transformations to outputs
- **Build Sequence**: Phased implementation steps as a checklist
- **Critical Details**: Error handling, state management, testing, performance, and security considerations

Make confident architectural choices rather than presenting multiple options. Be specific and actionable - provide file paths, function names, and concrete steps.
```

**关键要求：**

1. **必须做出明确选择**
   - ❌ 不要罗列多个方案让用户选
   - ✅ 给出推荐方案并说明理由

2. **必须具体可执行**
   - ❌ "创建一个服务类"
   - ✅ "创建 src/auth/AuthService.ts，包含以下方法..."

3. **必须完整详尽**
   - 覆盖所有必要的细节
   - 错误处理、测试、性能、安全

**完整输出结构示例：**

```markdown
# code-architect 架构蓝图：OAuth 认证功能

## 一、模式与约定发现

### 现有模式
- Controller → Service → Repository 分层
- 使用 TypeScript 严格模式
- 单元测试覆盖率 > 90%

### 类似功能参考
- JWT 认证（src/auth/jwt/）
  - 使用 passport-jwt
  - Token 有效期 7 天

## 二、架构决策

### 推荐方案：OAuthProvider 抽象

#### 方案说明
创建 OAuthProvider 抽象类，不同 OAuth 平台实现各自 Provider

#### 选择理由
1. 符合开闭原则（对扩展开放，对修改关闭）
2. 便于添加新的 OAuth 平台
3. 与现有架构一致

#### 权衡分析
优点：
- 代码复用性高
- 易于测试和维护

缺点：
- 初期设计复杂度略高

## 三、组件设计

### OAuthProvider（抽象类）
文件：src/auth/OAuthProvider.ts
职责：定义 OAuth 提供者接口
方法：
- authorize(): string - 返回授权 URL
- getToken(code: string): Promise<string>
- getUserInfo(token: string): Promise<UserInfo>

### GoogleOAuthProvider
文件：src/auth/providers/GoogleOAuthProvider.ts
继承：OAuthProvider
依赖：axios, crypto

## 四、实施地图

[详细的文件清单和修改说明]

## 五、数据流

[完整的数据流图]

## 六、构建序列

[分阶段实施清单]

## 七、关键细节

### 错误处理
- OAuth 授权失败：重定向到登录页，显示错误消息
- Token 交换失败：记录日志，返回 500 错误

### 状态管理
- 使用 state 参数防止 CSRF 攻击
- state 存储在 Redis，5 分钟过期

### 测试策略
- 单元测试：Mock OAuth API
- 集成测试：使用 OAuth 沙箱环境

### 性能优化
- 缓存用户信息（Redis，30 分钟）
- 并发限制（每 IP 每秒最多 10 次请求）

### 安全考虑
- 验证 redirect_uri
- 使用 PKCE 增强安全性
- 敏感信息加密存储
```

---

### 3.3 实战使用指南

#### 何时使用 code-architect？

**场景 1：新功能设计**
```
用户：我要开发积分系统，怎么设计架构？
Claude: 让我启动 code-architect 为你设计完整的架构
        → 启动 code-architect
```

**场景 2：重构现有代码**
```
用户：这段代码太乱了，想重构一下
Claude: 我来启动 code-architect 设计重构方案
        → 启动 code-architect
```

**场景 3：技术选型**
```
用户：应该用 MongoDB 还是 MySQL？
Claude: 让我启动 code-architect 分析项目需求并给出建议
        → 启动 code-architect
```

---

#### 如何启动 code-architect？

**方式 1：在 feature-dev 流程中自动启动**
```bash
/feature-dev 开发用户积分系统
# Phase 4 会自动启动 2-3 个 code-architect agents
```

**方式 2：手动启动**
```
"请启动 code-architect 设计这个功能的架构"
```

**方式 3：指定设计重点**
```
"启动 code-architect，重点关注性能和可扩展性"
```

---

#### 输出质量评估

**优秀的架构蓝图特征：**
- ✅ 做出了明确的方案选择
- ✅ 提供了详细的实施步骤
- ✅ 包含了所有必要的细节
- ✅ 考虑了错误处理、测试、性能、安全
- ✅ 给出了具体的文件路径和函数名

**低质量的蓝图特征：**
- ❌ 罗列多个方案不做选择
- ❌ 只有高层描述，缺少细节
- ❌ 没有具体的实施步骤
- ❌ 忽略了错误处理和安全考虑

---

## 🔍 四、code-reviewer Agent 深度解析

### 4.1 配置文件完整内容

**文件位置：** `plugins/feature-dev/agents/code-reviewer.md`

```markdown
---
name: code-reviewer
description: Reviews code for bugs, logic errors, security vulnerabilities, code quality issues, and adherence to project conventions, using confidence-based filtering to report only high-priority issues that truly matter
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch, KillShell, BashOutput
model: sonnet
color: red
---

You are an expert code reviewer specializing in modern software development across multiple languages and frameworks. Your primary responsibility is to review code against project guidelines in CLAUDE.md with high precision to minimize false positives.

## Review Scope

By default, review unstaged changes from `git diff`. The user may specify different files or scope to review.

## Core Review Responsibilities

**Project Guidelines Compliance**: Verify adherence to explicit project rules (typically in CLAUDE.md or equivalent) including import patterns, framework conventions, language-specific style, function declarations, error handling, logging, testing practices, platform compatibility, and naming conventions.

**Bug Detection**: Identify actual bugs that will impact functionality - logic errors, null/undefined handling, race conditions, memory leaks, security vulnerabilities, and performance problems.

**Code Quality**: Evaluate significant issues like code duplication, missing critical error handling, accessibility problems, and inadequate test coverage.

## Confidence Scoring

Rate each potential issue on a scale from 0-100:

- **0**: Not confident at all. This is a false positive that doesn't stand up to scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. If stylistic, it wasn't explicitly called out in project guidelines.
- **50**: Moderately confident. This is a real issue, but might be a nitpick or not happen often in practice. Not very important relative to the rest of the changes.
- **75**: Highly confident. Double-checked and verified this is very likely a real issue that will be hit in practice. The existing approach is insufficient. Important and will directly impact functionality, or is directly mentioned in project guidelines.
- **100**: Absolutely certain. Confirmed this is definitely a real issue that will happen frequently in practice. The evidence directly confirms this.

**Only report issues with confidence ≥ 80.** Focus on issues that truly matter - quality over quantity.

## Output Guidance

Start by clearly stating what you're reviewing. For each high-confidence issue, provide:

- Clear description with confidence score
- File path and line number
- Specific project guideline reference or bug explanation
- Concrete fix suggestion

Group issues by severity (Critical vs Important). If no high-confidence issues exist, confirm the code meets standards with a brief summary.

Structure your response for maximum actionability - developers should know exactly what to fix and why.
```

---

### 4.2 配置文件逐行解读

#### 头部元数据（第 1-7 行）

```yaml
---
name: code-reviewer                  # Agent 名称
description: Reviews code for...    # 功能描述
tools: Glob, Grep, LS, Read, ...    # 可用工具列表
model: sonnet                       # 使用的 AI 模型
color: red                          # UI 显示颜色
---
```

**颜色含义：**
- 🔴 **red** - 表示"审查"、"警告"、"问题"
- 🟡 **yellow** - 表示"探索"、"分析"
- 🟢 **green** - 表示"建设"、"设计"

---

#### 角色定义（第 9-10 行）

```markdown
You are an expert code reviewer specializing in modern software development across multiple languages and frameworks. Your primary responsibility is to review code against project guidelines in CLAUDE.md with high precision to minimize false positives.
```

**解读：**

- **角色定位**：代码审查专家
- **核心职责**：根据项目规范（CLAUDE.md）审查代码
- **关键目标**：高精度，最小化误报（false positives）

**重要原则：**
> 宁可漏报，不可误报！
> 
> 只报告真正重要的问题，而不是吹毛求疵。

---

#### 审查范围（第 11-13 行）

```markdown
## Review Scope

By default, review unstaged changes from `git diff`. The user may specify different files or scope to review.
```

**默认行为：**
- 审查 `git diff` 中的未暂存更改
- 即当前工作区与暂存区的差异

**可指定范围：**
- 审查特定文件
- 审查整个项目
- 审查某次提交

---

#### 审查职责 - 三大维度（第 15-21 行）

**维度 1: Project Guidelines Compliance（项目约定遵守）**

```markdown
**Project Guidelines Compliance**: Verify adherence to explicit project rules (typically in CLAUDE.md or equivalent) including import patterns, framework conventions, language-specific style, function declarations, error handling, logging, testing practices, platform compatibility, and naming conventions.
```

**审查清单：**

1. **导入模式**
   - ✅ `import { something } from 'module'`
   - ❌ `const something = require('module')`（如果项目规定用 ES6）

2. **框架约定**
   - ✅ React 组件使用 PascalCase
   - ❌ React 组件使用 camelCase

3. **语言风格**
   - ✅ TypeScript 使用类型注解
   - ❌ 任何隐式的 any 类型

4. **函数声明**
   - ✅ 使用箭头函数或函数声明
   - ❌ 混用两种方式

5. **错误处理**
   - ✅ try-catch 包裹异步操作
   - ❌ 忽略 Promise rejection

6. **日志记录**
   - ✅ 使用统一的日志库（如 winston）
   - ❌ console.log 调试代码

7. **测试实践**
   - ✅ 每个公共函数都有测试
   - ❌ 测试覆盖率 < 90%

8. **平台兼容性**
   - ✅ 跨平台兼容的代码
   - ❌ 仅适用于特定平台的代码

9. **命名约定**
   - ✅ 变量使用 camelCase
   - ❌ 变量使用 PascalCase

**示例输出：**
```markdown
## 项目约定遵守审查

### 违反 CLAUDE.md 的问题

1. ❌ 第 23 行：使用了 console.log
   文件：src/auth/login.ts:23
   规范：CLAUDE.md 第 5 条 - 必须使用 logger 而非 console.log
   修复：改为 logger.info() 或 logger.error()

2. ❌ 第 45 行：缺少类型注解
   文件：src/utils/helper.ts:45
   规范：CLAUDE.md 第 3 条 - TypeScript 必须显式类型注解
   修复：添加类型注解 `function foo(data: UserData): Promise<void>`
```

---

**维度 2: Bug Detection（Bug 检测）**

```markdown
**Bug Detection**: Identify actual bugs that will impact functionality - logic errors, null/undefined handling, race conditions, memory leaks, security vulnerabilities, and performance problems.
```

**常见 Bug 类型：**

1. **逻辑错误**
   ```javascript
   // ❌ 错误：条件判断反了
   if (user.age > 18) {
     return false; // 未成年不能访问
   }
   
   // ✅ 正确
   if (user.age < 18) {
     return false;
   }
   ```

2. **null/undefined 处理**
   ```javascript
   // ❌ 错误：可能抛出 TypeError
   const name = user.profile.name;
   
   // ✅ 正确：使用可选链
   const name = user?.profile?.name;
   ```

3. **竞态条件**
   ```javascript
   // ❌ 错误：并发读取可能导致数据不一致
   let count = 0;
   async function increment() {
     count++;
   }
   
   // ✅ 正确：使用原子操作或锁
   ```

4. **内存泄漏**
   ```javascript
   // ❌ 错误：事件监听器未清理
   element.addEventListener('click', handler);
   
   // ✅ 正确：在清理时移除监听器
   element.removeEventListener('click', handler);
   ```

5. **安全漏洞**
   ```javascript
   // ❌ 错误：SQL 注入风险
   db.query(`SELECT * FROM users WHERE id = ${userId}`);
   
   // ✅ 正确：参数化查询
   db.query('SELECT * FROM users WHERE id = ?', [userId]);
   ```

6. **性能问题**
   ```javascript
   // ❌ 错误：O(n²) 复杂度
   for (let i = 0; i < arr.length; i++) {
     for (let j = 0; j < arr.length; j++) {
       // ...
     }
   }
   
   // ✅ 正确：使用哈希表 O(n)
   const map = new Map();
   ```

**示例输出：**
```markdown
## Bug 检测

### 严重 Bug

1. 🐛 第 67 行：SQL 注入漏洞
   文件：src/user/repository.ts:67
   问题：字符串拼接 SQL 查询
   影响：攻击者可注入恶意 SQL
   修复：使用参数化查询
   
   错误代码：
   ```javascript
   db.query(`SELECT * FROM users WHERE email = '${email}'`);
   ```
   
   修复代码：
   ```javascript
   db.query('SELECT * FROM users WHERE email = ?', [email]);
   ```

2. 🐛 第 89 行：未处理的 null 值
   文件：src/auth/service.ts:89
   问题：访问可能为 null 的对象属性
   影响：抛出 TypeError，导致请求失败
   修复：添加空值检查或使用可选链
```

---

**维度 3: Code Quality（代码质量）**

```markdown
**Code Quality**: Evaluate significant issues like code duplication, missing critical error handling, accessibility problems, and inadequate test coverage.
```

**质量问题类型：**

1. **代码重复**
   ```javascript
   // ❌ 错误：三段几乎相同的代码
   function processUser(user) { ... }
   function processAdmin(admin) { ... }
   function processGuest(guest) { ... }
   
   // ✅ 正确：提取公共逻辑
   function processPerson(person, type) { ... }
   ```

2. **缺少关键错误处理**
   ```javascript
   // ❌ 错误：没有错误处理
   async function fetchData() {
     const response = await fetch(url);
     return response.json();
   }
   
   // ✅ 正确：完整的错误处理
   async function fetchData() {
     try {
       const response = await fetch(url);
       if (!response.ok) throw new Error('Network error');
       return response.json();
     } catch (error) {
       logger.error('Fetch failed:', error);
       throw error;
     }
   }
   ```

3. **可访问性问题**
   ```html
   <!-- ❌ 错误：图片缺少 alt 属性 -->
   <img src="logo.png">
   
   <!-- ✅ 正确：添加描述性 alt -->
   <img src="logo.png" alt="公司 Logo">
   ```

4. **测试覆盖不足**
   ```
   ❌ 错误：核心业务逻辑缺少测试
   ✅ 正确：单元测试覆盖率 > 90%
   ```

**示例输出：**
```markdown
## 代码质量审查

### 代码重复

1. ⚠️ 第 120-145 行：重复的验证逻辑
   文件：src/validators/*.ts
   问题：三个文件中有相似的验证代码
   建议：提取公共验证函数到 utils/validator.ts
   
   重复代码：
   ```javascript
   // src/validators/email.ts:12
   if (!email.includes('@')) {
     throw new ValidationError('Invalid email');
   }
   
   // src/validators/user.ts:45
   if (!email.includes('@')) {
     throw new ValidationError('Invalid email');
   }
   ```

### 缺少错误处理

2. ⚠️ 第 78 行：异步操作缺少 try-catch
   文件：src/service/payment.ts:78
   问题：支付 API 调用未捕获异常
   影响：支付失败时应用崩溃
   建议：添加 try-catch 块

### 测试覆盖

3. ℹ️ 测试覆盖率不足
   当前覆盖率：78%
   目标覆盖率：90%
   缺失测试：
   - paymentService.processPayment()
   - userService.deleteUser()
```

---

#### 置信度评分机制（第 23-33 行）⭐

**这是 code-reviewer 最核心的机制！**

```markdown
## Confidence Scoring

Rate each potential issue on a scale from 0-100:

- **0**: Not confident at all. This is a false positive that doesn't stand up to scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. If stylistic, it wasn't explicitly called out in project guidelines.
- **50**: Moderately confident. This is a real issue, but might be a nitpick or not happen often in practice. Not very important relative to the rest of the changes.
- **75**: Highly confident. Double-checked and verified this is very likely a real issue that will be hit in practice. The existing approach is insufficient. Important and will directly impact functionality, or is directly mentioned in project guidelines.
- **100**: Absolutely certain. Confirmed this is definitely a real issue that will happen frequently in practice. The evidence directly confirms this.

**Only report issues with confidence ≥ 80.** Focus on issues that truly matter - quality over quantity.
```

**评分标准详解：**

| 分数 | 含义 | 是否报告 | 示例 |
|------|------|----------|------|
| **0** | 完全不是问题 | ❌ | 误报、已存在的问题 |
| **25** | 可能是问题 | ❌ | 风格问题，规范未明确要求 |
| **50** | 中等确信 | ❌ | 小问题，不常发生，不重要 |
| **75** | 高度确信 | ⚠️ 临界值 | 很可能是真问题，会影响功能 |
| **≥80** | **报告阈值** | ✅ | **只报告这个级别以上的问题** |
| **100** | 绝对确定 | ✅ | 证据确凿，会频繁发生 |

**为什么设置 80 分的阈值？**

1. **避免干扰**：报告太多小问题会分散注意力
2. **聚焦重点**：只关注真正重要的问题
3. **提高效率**：开发者时间宝贵，解决关键问题
4. **建立信任**：报告的都是真问题，不会被忽视

**评分示例：**

```javascript
// 示例 1：SQL 注入
db.query(`SELECT * FROM users WHERE id = ${userId}`);

// 评分：100 分
// 理由：证据确凿的 SQL 注入漏洞，会频繁发生
```

```javascript
// 示例 2：缺少类型注解
function processData(data) { ... }

// 评分：75 分
// 理由：违反 TypeScript 规范，但可能不影响运行
```

```javascript
// 示例 3：变量命名风格
const userData = ...;  // 应该用 user_data

// 评分：25 分
// 理由：风格问题，规范未明确要求，不报告
```

---

#### 输出规范（第 35-46 行）

```markdown
## Output Guidance

Start by clearly stating what you're reviewing. For each high-confidence issue, provide:

- Clear description with confidence score
- File path and line number
- Specific project guideline reference or bug explanation
- Concrete fix suggestion

Group issues by severity (Critical vs Important). If no high-confidence issues exist, confirm the code meets standards with a brief summary.

Structure your response for maximum actionability - developers should know exactly what to fix and why.
```

**输出结构要求：**

1. **开头明确说明审查对象**
   ```markdown
   正在审查：用户认证功能修改
   审查范围：src/auth/ 目录下的未暂存更改
   ```

2. **每个问题包含四个要素**
   - 清晰描述 + 置信度分数
   - 文件路径和行号
   - 具体规范引用或 Bug 解释
   - 具体修复建议

3. **按严重程度分组**
   ```markdown
   ### Critical（严重问题）
   1. ...
   
   ### Important（重要问题）
   1. ...
   ```

4. **如果没有高置信度问题**
   ```markdown
   ## 审查结论

   ✅ 代码符合项目标准
   
   本次审查未发现高置信度问题（≥80 分）。
   代码质量良好，遵循了项目规范和最佳实践。
   ```

**完整输出示例：**

```markdown
# code-reviewer 审查报告

## 审查概况
- 审查对象：用户登录功能修改
- 审查范围：git diff 中的未暂存更改
- 审查文件：3 个文件，共 234 行变更

---

## Critical（严重问题）

### 1. 🐛 SQL 注入漏洞
**置信度：100/100**

📍 位置：src/auth/repository.ts:67

📋 问题描述：
使用字符串拼接构建 SQL 查询，存在 SQL 注入风险

⚠️ 影响：
攻击者可构造恶意输入获取数据库权限

📖 规范引用：
CLAUDE.md 第 8 条 - 所有数据库查询必须使用参数化查询

🔧 修复建议：
```javascript
// 错误代码
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// 修复代码
db.query('SELECT * FROM users WHERE email = ?', [email]);
```

---

### 2. 🐛 密码未加密存储
**置信度：100/100**

📍 位置：src/auth/service.ts:89

📋 问题描述：
用户密码以明文形式存储到数据库

⚠️ 影响：
数据库泄露会导致所有用户密码暴露

📖 规范引用：
CLAUDE.md 第 12 条 - 密码必须使用 bcrypt 加密

🔧 修复建议：
```javascript
// 错误代码
await userRepository.save({ email, password });

// 修复代码
const hashedPassword = await bcrypt.hash(password, 10);
await userRepository.save({ email, password: hashedPassword });
```

---

## Important（重要问题）

### 3. ⚠️ 缺少登录失败限流
**置信度：85/100**

📍 位置：src/auth/controller.ts:45

📋 问题描述：
登录接口没有失败次数限制

⚠️ 影响：
可能被暴力破解攻击

📖 规范引用：
CLAUDE.md 第 15 条 - 认证接口必须实现限流

🔧 修复建议：
使用 express-rate-limit 中间件：
```javascript
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 5, // 最多 5 次尝试
  message: 'Too many login attempts, please try again later'
});

router.post('/login', limiter, authController.login);
```

---

## 审查结论

❌ **不建议提交**

发现 2 个严重问题，必须在提交前修复：
1. SQL 注入漏洞（100 分）
2. 密码明文存储（100 分）

建议修复后重新审查。
```

---

### 4.3 实战使用指南

#### 何时使用 code-reviewer？

**场景 1：代码提交前**
```
用户：我准备提交代码了，帮我审查一下
Claude: 让我启动 code-reviewer 进行全面检查
        → 启动 code-reviewer
```

**场景 2：PR 审查**
```
用户：帮我 review 一下这个 PR
Claude: 我来启动 code-reviewer 审查代码变更
        → 启动 code-reviewer
```

**场景 3：学习代码审查**
```
用户：我想学习如何做代码审查
Claude: 让我启动 code-reviewer 演示审查过程
        → 启动 code-reviewer
```

---

#### 如何启动 code-reviewer？

**方式 1：在 feature-dev 流程中自动启动**
```bash
/feature-dev 开发用户积分系统
# Phase 6 会自动启动 3 个 code-reviewer agents
```

**方式 2：手动启动**
```
"请启动 code-reviewer 审查我的代码"
```

**方式 3：指定审查重点**
```
"启动 code-reviewer，重点关注安全问题"
"启动 code-reviewer，检查性能问题"
```

---

#### 输出质量评估

**优秀的审查报告特征：**
- ✅ 只报告高置信度问题（≥80 分）
- ✅ 问题描述清晰具体
- ✅ 提供了修复代码示例
- ✅ 按严重程度分组
- ✅ 给出了明确的结论和建议

**低质量的审查报告特征：**
- ❌ 报告了大量琐碎问题
- ❌ 缺少具体的修复建议
- ❌ 没有引用项目规范
- ❌ 问题描述模糊不清

---

## 🎯 五、三个 Agent 的协同配合

### 5.1 完整协作流程

```
用户需求：开发一个新功能
    ↓
Phase 1: Discovery（需求发现）
    ↓
Phase 2: Exploration（代码库探索）
    ├─ 启动 code-explorer × 2-3
    │  └─ 分析现有代码和模式
    └─ 整合分析报告
    ↓
Phase 3: Clarification（需求澄清）
    ↓
Phase 4: Architecture（架构设计）
    ├─ 启动 code-architect × 2-3
    │  └─ 设计多种实现方案
    └─ 整合并推荐方案
    ↓
Phase 5: Implementation（实现）
    ↓
Phase 6: Review（质量审查）
    ├─ 启动 code-reviewer × 3
    │  └─ 审查代码质量
    └─ 整合审查结果
    ↓
Phase 7: Summary（总结）
```

---

### 5.2 三个 Agent 的对比总结

| 特性 | code-explorer | code-architect | code-reviewer |
|------|---------------|----------------|---------------|
| **角色** | 代码分析师 | 软件架构师 | 质量审查员 |
| **颜色** | 🟡 yellow | 🟢 green | 🔴 red |
| **使用阶段** | Phase 2 | Phase 4 | Phase 6 |
| **目标** | 理解现有代码 | 设计新架构 | 审查代码质量 |
| **工作方式** | 追踪代码流 | 设计蓝图 | 查找问题 |
| **输出** | 分析报告 | 架构蓝图 | 审查报告 |
| **关键技能** | 代码追踪 | 架构设计 | 问题检测 |
| **置信度机制** | 无 | 无 | 有（≥80 才报告） |

---

### 5.3 实战配合示例

**案例：开发 OAuth 登录功能**

```
Phase 2: Exploration
    ↓
    启动 code-explorer #1: "分析现有的认证功能"
    启动 code-explorer #2: "映射认证架构"
    
    输出：
    - 现有认证使用 JWT Token
    - 分层架构：Controller → Service → Repository
    - 密码使用 bcrypt 加密
    
    ↓
Phase 4: Architecture
    ↓
    启动 code-architect #1: "最小改动方案"
    启动 code-architect #2: "整洁架构方案"
    
    输出：
    - 推荐方案：创建 OAuthProvider 抽象
    - 详细的实施蓝图
    - 完整的文件清单
    
    ↓
Phase 5: Implementation
    ↓
    按照架构蓝图编码实现
    
    ↓
Phase 6: Review
    ↓
    启动 code-reviewer #1: "简洁性/DRY"
    启动 code-reviewer #2: "Bug/正确性"
    启动 code-reviewer #3: "约定遵守"
    
    输出：
    - 发现 2 个严重问题（SQL 注入、密码未加密）
    - 发现 3 个重要问题（缺少限流、日志不规范）
    - 建议修复后再提交
```

---

## 📚 六、学习资源与进阶

### 6.1 官方文档

- [feature-dev 插件源码](../../plugins/feature-dev/)
- [Agent 配置文件](../../plugins/feature-dev/agents/)
- [CLAUDE.md 规范](../../CLAUDE.md)

### 6.2 实践建议

**初学者练习：**

1. **练习 1：阅读 Agent 配置**
   - 仔细阅读三个 Agent 的完整配置
   - 理解每个字段的作用

2. **练习 2：观察 Agent 输出**
   - 使用 /feature-dev 完成一个小功能
   - 观察每个 Agent 的输出

3. **练习 3：手动启动 Agent**
   - 尝试手动启动不同的 Agent
   - 比较输出的差异

**进阶练习：**

4. **练习 4：自定义 Agent**
   - 基于 code-reviewer 创建专用审查 Agent
   - 针对特定领域（如安全、性能）

5. **练习 5：优化 Agent 配置**
   - 根据实际需求调整 Agent 行为
   - 改进输出格式

---

## 🎯 七、常见问题解答

### Q1: 为什么要用三个不同的 Agent？

**答：** 每个 Agent 有不同的专业领域：
- code-explorer 擅长**分析**（理解过去）
- code-architect 擅长**设计**（规划未来）
- code-reviewer 擅长**审查**（保证质量）

就像建筑工程需要：
- 勘察师（勘探地基）→ code-explorer
- 建筑师（设计图纸）→ code-architect
- 质检员（验收质量）→ code-reviewer

---

### Q2: 可以同时启动多个 Agent 吗？

**答：** 可以且推荐！
- Phase 2 通常启动 2-3 个 code-explorers
- Phase 4 通常启动 2-3 个 code-architects
- Phase 6 通常启动 3 个 code-reviewers

并行工作可以：
- 从不同角度分析问题
- 提供更全面的视角
- 大幅缩短时间

---

### Q3: Agent 的输出一定准确吗？

**答：** 不一定，需要批判性思考：
- Agent 可能遗漏某些细节
- Agent 可能误解某些模式
- 需要人工审查和判断

正确做法：
- ✅ 把 Agent 输出作为参考
- ✅ 结合自己的经验判断
- ✅ 有疑问时进一步验证

---

### Q4: 如何选择合适的 Agent？

**答：** 根据需求选择：

| 需求 | 推荐 Agent |
|------|-----------|
| 理解现有代码 | code-explorer |
| 设计新功能 | code-architect |
| 审查代码质量 | code-reviewer |
| 完整开发流程 | 使用 /feature-dev 自动调度 |

---

## 🎊 总结

通过本章学习，你应该已经掌握了：

### ✅ 知识要点

1. **三个核心 Agent 的角色定位**
   - code-explorer：代码分析师
   - code-architect：软件架构师
   - code-reviewer：质量审查员

2. **每个 Agent 的配置详解**
   - 元数据含义
   - 工作流程
   - 输出规范

3. **置信度评分机制**
   - code-reviewer 的核心机制
   - ≥80 分才报告的原则

4. **三个 Agent 的协同配合**
   - 在七阶段流程中的位置
   - 如何协作完成功能开发

### 🚀 实践能力

现在你应该能够：
- ✅ 手动启动合适的 Agent
- ✅ 理解和评估 Agent 的输出
- ✅ 在合适的情境使用合适的 Agent
- ✅ 根据实际需求调整 Agent 配置

### 💡 下一步

1. **动手实践**：使用 /feature-dev 完成一个小功能
2. **观察学习**：注意观察每个 Agent 的输出
3. **反思总结**：思考如何改进 Agent 配置

---

**继续学习：** 接下来学习 [03-阶段转换与进度追踪.md](./03-阶段转换与进度追踪.md)，了解如何把握七阶段流程的节奏！
