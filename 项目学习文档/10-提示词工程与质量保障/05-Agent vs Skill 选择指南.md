# Agent vs Skill 选择指南

> **重要提示**：Agent 和 Skill 是 Claude Code 中两种不同的扩展机制。选择正确的机制对于构建高效的扩展至关重要。本章将帮助你做出正确的选择。

---

## 一、核心区别速览

### 1.1 一句话概括

```
Agent = 主动干活的专家（你叫他做事）
Skill = 默默提供知识的顾问（自动给你建议）
```

### 1.2 核心对比表

| 特性 | Agent | Skill |
|------|-------|-------|
| **文件格式** | `agents/*.md` | `skills/*/SKILL.md` |
| **触发方式** | 显式/隐式调用 | 检测上下文自动触发 |
| **交互模式** | 主动执行任务并返回结果 | 被动提供领域知识支持 |
| **用户感知** | 明确知道使用了某个 Agent | 通常感知不到 Skill 的存在 |
| **输出形式** | 独立的报告/方案/分析 | 融入主 Claude 的回答中 |
| **典型场景** | "用 code-reviewer 审查代码" | 写前端代码时自动获得设计建议 |
| **工作范围** | 有明确的任务边界 | 在特定领域内提供知识 |
| **配置复杂度** | 需要完整的 Frontmatter + System Prompt | 相对简单，主要是触发条件和知识内容 |

---

## 二、快速决策树

```
我需要创建一个扩展
    ↓
是否需要独立执行任务并产出报告？
    ├─ Yes → 使用 Agent
    │   └─ 例如：code-reviewer, code-explorer
    │
    └─ No → 继续向下
        ↓
是否希望自动触发？
    ├─ Yes → 使用 Skill
    │   └─ 例如：frontend-design
    │
    └─ No → 继续向下
        ↓
是否希望用户明确知道在使用扩展？
    ├─ Yes → 使用 Agent
    └─ No → 使用 Skill
```

---

## 三、Agent 详解

### 3.1 什么是 Agent？

**Agent** 是具有特定角色和能力的 AI 助手配置，可以独立执行复杂任务并返回结果。

**关键特点：**
- ✅ **独立执行**：Agent 是一个独立的执行单元
- ✅ **明确触发**：用户或 Claude 明确启动
- ✅ **专门输出**：产生独立的报告或成果
- ✅ **可追踪**：清楚知道哪个 Agent 做了什么

### 3.2 Agent 工作流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 主 Claude
    participant A as Agent
    
    U->>C: "用 code-reviewer 审查这段代码"
    
    Note over C: 识别需要启动 Agent
    C->>A: 启动 code-reviewer Agent
    
    Note over A: 加载配置文件<br/>读取 System Prompt
    
    A->>A: 执行审查任务
    
    A->>C: 返回审查报告
    
    C->>U: 展示结果
    
    Note over U,C,A: Agent 是独立执行的专家
```

### 3.3 何时使用 Agent？

**场景 1：需要主动执行任务**

```
"用 code-explorer 分析一下认证模块的实现"
"让 code-architect 设计一个架构方案"
"启动 code-reviewer 审查这段代码"
```

**场景 2：需要专门的分析和输出**

```
需要一份独立的分析报告
需要一个完整的架构蓝图
需要详细的审查意见清单
```

**场景 3：需要明确的交付物**

```
代码库探索报告
架构设计方案
安全审查清单
性能优化建议
```

**场景 4：需要多 Agent 协作**

```
七阶段流程中的多个 Agent 并行工作
PR 审查时 6 个 Agent 同时分析
```

### 3.4 Agent 配置示例

```markdown
---
name: code-reviewer
description: Use this agent when the user asks to "review code", "check code quality", or "find issues". Examples:

<example>
Context: User finished writing code
user: "Review my code"
assistant: "I'll use the code-reviewer agent to analyze the code."
<commentary>
Code review request triggers the code-reviewer agent.
</commentary>
</example>

model: inherit
color: red
tools: ["Read", "Grep", "Glob"]
---

You are an expert code reviewer specializing in modern software development.

**Your Core Responsibilities:**
1. Review code for bugs and security vulnerabilities
2. Check adherence to project guidelines (CLAUDE.md)
3. Identify code quality issues

**Review Process:**
1. Read code files using Read tool
2. Scan for common issues
3. Rate each issue with confidence score (0-100)
4. Only report issues with confidence ≥ 80

**Output Format:**
## Code Review Report

### Critical Issues
- [file:line] Issue - Fix

### Warnings
- [file:line] Issue - Recommendation
```

---

## 四、Skill 详解

### 4.1 什么是 Skill？

**Skill** 是模块化的知识包，通过提供专业领域知识来增强 Claude 的能力。

**关键特点：**
- ✅ **自动触发**：基于上下文自动激活
- ✅ **后台支持**：不直接面对用户
- ✅ **知识注入**：为主 Claude 提供领域知识
- ✅ **透明集成**：用户感觉不到 Skill 的存在

### 4.2 Skill 工作流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 主 Claude
    participant S as Skill
    
    U->>C: "帮我创建一个登录表单"
    
    Note over C: 检测到前端 UI 任务
    
    C->>S: 自动激活 frontend-design Skill
    
    Note over S: 在后台提供领域知识
    
    S-->>C: silently inject knowledge:<br/>- 大胆的设计选择<br/>- 排版层次<br/>- 动画建议
    
    C->>U: 给出包含 Skill 知识的回答<br/>但不会提到 Skill
    
    Note over U,C,S: Skill 是默默支持的顾问
```

### 4.3 何时使用 Skill？

**场景 1：需要领域知识注入**

```
写前端代码时自动获得设计建议
编写 API 时自动遵循 REST 规范
处理数据时自动考虑安全性
```

**场景 2：希望在日常工作中自动获得指导**

```
不需要每次都显式调用
希望在编码过程中自然获得建议
不想中断工作流
```

**场景 3：不需要额外的交互**

```
只要背景知识支持就好
不需要单独的报告或分析
希望 Skill 默默提供支持
```

**场景 4：增强主 Claude 的能力**

```
让 Claude 在特定领域更专业
提供该领域的最佳实践
注入行业标准和规范
```

### 4.4 Skill 配置示例

```markdown
---
name: frontend-design
description: This skill should be used when the user asks to "create a UI", "design a form", "style a component", "make it pretty", or mentions CSS, HTML, or frontend frameworks.
---

## When to Auto-Invoke

Automatically invoke this skill whenever the user requests or implies:
- Creating new UI components (forms, buttons, cards, modals)
- Designing page layouts
- Styling existing components
- Improving visual appearance

## What You Provide

When invoked, provide guidance on these aspects:

### 1. Bold Design Choices
❌ Avoid: Generic, boring AI-looking designs
✅ Encourage: Bold gradients, unique color combinations

### 2. Typography Hierarchy
- Establish clear visual hierarchy (H1: 2.5rem, H2: 2rem, Body: 1rem)
- Use appropriate font weights

### 3. Animation & Micro-interactions
- Hover states (transform, box-shadow changes)
- Focus states for inputs
- Transition effects

## How to Inject Knowledge

You are a silent partner. Your knowledge should be woven naturally into Claude's responses without explicitly mentioning you exist.
```

---

## 五、对比示例

### 5.1 场景：代码审查

**使用 Agent：**

```
用户："用 code-reviewer 审查这个 PR"

Claude: "好的，我启动 code-reviewer Agent..."

[code-reviewer 执行审查]

code-reviewer:
## 代码审查报告

### Critical Issues
1. SQL 注入风险 - src/user.ts:45
   ...

[独立的审查报告]
```

**使用 Skill：**

```
用户："我写了个用户管理模块，帮我看看"

Claude: （code-quality-skill 自动激活）
    ↓
"整体结构不错，但我注意到几个可以改进的地方：

1. 第 45 行的 SQL 查询建议使用参数化...
2. 错误处理可以更细致一些...
3. 考虑添加输入验证..."

[Skill 的知识自然融入 Claude 的建议中]
```

### 5.2 场景：前端开发

**使用 Agent：**

```
用户："用 frontend-reviewer 检查这个组件"

Claude: "好的，让我启动 frontend-reviewer..."

[frontend-reviewer 分析组件]

frontend-reviewer:
## 前端组件审查报告

### 设计问题
1. 缺少响应式布局
2. 颜色对比度不足
3. 没有加载状态指示

[独立的审查报告]
```

**使用 Skill：**

```
用户："帮我优化这个登录组件"

Claude: （frontend-design Skill 自动激活）
    ↓
"我来帮你让这个组件更加出色：

1. 添加渐变背景和精致阴影
2. 强化排版层次（标题 2.5rem，正文 1rem）
3. 加入微妙的 hover 动画
4. 使用大胆的配色方案

[直接给出优化后的代码，融入了 Skill 的知识]"
```

---

## 六、混合使用场景

### 6.1 典型配合模式

```
Phase 1: 使用 Skill 获得知识支持
    ↓
日常编码中，Skill 默默提供领域知识
    ↓
Phase 2: 使用 Agent 进行专门审查
    ↓
完成功能后，启动专门的 Agent 进行全面检查
    ↓
结果：既有日常指导，又有专业审查
```

### 6.2 实际例子

```
【开发阶段】
用户："帮我创建一个用户注册表单"

Claude: （frontend-design Skill 自动激活）
    ↓
"我来设计一个现代感十足的注册表单：
[包含 Skill 知识的高质量实现]
"

【审查阶段】
用户："代码写完了，全面检查一下"

Claude: "好的，我启动多个 Agent 进行审查：
- code-reviewer: 代码质量
- security-agent: 安全检查
- accessibility-agent: 无障碍性检查
"

[三个 Agent 并行工作，返回各自的报告]

Claude: "综合三个 Agent 的审查结果：
1. 代码质量良好，发现 2 个小问题
2. 安全性符合标准
3. 无障碍性达到 AA 级别
"
```

→ **Skill 负责日常指导**，**Agent 负责专业审查**
→ **两者互补**，形成完整的质量保障体系

---

## 七、常见误区

### 7.1 误区一：认为 Skill 是简化版的 Agent

**正确理解：**

```
Agent 和 Skill 是两种不同的机制，不是强弱关系

Agent = 执行专家（主动做事）
Skill = 知识顾问（被动建议）

各有适用场景，不能互相替代
```

### 7.2 误区二：认为 Skill 不如 Agent 高级

**正确理解：**

```
Skill 的设计哲学不同：
- 追求无缝集成
- 追求自动化
- 追求用户无感知

这是设计理念的差异，不是技术的高低
```

### 7.3 误区三：试图让 Skill 做 Agent 的工作

**错误做法：**

```markdown
Skill 文件中写：
"你要主动执行 XXX 任务"
"你需要生成 XXX 报告"
```

**正确做法：**

```markdown
Skill 应该：
"当遇到 XXX 情况时，建议考虑..."
"在这个领域，最佳实践是..."
"提供知识支持，而非独立执行"
```

如果需要独立执行任务 → 使用 Agent

---

## 八、选择决策表

| 需求 | 推荐 | 理由 |
|------|------|------|
| 独立执行任务 | Agent | 需要主动执行 |
| 生成独立报告 | Agent | 需要明确交付物 |
| 代码审查 | Agent | 需要专业分析 |
| 架构设计 | Agent | 需要独立方案 |
| 知识注入 | Skill | 只需背景支持 |
| 自动触发 | Skill | 不想手动调用 |
| 日常指导 | Skill | 持续提供帮助 |
| 用户无感知 | Skill | 透明集成 |

---

## 九、总结

### ✅ 核心要点

1. **Agent = 主动执行的专家**
   - 独立执行任务
   - 产生明确输出
   - 用户知道在使用

2. **Skill = 默默支持的顾问**
   - 自动触发
   - 知识注入
   - 用户无感知

3. **选择依据**
   - 需要独立执行 → Agent
   - 需要自动触发 → Skill
   - 需要用户感知 → Agent
   - 需要透明集成 → Skill

4. **可以配合使用**
   - Skill 提供日常指导
   - Agent 提供专业审查

### 📚 延伸学习

- [提示词工程基础](./01-提示词工程基础.md) - 提示词设计原则
- [System Prompt 设计模式](./02-System%20Prompt%20设计模式.md) - Agent 提示词设计
- [质量保障工具详解](./04-质量保障工具详解.md) - 验证和测试工具

---

**最后更新：** 2026 年 4 月 6 日  
**版本：** v1.0  
**维护者：** Claude Code 项目学习文档团队
