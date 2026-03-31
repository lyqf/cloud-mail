<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# dao

## Purpose
数据访问层（Data Access Object），使用 Drizzle ORM 封装 D1 数据库操作。定义数据库 schema、提供通用查询接口、统计分析查询。

## Key Files

| File | Description |
|------|-------------|
| `analysis-dao.js` | 统计分析查询：用户数、邮件数、账户数、日活等 |

## Directory Structure

```
dao/
├── AGENTS.md                 # 本文档
└── analysis-dao.js           # 统计分析数据访问对象
```

## Drizzle ORM Schema

所有 schema 定义在 `src/entity/` 下，按表名分文件（见 `src/AGENTS.md`）：

| Schema | Entity File | Description |
|--------|-------------|-------------|
| `user` | `entity/user.js` | 用户表（userId, email, password, status, createTime 等） |
| `account` | `entity/account.js` | 邮箱账户表（accountId, email, userId, status 等） |
| `email` | `entity/email.js` | 邮件表（emailId, sendEmail, toEmail, type, status, isDel 等） |
| `attachments` | `entity/att.js` | 附件表（attId, emailId, key, filename, size 等） |
| 其他表 | `entity/*.js` | role, permission, oauth, setting, verify_record 等 |

## Public API

### analysisDao

统计分析数据访问对象，暴露以下方法：

#### `numberCount(c)`
获取全局统计数据：用户总数、邮件总数、删除数、正常数、账户总数等。

**参数：**
- `c` {Object} - Hono 上下文对象

**返回：**
- {Promise<Object>} 包含统计结果：
  ```javascript
  {
    receiveTotal: number,           // 总接收邮件数
    sendTotal: number,              // 总发送邮件数
    delReceiveTotal: number,        // 已删除接收邮件数
    delSendTotal: number,           // 已删除发送邮件数
    normalReceiveTotal: number,     // 正常接收邮件数
    normalSendTotal: number,        // 正常发送邮件数
    userTotal: number,              // 用户总数
    normalUserTotal: number,        // 正常用户数
    delUserTotal: number,           // 已删除用户数
    accountTotal: number,           // 账户总数
    normalAccountTotal: number,     // 正常账户数
    delAccountTotal: number         // 已删除账户数
  }
  ```

**SQL 逻辑：**
- 从 `email` 表统计邮件（按 type 区分接收/发送，按 is_del 区分删除状态）
- 从 `user` 表统计用户（按 is_del 区分删除状态）
- 从 `account` 表统计账户（按 is_del 区分删除状态）
- 多表交叉连接（CROSS JOIN）获取全量统计

#### `userDayCount(c, diffHours)`
按日期统计用户创建数（最近 15 天）。

**参数：**
- `c` {Object} - Hono 上下文对象
- `diffHours` {number} - 时区差值（小时），用于日期转换

**返回：**
- {Promise<Array<Object>>} 日期和创建数数组：
  ```javascript
  [
    { date: '2026-03-10', total: 5 },
    { date: '2026-03-11', total: 3 }
  ]
  ```

#### `receiveDayCount(c, diffHours)`
按日期统计接收邮件数（最近 15 天）。

**参数：**
- `c` {Object} - Hono 上下文对象
- `diffHours` {number} - 时区差值（小时）

**返回：**
- {Promise<Array<Object>>} 日期和接收邮件数数组

#### `sendDayCount(c, diffHours)`
按日期统计发送邮件数（最近 15 天）。

**参数：**
- `c` {Object} - Hono 上下文对象
- `diffHours` {number} - 时区差值（小时）

**返回：**
- {Promise<Array<Object>>} 日期和发送邮件数数组

## For AI Agents

### Working In This Directory

#### 1. 创建新 DAO 文件
- 按功能划分 DAO 文件（如 `user-dao.js`, `email-dao.js`）
- 每个 DAO 文件导出一个对象，包含相关查询方法
- 使用 Drizzle ORM API 进行查询

#### 2. 使用 Drizzle ORM 查询
```javascript
import orm from '../entity/orm';
import { user } from '../entity/user';
import { eq, and } from 'drizzle-orm';

const userDao = {
  async findById(c, userId) {
    const db = orm(c);
    const result = await db.select().from(user)
      .where(eq(user.userId, userId));
    return result[0];
  },

  async findByEmail(c, email) {
    const db = orm(c);
    return await db.select().from(user)
      .where(eq(user.email, email));
  }
};
```

#### 3. 复杂查询
- 多表连接：使用 `leftJoin`, `innerJoin` 等
- 条件查询：使用 `eq`, `and`, `or`, `gt`, `like` 等
- 原始 SQL：使用 `db.prepare(sql).all()` 处理复杂统计查询

#### 4. 性能考虑
- 在 DAO 层按需选择字段（avoid `SELECT *`）
- 使用索引优化查询（在 schema 定义中标记主键、外键）
- 复杂统计查询使用原始 SQL 而非 ORM 链式调用

### Testing Requirements

DAO 层应通过单元测试验证：
- 查询返回正确的数据结构
- 筛选条件生效
- 异常情况处理（如无结果时的返回值）

```bash
pnpm test  # 运行 DAO 单元测试
```

### Common Patterns

#### 简单查询
```javascript
const userDao = {
  async getAll(c) {
    const db = orm(c);
    return await db.select().from(user);
  }
};
```

#### 条件查询
```javascript
async findActive(c) {
  const db = orm(c);
  return await db.select().from(user)
    .where(eq(user.status, 0)); // status 0 = 活跃
}
```

#### 分页查询
```javascript
async paginate(c, page, pageSize) {
  const db = orm(c);
  return await db.select().from(user)
    .limit(pageSize)
    .offset((page - 1) * pageSize);
}
```

#### 统计查询（复杂 SQL）
```javascript
async countByStatus(c) {
  const { results } = await c.env.db.prepare(`
    SELECT status, COUNT(*) as count
    FROM user
    GROUP BY status
  `).all();
  return results;
}
```

## Dependencies

### Internal
- `src/entity/` - Drizzle schema 定义（user, account, email, att 等）
- `src/entity/orm.js` - ORM 实例化函数

### External
- `drizzle-orm` 0.42+ - ORM 框架
- `drizzle-orm/d1` - D1 适配器
- `drizzle-orm/sqlite-core` - SQLite 核心库

## Key Concepts

### Hono 上下文对象 (c)
- `c.env.db` - D1 数据库连接（D1 原生 API）
- `c.env.orm_log` - Drizzle 查询日志（可选）

### 数据库架构
- **主键自增**：大多数表使用 `autoIncrement` 主键
- **时间戳**：创建时间使用 `sql\`CURRENT_TIMESTAMP\` 默认值
- **软删除**：通过 `is_del` 字段标记删除（0=正常，1=删除）
- **外键关系**：userId, accountId, emailId 为外键

### 时区处理
统计查询支持 `diffHours` 参数调整时区，用于全球用户的本地时间统计。

<!-- MANUAL: -->
