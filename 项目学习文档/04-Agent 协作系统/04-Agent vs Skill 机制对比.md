# Agent vs Skill 机制对比

> **重要提示**：Agent 和 Skill 是 Claude Code 中两种不同的扩展机制。虽然它们都是 Markdown 文件，但在触发方式、使用场景和工作模式上有本质区别。

---

## 一、核心区别总览

### 1.1 一句话概括

```
Agent = 主动干活的专家（你叫他做事）
Skill = 默默提供知识的顾问（自动给你建议）
```

### 1.2 对比表格

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

## 二、工作机制详解

### 2.1 Agent 的工作流程

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

**关键特点：**
- ✅ **独立执行**：Agent 是一个独立的执行单元
- ✅ **明确触发**：用户或 Claude 明确启动
- ✅ **专门输出**：产生独立的报告或成果
- ✅ **可追踪**：清楚知道哪个 Agent 做了什么

**实际例子：**

```\n用户："我写完了登录功能，帮我检查一下有没有安全问题"

Claude: "好的，我来用 security-agent 检查一下安全性..."

[启动 security-agent]

[security-agent 执行分析]

security-agent: 
## 安全审查报告

### 严重问题
1. **SQL 注入风险** - `src/auth/login.ts:45`
   - 问题：字符串拼接 SQL
   - 修复：使用参数化查询

### 建议
1. 立即修复 SQL 注入问题
2. 添加输入验证
3. 实施速率限制

[主 Claude 整合结果并回复用户]

Claude: "根据 security-agent 的审查，发现了以下安全问题..."
```

→ **用户清楚地知道**：security-agent 被启动 → 执行了审查 → 返回了报告

---

### 2.2 Skill 的工作流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 主 Claude
    participant S as Skill
    
    U->>C: "帮我创建一个登录表单"
    
    Note over C: 检测到前端 UI 任务
    
    C->>S: 自动激活 frontend-design Skill
    
    Note over S: 在后台提供领域知识
    
    S-->>C:  silently inject knowledge:<br/>- 大胆的设计选择<br/>- 排版层次<br/>- 动画建议<br/>- 视觉细节
    
    C->>U: 给出包含 Skill 知识的回答<br/>但不会提到 Skill
    
    Note over U,C,S: Skill 是默默支持的顾问
```

**关键特点：**
- ✅ **自动触发**：基于上下文自动激活
- ✅ **后台支持**：不直接面对用户
- ✅ **知识注入**：为主 Claude 提供领域知识
- ✅ **透明集成**：用户感觉不到 Skill 的存在

**实际例子：**

```\n用户："帮我创建一个登录表单"

Claude: （检测到"登录表单"是前端 UI 任务）
    ↓
frontend-design Skill 自动激活
    ↓
在后台提供以下知识：
- 使用大胆的渐变背景，不要用纯白色
- 标题使用 2.5rem，正文使用 1rem，形成对比
- 添加 hover 状态的微动画（transform: translateY(-2px)）
- 按钮使用圆角 8px，阴影 0 4px 6px rgba(0,0,0,0.1)
- 避免使用默认的浏览器样式
    ↓
Claude 将这些知识融入到回答中：

"我来帮你设计一个现代感十足的登录表单：

```html
<form class="login-form">
  <h2>Welcome Back</h2>
  <!-- 使用渐变背景和精致的阴影 -->
  <div class="form-group">
    <label>Email</label>
    <input type="email" class="input-field" />
  </div>
  
  <style>
    .login-form {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      padding: 3rem;
      border-radius: 16px;
      box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    }
    
    h2 {
      font-size: 2.5rem;  /* 大标题形成视觉焦点 */
      color: white;
      margin-bottom: 2rem;
    }
    
    .input-field:hover {
      transform: translateY(-2px);  /* 微动画提升交互感 */
      box-shadow: 0 6px 12px rgba(0,0,0,0.15);
    }
  </style>
</form>
```

注意看这个设计：
- ✨ 渐变背景营造现代感
- ✨ 清晰的排版层次
- ✨ 精致的交互动画
- ✨ 避免平庸的默认样式
"
```

→ **用户完全不知道** Skill 的存在，以为这些都是 Claude 自己的想法
→ **实际上**：Claude 借助 Skill 的专业知识给出了更优质的回答

---

## 三、frontend-design Skill 深度剖析

让我们通过最典型的 Skill 案例，深入理解 Skill 的工作方式。

### 3.1 Skill 文件结构

**文件位置：** `plugins/frontend-design/skills/frontend-design/SKILL.md`

**完整结构：**

```markdown
---
name: frontend-design
description: |
  Auto-invoked for frontend UI work to provide guidance on:
  - Bold design choices (avoid generic AI aesthetics)
  - Typography hierarchy and contrast
  - Animations and micro-interactions
  - Visual details (spacing, colors, shadows)
trigger: |
  Automatically invoked when user requests or implies:
  - Frontend UI development (forms, pages, layouts, components)
  - Styling work (CSS, Tailwind, styled-components)
  - Interactive elements (buttons, modals, animations)
  - Visual design decisions
---

## When to Auto-Invoke

Automatically invoke this skill whenever the user requests or implies:

### Primary Triggers
- Creating new UI components (forms, buttons, cards, modals)
- Designing page layouts
- Styling existing components
- Improving visual appearance
- Adding animations or transitions

### Secondary Triggers
- User mentions "make it pretty", "improve the design"
- Working with CSS frameworks (Tailwind, Bootstrap, etc.)
- Discussing color schemes, typography, spacing

## What You Provide

When invoked, provide guidance on these aspects:

### 1. Bold Design Choices
❌ Avoid: Generic, boring AI-looking designs
✅ Encourage: Bold gradients, unique color combinations, distinctive layouts

Example:
Instead of plain white card:
```css
/* Boring */
.card {
  background: white;
  border: 1px solid #ddd;
}

/* Bold */
.card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}
```

### 2. Typography Hierarchy
- Establish clear visual hierarchy (H1: 2.5rem, H2: 2rem, Body: 1rem)
- Use appropriate font weights (bold headings, regular body)
- Ensure sufficient contrast (WCAG AA minimum)
- Consider line-height for readability

### 3. Animation & Micro-interactions
- Hover states (transform, box-shadow changes)
- Focus states for inputs
- Transition effects (smooth easing)
- Loading states and feedback
- Subtle animations that enhance UX

### 4. Visual Details
- Spacing: Consistent padding/margins (use 4px grid)
- Colors: Complementary palette, avoid pure black/white
- Shadows: Layered shadows for depth
- Border radius: Consistent rounding (4px, 8px, 16px)
- Borders: Purposeful, not decorative

## How to Inject Knowledge

You are a silent partner. Your knowledge should be woven naturally 
into Claude's responses without explicitly mentioning you exist.

### Do:
- Suggest specific design improvements
- Provide code examples with explanations
- Explain why certain choices look better
- Reference design principles

### Don't:
- Say "As a Skill, I suggest..."
- Break the fourth wall
- Make the user aware of your existence

## Example Interactions

### Example 1: Form Design
User: "Create a signup form"

Your knowledge injection:
- Suggest modern form patterns (floating labels, inline validation)
- Recommend color scheme (trustworthy blues, clean whites)
- Add micro-interactions (focus glow, success animations)
- Emphasize accessibility (proper labels, error messages)

### Example 2: Button Styling
User: "Style this button"

Your knowledge injection:
- Recommend bold hover effects
- Suggest gradient backgrounds
- Add subtle scale animation on click
- Ensure proper contrast ratios

### Example 3: Layout Improvement
User: "Make this page look better"

Your knowledge injection:
- Establish visual hierarchy
- Add whitespace strategically
- Introduce accent colors
- Create focal points
```

### 3.2 触发机制详解

**触发条件检测：**

```markdown
当 Claude 处理用户请求时，会进行以下检测：

1. **关键词匹配**
   - "form", "button", "card", "modal" → UI 组件
   - "style", "design", "pretty" → 样式需求
   - "animate", "transition", "hover" → 交互效果

2. **语义理解**
   - "创建一个登录页面" → 前端 UI
   - "美化这个按钮" → 样式改进
   - "让界面更好看" → 设计建议

3. **上下文分析**
   - 之前的对话是否在讨论前端
   - 代码片段是否包含 JSX/HTML/CSS
   - 项目类型是否是前端项目
```

**触发决策树：**

```
用户请求
    ↓
包含前端关键词？
    ├─ 是 → 激活 Skill
    └─ 否 → 继续检查
        ↓
语义上是否需要设计建议？
    ├─ 是 → 激活 Skill
    └─ 否 → 不激活
        ↓
上下文中是否有前端代码？
    ├─ 是 → 激活 Skill
    └─ 否 → 不激活
```

### 3.3 知识注入机制

**Silent Injection（静默注入）：**

```markdown
Skill 不会说："我是 frontend-design Skill，我建议..."

而是将知识自然融入 Claude 的思考过程：

Claude 的内心独白（用户看不到）:
"用户需要一个登录表单...
 frontend-design Skill 激活了...
 它建议：
 - 使用大胆的渐变背景
 - 建立清晰的排版层次
 - 添加微妙的动画效果
 - 注意阴影和圆角的细节
 
好，我会把这些建议融入到设计中..."

Claude 的实际输出（用户看到的）:
"我来为你设计一个引人注目的登录表单：

[包含渐变、动画、精致细节的代码]

这个设计的亮点：
✨ 渐变背景营造现代感
✨ 清晰的视觉层次
✨ 精致的交互反馈"
```

→ **关键**：Skill 的知识变成了 Claude 自己的思考和建议

---

## 四、Agent 与 Skill 的选择指南

### 4.1 何时使用 Agent？

**使用 Agent 的场景：**

✅ **需要主动执行任务**
```
"用 code-explorer 分析一下认证模块的实现"
"让 code-architect 设计一个架构方案"
"启动 code-reviewer 审查这段代码"
```

✅ **需要专门的分析和输出**
```
需要一份独立的分析报告
需要一个完整的架构蓝图
需要详细的审查意见清单
```

✅ **需要明确的交付物**
```
代码库探索报告
架构设计方案
安全审查清单
性能优化建议
```

✅ **需要多 Agent 协作**
```
七阶段流程中的多个 Agent 并行工作
PR 审查时 6 个 Agent 同时分析
```

**判断标准：**
- 是否需要一个"专家"来专门处理？
- 是否需要独立的输出成果？
- 是否需要明确的执行过程？

→ **如果答案是 YES**，使用 Agent

---

### 4.2 何时使用 Skill？

**使用 Skill 的场景：**

✅ **需要领域知识注入**
```
写前端代码时自动获得设计建议
编写 API 时自动遵循 REST 规范
处理数据时自动考虑安全性
```

✅ **希望在日常工作中自动获得指导**
```
不需要每次都显式调用
希望在编码过程中自然获得建议
不想中断工作流
```

✅ **不需要额外的交互**
```
只要背景知识支持就好
不需要单独的报告或分析
希望 Skill 默默提供支持
```

✅ **增强主 Claude的能力**
```
让 Claude 在特定领域更专业
提供该领域的最佳实践
注入行业标准和规范
```

**判断标准：**
- 是否只需要知识支持而不需要独立执行？
- 是否希望自动触发而非手动调用？
- 是否想让建议自然融入回答中？

→ **如果答案是 YES**，使用 Skill

---

### 4.3 对比示例

**场景 1：代码审查**

```
【使用 Agent】
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

```
【使用 Skill】（假设有个 code-quality-skill）
用户："我写了个用户管理模块，帮我看看"

Claude: （code-quality-skill 自动激活）
    ↓
"整体结构不错，但我注意到几个可以改进的地方：

1. 第 45 行的 SQL 查询建议使用参数化...
2. 错误处理可以更细致一些...
3. 考虑添加输入验证..."

[Skill 的知识自然融入 Claude 的建议中]
```

---

**场景 2：前端开发**

```
【使用 Agent】（假设有 frontend-reviewer Agent）
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

```
【使用 Skill】（frontend-design Skill）
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

## 五、混合使用场景

在实际项目中，Agent 和 Skill 可以配合使用：

### 5.1 典型配合模式

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

**实际例子：**

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

## 六、常见误区

### ❌ 误区 1：认为 Skill 是简化版的 Agent

**正确理解：**
```
Agent 和 Skill 是两种不同的机制，不是强弱关系

Agent = 执行专家（主动做事）
Skill = 知识顾问（被动建议）

各有适用场景，不能互相替代
```

### ❌ 误区 2：认为 Skill 不如 Agent 高级

**正确理解：**
```
Skill 的设计哲学不同：
- 追求无缝集成
- 追求自动化
- 追求用户无感知

这是设计理念的差异，不是技术的高低
```

### ❌ 误区 3：试图让 Skill 做 Agent 的工作

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

## 七、设计决策背后的思考

### 7.1 为什么需要两种机制？

**问题：** 为什么不统一用一种机制？

**答案：** 因为有两种截然不同的需求场景

**场景 A：需要专门执行**
```
"审查这个 PR"
"分析代码库"
"设计架构方案"

→ 需要一个"专家"来专门处理
→ 适合 Agent
```

**场景 B：需要知识增强**
```
日常编码时希望：
- 自动遵循最佳实践
- 自然获得领域知识
- 不中断工作流

→ 需要默默的顾问
→ 适合 Skill
```

**类比理解：**

```
公司里的两种角色：

Agent = 外部咨询公司
    - 有专门问题时聘请
    - 独立完成项目
    - 提交正式报告
    
Skill = 内部资深顾问
    - 一直在你身边
    - 随时提供建议
    - 不独立执行任务
```

---

### 7.2 触发方式的设计哲学

**Agent 的显式触发：**

```markdown
哲学：让用户知道发生了什么

优点：
✓ 透明度高
✓ 可控性强
✓ 容易调试
✓ 明确的交付物

缺点：
✗ 需要额外操作
✗ 可能忘记使用
✗ 工作流中断
```

**Skill 的自动触发：**

```markdown
哲学：让用户感受不到存在

优点：
✓ 无缝集成
✓ 自动化
✓ 不中断工作流
✓ 自然的体验

缺点：
✗ 透明度低
✗ 不易调试
✗ 难以定制
```

---

## 八、实战建议

### 8.1 作为使用者

**日常开发推荐做法：**

```markdown
1. **充分利用 Skill**
   - 让 Skill 默默提供支持
   - 享受自动化的知识注入
   - 不必关心 Skill 的存在

2. **关键节点使用 Agent**
   - 完成功能后用 code-reviewer 审查
   - 开发新功能前用 code-architect 设计
   - 遇到问题时用 code-explorer 分析

3. **理解两者的价值**
   - Skill 让你日常 coding 更专业
   - Agent 让关键时刻更可靠
```

### 8.2 作为开发者

**创建扩展时的选择：**

```markdown
问自己三个问题：

Q1: 是否需要独立执行任务？
   Yes → Agent
   No → 继续 Q2

Q2: 是否需要自动触发？
   Yes → Skill
   No → Agent

Q3: 是否需要用户感知到存在？
   Yes → Agent
   No → Skill

示例决策：

"我想创建一个帮助前端设计的扩展"
Q1: 独立执行？No（只是给建议）
Q2: 自动触发？Yes（写前端时自动出现）
Q3: 用户感知？No（默默支持就好）
→ 选择：Skill

"我想创建一个代码审查工具"
Q1: 独立执行？Yes（要生成报告）
→ 选择：Agent
```

---

## 九、总结对比表

| 维度 | Agent | Skill |
|------|-------|-------|
| **本质** | 执行专家 | 知识顾问 |
| **触发** | 显式/隐式调用 | 自动检测触发 |
| **存在感** | 强烈（独立输出） | 透明（融入回答） |
| **交付物** | 独立报告/方案 | 无（知识注入） |
| **工作模式** | 主动执行 | 被动支持 |
| **适用场景** | 专门任务、审查、分析 | 日常指导、最佳实践 |
| **文件位置** | `agents/*.md` | `skills/*/SKILL.md` |
| **配置复杂度** | 高（完整配置） | 中（触发条件 + 知识） |
| **用户感知** | "XXX Agent 在做 XXX" | 无感知（以为是 Claude 自己的想法） |
| **类比** | 外部咨询顾问 | 内部资深专家 |

---

## 十、快速决策树

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

**最后更新：** 2026 年 4 月 2 日  
**版本：** v1.0  
**维护者：** Claude Code 项目学习文档团队
