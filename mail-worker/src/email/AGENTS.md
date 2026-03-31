<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# email

## Purpose
邮件接收处理模块，通过 Cloudflare Email Routing 接收原始 MIME 邮件，解析邮件内容（使用 postal-mime），提取附件，并将邮件数据和附件存储到 D1 数据库和 R2 存储。

## Key Files

| File | Description |
|------|-------------|
| `email.js` | Email Routing 事件处理器，解析 MIME、权限验证、邮件接收流程、转发逻辑 |

## Entry Point

**`email(message, env, ctx)`**
- Cloudflare Email Routing handler
- 参数：
  - `message`: 邮件对象（包含 raw 流、to、forward 等方法）
  - `env`: Worker 环境变量（D1、R2、KV、API Key 等）
  - `ctx`: 执行上下文
- 处理流程：
  1. 查询接收设置和转发规则
  2. 读取原始 MIME 内容（流式读取）
  3. 使用 postal-mime 解析邮件结构
  4. 查询账户和用户信息
  5. 权限验证（域名授权、禁用列表）
  6. 构建邮件参数（收件人、发件人、主题、内容等）
  7. 提取并处理附件（生成唯一 key、计算大小）
  8. 调用 emailService.receive 保存邮件和 CID 附件
  9. 调用 attService.addAtt 保存非 CID 附件到 R2
  10. 更新邮件状态为已完成
  11. 根据设置执行转发（Telegram、其他邮箱）

## Key Operations

### 邮件解析
```javascript
// 1. 流式读取原始 MIME 内容
const reader = message.raw.getReader()
let content = ''
while (true) {
  const { done, value } = await reader.read()
  if (done) break
  content += new TextDecoder().decode(value)
}

// 2. 使用 postal-mime 解析
const email = await PostalMime.parse(content)
// 返回结构：
// {
//   from: { address, name }
//   to: [{ address, name }, ...]
//   cc: [...]
//   bcc: [...]
//   subject: string
//   html: string
//   text: string
//   attachments: [{ filename, content, contentId, ... }]
//   messageId: string
//   inReplyTo: string
//   references: string
// }
```

### 权限验证
- **账户存在性**：查询 accountService 验证收件人邮箱是否存在
- **服务状态**：检查全局 receive 设置是否开启（关闭则拒收）
- **域名授权**：非管理员用户需验证 roleService.hasAvailDomainPerm
- **发件人禁用列表**：检查 roleService.isBanEmail

### 附件处理
- **附件分类**：区分 CID 附件（内联图片）和普通附件
- **唯一键生成**：`ATTACHMENT_PREFIX + hash(content) + ext`
- **大小计算**：`item.content.length ?? item.content.byteLength`
- **CID 附件**：随邮件主体一起保存（emailService.receive）
- **普通附件**：单独调用 attService.addAtt 上传到 R2

### 转发规则
- **转发到 Telegram**：检查 tgBotStatus 和 tgChatId
- **转发到邮箱**：检查 forwardStatus 和 forwardEmail 列表
- **规则过滤**：若启用规则模式，只转发 ruleEmail 中指定的收件人

## Data Flow

```
Cloudflare Email Routing
  ↓
email() handler
  ├─ 读取原始 MIME → PostalMime.parse()
  ├─ accountService.selectByEmailIncludeDel() 查询账户
  ├─ 权限验证 (roleService)
  ├─ 邮件参数构建
  ├─ 附件处理
  │  ├─ emailService.receive() 保存邮件 + CID 附件
  │  └─ attService.addAtt() 保存普通附件到 R2
  ├─ 邮件状态更新 (completeReceive)
  └─ 条件转发
     ├─ Telegram 通知 (telegramService)
     └─ 邮箱转发 (message.forward)
```

## Dependencies

### Internal
- `service/email-service` - 邮件业务逻辑（receive、completeReceive）
- `service/account-service` - 账户查询
- `service/setting-service` - 设置查询（receive、tgBotStatus、forwardStatus 等）
- `service/att-service` - 附件管理
- `service/role-service` - 权限管理（hasAvailDomainPerm、isBanEmail）
- `service/user-service` - 用户信息查询
- `service/telegram-service` - Telegram 通知
- `utils/email-utils` - 邮件工具（getName 等）
- `utils/file-utils` - 文件工具（getBuffHash、getExtFileName）
- `const/entity-const` - 业务常量（emailConst、settingConst 等）

### External
- `postal-mime` - MIME 邮件解析库

## For AI Agents

### Working In This Directory
- 处理 Cloudflare Email Routing 的 `message` 对象
- 邮件解析依赖 postal-mime 库（解析后的对象结构明确）
- 权限验证链：账户存在 → 域名授权 → 发件人黑名单
- 附件处理：区分 CID 附件和普通附件的存储方式
- 转发规则支持多目标（Telegram、邮箱列表）

### Error Handling
- 服务关闭时 `message.setReject('Service suspended')`
- 账户不存在时（若禁用无收件人）`message.setReject('Recipient not found')`
- 域名未授权 `message.setReject('The recipient is not authorized to use this domain.')`
- 发件人被禁用 `message.setReject('The recipient is disabled from receiving emails.')`
- 附件处理异常：记录错误但不中断邮件保存

### Testing Requirements
- Mock `message.raw.getReader()` 返回 MIME 流
- Mock `PostalMime.parse()` 返回完整邮件对象
- Mock `accountService`、`settingService`、`roleService` 等依赖
- 验证邮件参数构建正确性（收件人、发件人、主题、附件列表）
- 验证转发逻辑（Telegram、邮箱）

## Common Patterns

```javascript
// 权限验证链
if (account && userRow.email !== env.admin) {
  const { banEmail, availDomain } = await roleService.selectByUserId({ env }, account.userId)

  if (!roleService.hasAvailDomainPerm(availDomain, message.to)) {
    message.setReject('The recipient is not authorized to use this domain.')
    return
  }

  if (roleService.isBanEmail(banEmail, email.from.address)) {
    message.setReject('The recipient is disabled from receiving emails.')
    return
  }
}

// 附件处理
for (let item of email.attachments) {
  let attachment = { ...item }
  attachment.key = constant.ATTACHMENT_PREFIX + await fileUtils.getBuffHash(attachment.content) + fileUtils.getExtFileName(item.filename)
  attachment.size = item.content.length ?? item.content.byteLength
  attachments.push(attachment)
  if (attachment.contentId) {
    cidAttachments.push(attachment)
  }
}
```

<!-- MANUAL: -->
