# Agent 协作流程图集

本章节包含多个流程图，帮助你直观理解 Agent 系统的工作机制。

---

## 图 1: Agent 系统总览

```mermaid
graph TB
    A[用户需求] --> B{需求类型判断}
    
    B -->|功能开发 | C[feature-dev 插件]
    B -->|PR 审查 | D[pr-review-toolkit 插件]
    B -->|代码审查 | E[code-review 插件]
    B -->|安全问题 | F[security-guidance 插件]
    B -->|插件开发 | G[plugin-dev 插件]
    
    C --> C1[code-explorer Agent]
    C --> C2[code-architect Agent]
    C --> C3[code-reviewer Agent]
    
    D --> D1[silent-failure-hunter]
    D --> D2[comment-analyzer]
    D --> D3[type-design-analyzer]
    D --> D4[code-simplifier]
    D --> D5[pr-test-analyzer]
    D --> D6[code-reviewer]
    
    E --> E1[5 个并行审查 Agent]
    
    G --> G1[agent-creator]
    G --> G2[plugin-validator]
    G --> G3[skill-reviewer]
    
    style C fill:#e1f5ff
    style D fill:#fff4e1
    style E fill:#f0e1ff
    style G fill:#e1ffe1
```

**说明：**
- 不同的需求触发不同的插件
- 每个插件包含一组专门的 Agent
- Agent 之间可以并行工作

---

## 图 2: 单个 Agent 的执行流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 主 Claude
    participant A as Agent
    participant T as Tools
    
    U->>C: "用 code-reviewer 审查代码"
    
    Note over C: 解析请求<br/>识别需要启动的 Agent
    
    C->>A: 启动 code-reviewer Agent
    
    Note over A: 加载 Frontmatter 配置<br/>读取 System Prompt
    
    A->>A: 分析任务需求
    
    A->>T: 调用 Glob/Grep/Read 等工具
    
    T-->>A: 返回代码内容
    
    A->>A: 按照 System Prompt 分析代码
    
    Note over A: 置信度评分 ≥80 才报告
    
    A->>C: 返回审查报告
    
    C->>U: 展示审查结果
```

**关键步骤：**
1. **触发识别**：Claude 判断是否需要启动 Agent
2. **配置加载**：读取 Agent 的 Frontmatter 和 System Prompt
3. **工具调用**：使用授予的工具权限收集信息
4. **分析处理**：按照 System Prompt 的指导进行分析
5. **结果输出**：返回格式化的报告

---

## 图 3: feature-dev 七阶段流程中的 Agent 协作

```mermaid
graph LR
    P1[Phase 1: Discovery<br/>需求发现] --> P2
    
    P2[Phase 2: Exploration<br/>代码库探索] --> P2A[启动 2-3 个<br/>code-explorer Agents]
    P2A --> P2B[并行分析不同方面]
    P2B --> P2C[整合分析报告]
    P2C --> P3
    
    P3[Phase 3: Clarification<br/>需求澄清] --> P4
    
    P4[Phase 4: Architecture<br/>架构设计] --> P4A[启动 2-3 个<br/>code-architect Agents]
    P4A --> P4B[并行设计方案]
    P4B --> P4C[整合并推荐方案]
    P4C --> P5
    
    P5[Phase 5: Implementation<br/>实现] --> P6
    
    P6[Phase 6: Review<br/>质量审查] --> P6A[启动 3 个<br/>code-reviewer Agents]
    P6A --> P6B[并行审查不同维度]
    P6B --> P6C[整合审查结果]
    P6C --> P7
    
    P7[Phase 7: Summary<br/>总结交付]
    
    style P2A fill:#fff4e1
    style P4A fill:#e1ffe1
    style P6A fill:#ffe1e1
```

**关键点：**
- Phase 2、4、6 都使用了多个 Agent 并行工作
- 主 Claude 负责整合各 Agent 的输出
- 并行处理大幅缩短时间（从 30 分钟降至 10-15 分钟）

---

## 图 4: code-explorer Agent 详细工作流程

```mermaid
flowchart TD
    Start[开始] --> Step1[第一步：Feature Discovery]
    
    Step1 --> S1a[查找入口点<br/>APIs, UI, CLI]
    Step1 --> S1b[定位核心实现文件]
    Step1 --> S1c[映射功能边界]
    
    S1a & S1b & S1c --> Step2[第二步：Code Flow Tracing]
    
    Step2 --> S2a[追踪调用链]
    Step2 --> S2b[追踪数据转换]
    Step2 --> S2c[识别依赖关系]
    Step2 --> S2d[记录状态变化]
    
    S2a & S2b & S2c & S2d --> Step3[第三步：Architecture Analysis]
    
    Step3 --> S3a[映射抽象层次]
    Step3 --> S3b[识别设计模式]
    Step3 --> S3c[记录组件接口]
    Step3 --> S3d[关注横切关注点]
    
    S3a & S3b & S3c & S3d --> Step4[第四步：Implementation Details]
    
    Step4 --> S4a[关键算法和数据结构]
    Step4 --> S4b[错误处理和边界情况]
    Step4 --> S4c[性能考虑]
    Step4 --> S4d[技术债务识别]
    
    Step4 --> Output[生成综合分析报告]
    
    Output --> End[结束]
    
    style Step1 fill:#e1f5ff
    style Step2 fill:#e1f5ff
    style Step3 fill:#e1f5ff
    style Step4 fill:#e1f5ff
    style Output fill:#fff4e1
```

**输出要求：**
- 入口点必须包含文件路径和行号（如 `src/auth/login.ts:15`）
- 执行流程必须包含数据转换说明
- 架构分析必须指出设计模式和层次
- 必须列出 essential files 清单

---

## 图 5: code-architect Agent 决策流程

```mermaid
flowchart TD
    Start[开始] --> Analysis[代码库模式分析]
    
    Analysis --> A1[提取现有模式]
    Analysis --> A2[识别技术栈]
    Analysis --> A3[查找类似功能]
    Analysis --> A4[读取 CLAUDE.md]
    
    A1 & A2 & A3 & A4 --> Design[架构设计]
    
    Design --> D1[基于现有模式设计]
    Design --> D2[做出果断决策]
    Design --> D3[确保无缝集成]
    Design --> D4[考虑可测试性]
    
    D1 & D2 & D3 & D4 --> Blueprint[实施蓝图]
    
    Blueprint --> B1[指定每个文件]
    Blueprint --> B2[定义组件职责]
    Blueprint --> B3[明确集成点]
    Blueprint --> B4[绘制数据流]
    Blueprint --> B5[分阶段检查清单]
    
    B1 & B2 & B3 & B4 & B5 --> Critical[关键细节]
    
    Critical --> C1[错误处理]
    Critical --> C2[状态管理]
    Critical --> C3[测试策略]
    Critical --> C4[性能优化]
    Critical --> C5[安全考虑]
    
    C1 & C2 & C3 & C4 & C5 --> Output[输出完整蓝图]
    
    style Analysis fill:#e1f5ff
    style Design fill:#e1ffe1
    style Blueprint fill:#fff4e1
    style Critical fill:#ffe1e1
    style Output fill:#f0e1ff
```

**设计原则：**
- **基于现有模式**：不是凭空设计，而是复用已有模式
- **果断决策**：选择一个方案并坚持，不模棱两可
- **完整蓝图**：具体到每个文件、每个函数、每个集成点

---

## 图 6: code-reviewer Agent 置信度评分机制

```mermaid
flowchart TD
    Start[发现潜在问题] --> Analyze[问题分析]
    
    Analyze --> Check[对照评分标准]
    
    Check --> Score{评分}
    
    Score -->|0 分 | Reject0[完全误报<br/>直接丢弃]
    Score -->|25 分 | Reject25[可能是误报<br/>或风格问题<br/>丢弃]
    Score -->|50 分 | Reject50[真实但次要<br/>可选修复<br/>丢弃]
    Score -->|75 分 | Report75[高度确信<br/>重要问题<br/>报告]
    Score -->|100 分 | Report100[绝对确定<br/>频繁发生<br/>必须修复<br/>报告]
    
    Report75 --> Format[格式化输出]
    Report100 --> Format
    
    Format --> Include[包含:<br/>文件路径 + 行号<br/>置信度分数<br/>问题描述<br/>修复建议<br/>代码示例]
    
    Include --> Output[输出报告]
    
    style Reject0 fill:#ffe1e1
    style Reject25 fill:#ffe1e1
    style Reject50 fill:#fff4e1
    style Report75 fill:#e1ffe1
    style Report100 fill:#e1ffe1
    style Output fill:#e1f5ff
```

**评分标准详解：**

| 分数 | 含义 | 是否报告 | 示例 |
|------|------|---------|------|
| **0** | 完全误报 | ❌ | 预存在的问题或误解 |
| **25** | 可能误报 | ❌ | 风格问题，未违反规范 |
| **50** | 真实但次要 | ❌ | 小问题，不常发生 |
| **75** | 高度确信 | ✅ | 影响功能，违反规范 |
| **100** | 绝对确定 | ✅ | 频繁发生，证据确凿 |

**阈值设置：** 只报告 confidence ≥ 80 的问题（即 75 分和 100 分）

---

## 图 7: silent-failure-hunter Agent 审查流程

```mermaid
flowchart TD
    Start[开始审查] --> Locate[定位错误处理代码]
    
    Locate --> L1[try-catch 块]
    Locate --> L2[error callbacks]
    Locate --> L3[条件分支]
    Locate --> L4[回退逻辑]
    Locate --> L5[日志后继续]
    Locate --> L6[可选链/空值合并]
    
    L1 & L2 & L3 & L4 & L5 & L6 --> Scrutinize[严格审查每处]
    
    Scrutinize --> Q1[日志质量？<br/>有足够上下文吗？]
    Scrutinize --> Q2[用户反馈？<br/>清晰可操作吗？]
    Scrutinize --> Q3[catch 特异性？<br/>会隐藏其他错误吗？]
    Scrutinize --> Q4[回退合理？<br/>掩盖问题吗？]
    Scrutinize --> Q5[错误传播？<br/>应该向上传递吗？]
    
    Q1 & Q2 & Q3 & Q4 & Q5 --> Classify[问题分类]
    
    Classify --> Critical[CRITICAL<br/>静默失败/宽泛 catch]
    Classify --> High[HIGH<br/>差劲的错误消息<br/>无理的 fallback]
    Classify --> Medium[MEDIUM<br/>缺少上下文<br/>可以更具体]
    
    Critical --> Output[输出详细报告]
    High --> Output
    Medium --> Output
    
    Output --> Include[包含:<br/>位置 + 严重性<br/>问题描述<br/>隐藏的错误类型<br/>用户影响<br/>修复建议<br/>代码示例]
    
    style Locate fill:#e1f5ff
    style Scrutinize fill:#fff4e1
    style Critical fill:#ffe1e1
    style High fill:#ffe1a1
    style Medium fill:#ffffe1
    style Output fill:#e1ffe1
```

**审查深度：**
- 不仅检查有没有 catch 块
- 还要检查 catch 块的质量
- 更要检查错误是否真正被处理而非隐藏

---

## 图 8: Agent 与 Hook 的配合机制

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 主 Claude
    participant A as Agent
    participant H as Hook
    participant S as Stop Hook
    
    U->>C: "开发积分系统"
    
    C->>A: 启动 code-explorer
    
    Note over H: PreToolUse Hook<br/>检查工具使用合法性
    
    A->>H: 请求使用 Read 工具
    
    H-->>A: ✓ 允许
    
    A->>C: 返回分析报告
    
    C->>A: 启动 code-architect
    
    A->>C: 返回架构方案
    
    C->>A: 开始实现代码
    
    Note over H: security-guidance Hook<br/>实时监控安全风险
    
    A->>H: 准备写入代码
    
    H-->>A: ⚠️ 发现精度问题<br/>float vs decimal
    
    A->>A: 修正问题
    
    A->>C: 实现完成
    
    C->>A: 启动 code-reviewer
    
    A->>C: 返回审查报告<br/>发现 3 个问题
    
    C->>S: 准备完成任务
    
    Note over S: Stop Hook<br/>七阶段完整性检查
    
    S->>S: 检查清单:<br/>✓ Phase 1-2 完成<br/>✓ Phase 3 完成<br/>✓ Phase 4 完成<br/>✓ Phase 5 完成<br/>✓ Phase 6 完成 (3 个问题已修复)<br/>✓ Phase 7 完成
    
    S-->>C: ✓ 批准完成
    
    C->>U: 任务完成!
    
    style H fill:#fff4e1
    style S fill:#ffe1e1
```

**配合要点：**
- **Hook 是防御机制**：检查、验证、阻止
- **Agent 是进攻机制**：分析、设计、实现
- **各司其职**：Hook 不做创造性工作，Agent 不做规范性检查
- **协同工作**：在关键节点配合，确保质量和安全

---

## 图 9: pr-review-toolkit 六 Agent 并行审查

```mermaid
graph TB
    PR[Pull Request] --> Parallel[并行启动 6 个 Agent]
    
    Parallel --> A1[comment-analyzer<br/>注释准确性]
    Parallel --> A2[pr-test-analyzer<br/>测试覆盖率]
    Parallel --> A3[silent-failure-hunter<br/>错误处理]
    Parallel --> A4[type-design-analyzer<br/>类型设计]
    Parallel --> A5[code-reviewer<br/>综合质量]
    Parallel --> A6[code-simplifier<br/>代码简化]
    
    A1 --> R1[注释报告]
    A2 --> R2[测试报告]
    A3 --> R3[错误处理报告]
    A4 --> R4[类型设计报告]
    A5 --> R5[综合审查报告]
    A6 --> R6[简化建议]
    
    R1 & R2 & R3 & R4 & R5 & R6 --> Consolidate[主 Claude 整合]
    
    Consolidate --> Final[最终审查报告]
    
    Final --> Summary[摘要]
    Final --> Critical[Critical 问题]
    Final --> Important[Important 问题]
    Final --> Suggestions[改进建议]
    Final --> Positive[做得好的地方]
    
    style Parallel fill:#e1f5ff
    style A1 fill:#fff4e1
    style A2 fill:#e1ffe1
    style A3 fill:#ffe1e1
    style A4 fill:#f0e1ff
    style A5 fill:#ffffe1
    style A6 fill:#e1f5ff
    style Consolidate fill:#ffe1f5
```

**并行优势：**
- **专业分工**：每个 Agent 只关注一个维度
- **同时执行**：6 个 Agent 同时工作，而非串行
- **全面覆盖**：注释、测试、错误处理、类型、质量、简洁性，无一遗漏
- **减少时间**：从 60 分钟降至 10-15 分钟

---

## 图 10: Agent 创建流程（agent-creator）

```mermaid
flowchart TD
    UserReq[用户需求] --> Describe["描述:\n\"创建一个帮我 XXX 的 Agent\""]
    
    Describe --> Extract[提取核心意图]
    
    Extract --> E1[根本目的]
    Extract --> E2[关键职责]
    Extract --> E3[成功标准]
    
    E1 & E2 & E3 --> Persona[设计专家角色]
    
    Persona --> P1[体现领域知识]
    P1 --> P2[引导决策方式]
    
    Persona --> Instructions[编写 System Prompt]
    
    Instructions --> I1[行为边界]
    Instructions --> I2[方法论和最佳实践]
    Instructions --> I3[边缘情况处理]
    Instructions --> I4[输出格式要求]
    
    I1 & I2 & I3 & I4 --> Optimize[性能优化]
    
    Optimize --> O1[决策框架]
    Optimize --> O2[质量控制]
    Optimize --> O3[工作流模式]
    Optimize --> O4[降级策略]
    
    O1 & O2 & O3 & O4 --> Identifier[创建标识符]
    
    Identifier --> ID[小写 + 连字符<br/>2-4 个词<br/>3-50 字符<br/>避免通用词]
    
    Identifier --> Examples[编写触发示例]
    
    Examples --> Ex1[2-4 个<example>块]
    Ex1 --> Sh1[显式触发]
    Ex1 --> Sh2[隐式触发]
    Ex1 --> Sh3[主动触发]
    
    ID & Examples --> Generate[生成 Agent 文件]
    
    Generate --> Write[写入 agents/[name].md]
    
    Write --> Validate[建议验证]
    
    Validate --> Test[测试触发]
    
    style UserReq fill:#e1f5ff
    style Extract fill:#fff4e1
    style Persona fill:#e1ffe1
    style Instructions fill:#ffe1e1
    style Optimize fill:#f0e1ff
    style Generate fill:#ffffe1
    style Write fill:#ffe1f5
```

**自动化程度：**
- 用户只需一句话描述需求
- agent-creator 自动生成完整配置
- 包括 Frontmatter、System Prompt、触发示例
- 生成的文件立即可用

---

## 总结

这些流程图展示了 Agent 系统的各个层面：

1. **图 1**：整体架构 - 不同插件包含不同 Agent
2. **图 2**：单个 Agent 执行 - 从触发到输出的完整流程
3. **图 3**：七阶段协作 - Agent 如何在流程中配合
4. **图 4**：code-explorer - 四步分析法
5. **图 5**：code-architect - 基于现有模式的决策流程
6. **图 6**：code-reviewer - 置信度评分过滤机制
7. **图 7**：silent-failure-hunter - 深度错误处理审查
8. **图 8**：Agent-Hook 配合 - 攻防结合
9. **图 9**：PR 审查矩阵 - 6 个 Agent 并行工作
10. **图 10**：agent-creator - 自动化创建 Agent

建议结合文字内容一起学习，效果更佳！
