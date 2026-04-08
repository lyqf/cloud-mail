# 运维操作手册

## D1 数据库结构说明

`account` 表存储所有邮箱账号，包含：
- **主账号**（如 `admin@mail.llmailab.xyz`）
- **别名邮箱**（同一 `user_id` 下的其他邮件地址）

两者都在同一张表，通过 `is_del` 字段软删除（0=正常，1=已删除）。

---

## 清理指定用户的别名邮箱

> **注意**：必须用 `AND email != '主账号'` 排除主账号，否则主账号也会被软删除。

### 1. 查询用户 ID 和别名数量

```bash
npx wrangler d1 execute email --remote --command \
  "SELECT u.user_id, u.email, COUNT(a.account_id) as alias_count \
   FROM user u LEFT JOIN account a ON u.user_id = a.user_id AND a.is_del = 0 \
   WHERE u.email = 'xxx@mail.llmailab.xyz' GROUP BY u.user_id"
```

### 2. 软删除别名（保留主账号）

```bash
# 把 user_id=5 和主账号邮件地址替换为实际值
npx wrangler d1 execute email --remote --command \
  "UPDATE account SET is_del = 1 \
   WHERE user_id = 5 AND email != 'admin@mail.llmailab.xyz'"
```

### 3. 验证

```bash
npx wrangler d1 execute email --remote --command \
  "SELECT email, is_del FROM account \
   WHERE user_id = 5 AND is_del = 0"
```

---

## 恢复误删的主账号

如果主账号被误删（登录后显示账号不存在/已注销），先找到 account_id：

```bash
npx wrangler d1 execute email --remote --command \
  "SELECT account_id, email, is_del FROM account \
   WHERE email = 'admin@mail.llmailab.xyz'"
```

然后恢复：

```bash
npx wrangler d1 execute email --remote --command \
  "UPDATE account SET is_del = 0 WHERE account_id = 64"
```
