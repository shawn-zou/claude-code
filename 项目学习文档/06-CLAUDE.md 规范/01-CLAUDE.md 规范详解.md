# CLAUDE.md 规范详解

CLAUDE.md 是 Claude Code 项目中用于定义项目规范、约定和最佳实践的配置文件。它帮助 Claude 理解项目的特定要求，从而提供更符合项目风格的代码和建议。

---

## 一、什么是 CLAUDE.md

### 1.1 基本概念

**CLAUDE.md** 是一个 Markdown 格式的项目规范文件，类似于：
- `.eslintrc` - ESLint 配置
- `.prettierrc` - Prettier 配置
- `CONTRIBUTING.md` - 贡献指南

但 CLAUDE.md 更加**全面和智能**，它不仅包含技术规则，还包括：
- 📋 项目规范和约定
- 🏗️ 架构设计原则
- 💻 编码风格指南
- 🧪 测试要求
- 📝 文档标准
- 🔒 安全规范

---

### 1.2 为什么需要 CLAUDE.md

**没有 CLAUDE.md 的问题：**
```
用户：帮我写一个用户服务

Claude: 好的，按照通用最佳实践...
    ↓
问题：
❌ 使用了下划线命名（项目要求驼峰）
❌ 缺少 JSDoc 注释（项目强制要求）
❌ 直接操作数据库（应该通过 Repository）
❌ 没有错误处理（项目要求统一异常处理）
```

**有 CLAUDE.md 的优势：**
```
用户：帮我写一个用户服务

Claude: 好的，根据项目的 CLAUDE.md 规范...
    ↓
✅ 使用驼峰命名（符合项目规范）
✅ 包含完整 JSDoc 注释（符合要求）
✅ 通过 UserRepository 访问数据（遵循分层）
✅ 统一的异常处理模式（符合项目标准）

结果：代码风格一致，易于维护
```

---

### 1.3 CLAUDE.md vs SessionStart Hook

**相似点：**
- 都在会话开始时加载上下文
- 都影响 Claude 的行为
- 都可以定义项目特定规则

**区别：**

| 特性 | CLAUDE.md | SessionStart Hook |
|------|-----------|-------------------|
| **格式** | Markdown 文本 | JSON 配置 + 脚本 |
| **用途** | 项目规范说明 | 自动化任务执行 |
| **灵活性** | 静态文档 | 动态执行代码 |
| **学习成本** | 低（易读易写） | 中（需要了解 Hook API） |
| **适用场景** | 规范、约定、最佳实践 | 环境设置、上下文加载 |

**最佳实践：**
- ✅ 用 CLAUDE.md 定义规范
- ✅ 用 SessionStart Hook 执行自动化
- ✅ 两者配合使用效果更佳

---

## 二、CLAUDE.md 的作用和价值

### 2.1 对项目的价值

#### 1. 统一代码风格
```markdown
## 命名规范
- 变量：camelCase (如：userName)
- 类名：PascalCase (如：UserService)
- 常量：UPPER_SNAKE_CASE (如：MAX_RETRY_COUNT)
- 私有属性：_prefix (如：_cache)
```

**效果：**
- ✅ 多人协作代码风格一致
- ✅ 减少 code review 中的风格争论
- ✅ 新人快速融入项目

---

#### 2. 保证代码质量
```markdown
## 代码质量要求
1. 函数不超过 50 行
2. 嵌套不超过 4 层
3. 所有公共方法必须有 JSDoc
4. 单元测试覆盖率 > 90%
```

**效果：**
- ✅ 代码可读性强
- ✅ 易于维护和扩展
- ✅ Bug 率降低

---

#### 3. 传承最佳实践
```markdown
## 错误处理最佳实践
1. 使用自定义错误类型
   ```javascript
   class AppError extends Error {
     constructor(code, message, statusCode = 500) {
       super(message);
       this.code = code;
       this.statusCode = statusCode;
     }
   }
   ```

2. 统一错误处理中间件
   ```javascript
   app.use((err, req, res, next) => {
     logger.error(err);
     res.status(err.statusCode).json({
       error: err.message,
       code: err.code
     });
   });
   ```
```

**效果：**
- ✅ 避免重复踩坑
- ✅ 经验沉淀为规范
- ✅ 团队能力整体提升

---

#### 4. 提高开发效率
```markdown
## 常用命令
# 开发
npm run dev          # 启动开发服务器
npm run build        # 构建生产版本

# 测试
npm test            # 运行测试
npm run test:cover  # 查看覆盖率

# 代码质量
npm run lint        # ESLint 检查
npm run format      # Prettier 格式化
```

**效果：**
- ✅ 新人快速上手
- ✅ 减少查阅时间
- ✅ 标准化操作流程

---

### 2.2 对 Claude 的指导作用

#### 1. 理解项目背景
```markdown
## 项目简介
本项目是一个电商平台的后端服务，采用微服务架构。
核心功能包括：用户管理、商品管理、订单处理、支付集成。

技术栈：
- 运行时：Node.js 18+
- 框架：Express.js
- 数据库：MySQL 8.0 (主), Redis 6.2 (缓存)
- 消息队列：RabbitMQ
- 部署：Docker + Kubernetes
```

**Claude 的理解：**
- 知道是电商领域
- 了解技术选型
- 理解架构模式

---

#### 2. 遵循编码规范
```markdown
## TypeScript 规范
1. 优先使用 interface 而非 type
2. 使用严格模式（strict: true）
3. 避免使用 any，使用 unknown 代替
4. 泛型命名：T, U, V 或具体含义（如 TData）

示例：
```typescript
// ✅ 好的写法
interface User {
  id: number;
  name: string;
}

// ❌ 不好的写法
type User = any;
```
```

**Claude 的行为：**
- ✅ 自动生成符合规范的代码
- ✅ 避免使用禁止的模式
- ✅ 推荐最佳实践

---

#### 3. 遵循架构约束
```markdown
## 分层架构
本项目采用经典三层架构：

Controller 层
- 职责：接收请求、参数验证、调用 Service
- 位置：src/controllers/
- 不允许：直接访问数据库

Service 层
- 职责：业务逻辑、事务管理
- 位置：src/services/
- 依赖：Repository、外部服务

Repository 层
- 职责：数据访问、ORM 封装
- 位置：src/repositories/
- 依赖：Database
```

**Claude 的约束：**
- ✅ Controller 不调用 Repository
- ✅ Service 不包含 SQL
- ✅ 层次清晰，职责明确

---

## 三、CLAUDE.md 的标准格式

### 3.1 基本结构

```markdown
# 项目名称 - CLAUDE.md

## 1. 项目概述
简要介绍项目的业务背景和技术定位。

## 2. 技术栈
列出使用的核心技术。

## 3. 项目结构
目录组织说明。

## 4. 编码规范
详细的编码要求。

## 5. 架构原则
架构设计和分层规范。

## 6. 测试要求
测试策略和覆盖率要求。

## 7. 安全规范
安全相关的注意事项。

## 8. 常用命令
开发和部署命令。
```

---

### 3.2 各部分详解

#### 章节 1：项目概述

```markdown
## 项目概述

### 业务背景
这是一个 [什么类型的产品]，服务于 [目标用户群体]，
解决 [什么痛点]，创造 [什么价值]。

### 核心功能
1. [功能模块 1] - 描述
2. [功能模块 2] - 描述
3. [功能模块 3] - 描述

### 技术愿景
- 高可用：99.9% SLA
- 高性能：P95 < 200ms
- 高可维护性：测试覆盖率 > 90%
```

**示例：**
```markdown
## 项目概述

### 业务背景
这是一个电商平台后端系统，服务于 C 端消费者和 B 端商家，
提供在线购物、订单管理、支付结算等服务。

### 核心功能
1. 用户中心 - 注册登录、个人信息、收货地址
2. 商品系统 - 商品管理、库存管理、分类检索
3. 订单系统 - 下单、订单查询、售后服务
4. 支付系统 - 微信支付、支付宝、银联

### 技术愿景
- 支持双 11 级别流量（10 万 QPS）
- 核心接口 P95 < 100ms
- 零重大安全事故
```

---

#### 章节 2：技术栈

```markdown
## 技术栈

### 核心 runtime
- Node.js 18.x LTS
- TypeScript 5.x

### Web 框架
- Express.js 4.x
- Koa.js 2.x (部分微服务)

### 数据存储
- MySQL 8.0 (主数据库)
- Redis 6.2 (缓存、会话)
- MongoDB 6.0 (日志、审计)

### 消息队列
- RabbitMQ 3.11 (异步任务)
- Kafka 3.3 (事件流)

### 基础设施
- Docker 20.x
- Kubernetes 1.26
- Helm 3.x

### 监控告警
- Prometheus + Grafana
- ELK Stack (日志分析)
- Sentry (错误追踪)
```

---

#### 章节 3：项目结构

```markdown
## 项目结构

```
my-project/
├── src/                      # 源代码
│   ├── controllers/          # 控制器层
│   │   ├── userController.ts
│   │   └── orderController.ts
│   ├── services/             # 服务层
│   │   ├── UserService.ts
│   │   └── OrderService.ts
│   ├── repositories/         # 数据访问层
│   │   ├── UserRepository.ts
│   │   └── OrderRepository.ts
│   ├── models/               # 数据模型
│   │   └── User.ts
│   ├── middleware/           # 中间件
│   │   ├── auth.ts
│   │   └── errorHandler.ts
│   ├── utils/                # 工具函数
│   │   └── logger.ts
│   └── index.ts              # 入口文件
├── tests/                    # 测试代码
│   ├── unit/                 # 单元测试
│   └── integration/          # 集成测试
├── docs/                     # 文档
├── scripts/                  # 脚本工具
├── config/                   # 配置文件
└── package.json
```

### 目录说明
- `src/controllers/`: 仅处理 HTTP 请求，不包含业务逻辑
- `src/services/`: 核心业务逻辑，可复用
- `src/repositories/`: 数据访问，封装 ORM
- `src/models/`: 数据模型定义
```

---

#### 章节 4：编码规范

```markdown
## 编码规范

### 命名规范

#### 变量和函数
- 使用 camelCase
- 名称要有意义
- 布尔值用 is/has/can 前缀

```typescript
// ✅ 好的
const userName = 'john';
const isLoggedIn = true;
const hasPermission = false;

function calculateTotalPrice() {}
function isValidEmail() {}

// ❌ 不好的
const name = 'john';  // 太模糊
const flag = true;    // 无意义
function calc() {}    // 不完整
```

#### 类名
- 使用 PascalCase
- 名词命名

```typescript
// ✅ 好的
class UserService {}
class OrderController {}

// ❌ 不好的
class userService {}  // 大小写错误
class ManageUser {}   // 动词开头
```

### 注释规范

#### JSDoc 要求
所有公共方法必须有 JSDoc：

```typescript
/**
 * 根据用户名查询用户信息
 * @param username - 用户名（不区分大小写）
 * @param includeInactive - 是否包含已禁用用户，默认 false
 * @returns 用户对象，如果不存在返回 null
 * @throws NotFoundError 当用户不存在时
 */
async getUserByUsername(
  username: string,
  includeInactive = false
): Promise<User | null> {
  // ...
}
```

### 错误处理

#### 统一错误类型
```typescript
// 基础错误类
class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode = 500
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

// 业务错误
class NotFoundError extends AppError {
  constructor(resource: string) {
    super('NOT_FOUND', `${resource} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super('VALIDATION_ERROR', message, 400);
  }
}
```

#### 使用规范
```typescript
// ✅ 好的
try {
  const user = await userService.getUser(id);
  if (!user) {
    throw new NotFoundError('User');
  }
} catch (error) {
  if (error instanceof AppError) {
    throw error;
  }
  logger.error('Unexpected error', error);
  throw new AppError('INTERNAL_ERROR', 'Internal server error');
}

// ❌ 不好的
try {
  const user = await userService.getUser(id);
} catch (error) {
  console.log(error);  // 吞掉错误
}
```
```

---

#### 章节 5：架构原则

```markdown
## 架构原则

### 分层架构

#### Controller 层
**职责：**
- 接收 HTTP 请求
- 参数验证
- 调用 Service 层
- 格式化响应

**不允许：**
- ❌ 直接访问数据库
- ❌ 包含业务逻辑
- ❌ 调用外部 API

```typescript
// ✅ 好的
@RestController('/users')
class UserController {
  constructor(private userService: UserService) {}
  
  @Get('/:id')
  async getUser(@Param('id') id: number) {
    const user = await this.userService.findById(id);
    return { data: user };
  }
}

// ❌ 不好的
@Controller('/users')
class WrongController {
  @Get('/:id')
  async getUser(@Param('id') id: number) {
    // 直接访问数据库
    const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
    return user;
  }
}
```

#### Service 层
**职责：**
- 实现业务逻辑
- 管理事务
- 调用 Repository
- 调用外部服务

**不允许：**
- ❌ 直接处理 HTTP 请求
- ❌ 包含 SQL 语句
```

---

#### 章节 6：测试要求

```markdown
## 测试要求

### 测试策略

#### 测试金字塔
```
        /\
       /  \
      / E2E \     端到端测试 (10%)
     /______\
    /        \
   / Integration \  集成测试 (20%)
  /______________\
 /                \
/    Unit Tests    \ 单元测试 (70%)
------------------
```

### 覆盖率要求
- 语句覆盖率：> 90%
- 分支覆盖率：> 85%
- 函数覆盖率：> 95%

### 测试命名
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('应该成功创建用户并返回用户对象', async () => {
      // ...
    });
    
    it('当用户名已存在时应该抛出 UniqueConstraintError', async () => {
      // ...
    });
    
    it('密码应该自动哈希加密', async () => {
      // ...
    });
  });
});
```
```

---

#### 章节 7：安全规范

```markdown
## 安全规范

### 认证授权

#### JWT Token 规范
- 有效期：7 天
- 刷新机制：Refresh Token (30 天)
- 存储方式：HttpOnly Cookie

```typescript
// Token 生成
const token = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET!,
  { expiresIn: '7d' }
);
```

### 数据安全

#### 密码加密
- 算法：bcrypt
- Salt Rounds: 12

```typescript
const hashedPassword = await bcrypt.hash(password, 12);
const isValid = await bcrypt.compare(password, hashedPassword);
```

#### SQL 注入防护
- ✅ 使用参数化查询
- ❌ 禁止字符串拼接

```typescript
// ✅ 好的
await db.query('SELECT * FROM users WHERE email = ?', [email]);

// ❌ 禁止
await db.query(`SELECT * FROM users WHERE email = '${email}'`);
```

### XSS 防护
- 前端输入必须转义
- 后端输出必须编码
```

---

#### 章节 8：常用命令

```markdown
## 常用命令

### 开发
```bash
npm run dev           # 启动开发服务器（热重载）
npm run build         # 构建生产版本
npm run type-check    # TypeScript 类型检查
```

### 测试
```bash
npm test              # 运行所有测试
npm run test:unit     # 只运行单元测试
npm run test:coverage # 生成覆盖率报告
npm run test:watch    # 监视模式
```

### 代码质量
```bash
npm run lint          # ESLint 检查
npm run lint:fix      # 自动修复格式问题
npm run format        # Prettier 格式化
```

### 数据库
```bash
npm run db:migrate    # 运行迁移
npm run db:seed       # 填充种子数据
npm run db:rollback   # 回滚迁移
```
```

---

## 四、实战案例

### 4.1 完整的 CLAUDE.md 示例

```markdown
# 电商平台后端 - CLAUDE.md

## 项目概述

### 业务背景
这是一个 B2C 电商平台后端系统，为用户提供在线购物体验。
日均订单量 10 万+，注册用户 500 万+。

### 核心功能
1. 用户中心 - 注册登录、个人中心、收货地址
2. 商品系统 - 商品管理、库存管理、搜索推荐
3. 订单系统 - 购物车、下单、订单查询、售后
4. 支付系统 - 多渠道支付、退款、对账
5. 营销系统 - 优惠券、积分、秒杀活动

### 技术栈
- Runtime: Node.js 18.16 LTS
- Language: TypeScript 5.1
- Framework: NestJS 10.x
- Database: MySQL 8.0 (主从复制)
- Cache: Redis 6.2 Cluster
- MQ: RabbitMQ 3.11
- Search: Elasticsearch 8.x
- Deploy: Docker + K8s

## 项目结构

```
ecommerce-backend/
├── src/
│   ├── modules/              # 业务模块
│   │   ├── user/             # 用户模块
│   │   │   ├── user.controller.ts
│   │   │   ├── user.service.ts
│   │   │   ├── user.repository.ts
│   │   │   └── dto/
│   │   ├── product/          # 商品模块
│   │   └── order/            # 订单模块
│   ├── common/               # 公共模块
│   │   ├── decorators/       # 装饰器
│   │   ├── filters/          # 过滤器
│   │   ├── guards/           # 守卫
│   │   └── interceptors/     # 拦截器
│   ├── config/               # 配置
│   └── main.ts               # 入口
├── test/
├── docs/
└── scripts/
```

## 编码规范

### 命名规范
- 文件：kebab-case (user-service.ts)
- 类：PascalCase (UserService)
- 变量：camelCase (userName)
- 常量：UPPER_SNAKE_CASE (MAX_PAGE_SIZE)
- DTO: 后缀 Dto (CreateUserDto)

### 模块规范
每个业务模块必须包含：
1. Controller - HTTP 接口
2. Service - 业务逻辑
3. Repository - 数据访问
4. Entity - 实体定义
5. DTO - 数据传输对象

### 注释规范
公共 API 必须有 JSDoc：
```typescript
/**
 * 创建新用户
 * @param dto - 用户信息
 * @returns 创建的用户对象
 * @throws ConflictException 当用户名已存在
 */
async createUser(dto: CreateUserDto): Promise<User> {
  // ...
}
```

## 架构原则

### 依赖注入
- 使用 NestJS DI
- Service 注入 Repository
- Controller 注入 Service

### 事务管理
- 使用 @Transactional 装饰器
- 事务范围尽可能小
- 避免跨服务事务

### 错误处理
- 使用 HttpException 体系
- 自定义业务异常
- 全局异常过滤器

## 测试要求

### 单元测试
- 覆盖所有 Service 方法
- Mock 外部依赖
- 断言要具体

### 集成测试
- 测试完整流程
- 使用测试数据库
- 清理测试数据

### 覆盖率
- 新增代码覆盖率 > 95%
- 总体覆盖率 > 90%

## 安全规范

### 认证
- JWT Token (7 天有效期)
- Refresh Token (30 天)
- 双重验证可选

### 授权
- RBAC 角色权限控制
- 基于资源的授权
- 数据权限隔离

### 数据安全
- 密码 bcrypt 加密
- 敏感数据脱敏
- SQL 参数化查询

## 性能优化

### 缓存策略
- 热点数据 Redis 缓存
- 缓存穿透：布隆过滤器
- 缓存雪崩：随机 TTL

### 数据库优化
- 读写分离
- 分库分表（按用户 ID）
- 慢查询监控

### 限流降级
- 令牌桶限流
- 熔断降级
- 优雅降级策略

## 常用命令

### 开发
```bash
npm run start:dev      # 开发模式
npm run build          # 构建
npm run lint           # 代码检查
```

### 测试
```bash
npm run test           # 运行测试
npm run test:e2e       # 端到端测试
npm run test:coverage  # 覆盖率
```

### 数据库
```bash
npm run migration:run     # 运行迁移
npm run migration:revert  # 回滚
```

## 监控告警

### 核心指标
- QPS (每秒查询数)
- 响应时间 (P95, P99)
- 错误率
- CPU/Memory 使用率

### 告警阈值
- 错误率 > 1% → P1 告警
- P95 > 500ms → P2 告警
- CPU > 80% → P3 告警

## 部署流程

### 环境
- Development (开发)
- Staging (预发布)
- Production (生产)

### CI/CD
1. Code Review
2. 合并到 develop
3. 自动部署到 Staging
4. 回归测试
5. 手动批准到 Production

## 常见问题 FAQ

### Q: 如何处理分布式事务？
A: 使用最终一致性方案：
1. 本地事务 + 消息队列
2. Saga 模式
3. TCC 模式（复杂场景）

### Q: 如何保证幂等性？
A: 使用唯一请求 ID + 数据库唯一索引

### Q: 如何处理高并发秒杀？
A: 
1. Redis 预减库存
2. 消息队列削峰
3. 限流防刷
```

---

## 五、最佳实践

### ✅ 推荐做法

1. **保持文档更新**
   - 每次重构后更新 CLAUDE.md
   - 新技术引入时补充说明
   - 定期回顾修订

2. **具体明确**
   ```markdown
   ✅ 好的：函数不超过 50 行
   ❌ 不好的：函数要简洁
   ```

3. **提供示例**
   ```markdown
   不仅要说"做什么"，还要展示"怎么做"
   ```

4. **版本化管理**
   - 在 README 中标注 CLAUDE.md 版本
   - 重大变更写迁移指南

### ❌ 避免的错误

1. **过于笼统**
   ```markdown
   ❌ "代码要高质量"
   ✅ "圈复杂度 < 10"
   ```

2. **只有禁止没有引导**
   ```markdown
   ❌ "禁止使用 var"
   ✅ "使用 let/const，var 有提升问题..."
   ```

3. **长期不更新**
   ```markdown
   过时的规范比没有规范更糟糕
   ```

---

## 六、参考资源

### 官方示例
- `plugins/feature-dev/CLAUDE.md`
- `plugins/security-guidance/CLAUDE.md`

### 模板
- `templates/claude-md-template.md`

### 相关文档
- [Hook 系统基础](../02-Hook 系统/01-Hook 系统基础.md)
- [七阶段流程](../05-七阶段流程/01-七阶段流程详解.md)

---

**下一步：** 学习如何 [开发插件](../07-插件系统/01-插件系统详解.md)！
