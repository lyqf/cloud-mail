# 部署配置说明

## 账号信息

| 项目 | 值 |
|------|-----|
| Cloudflare 账号 | sdjnhmx@gmail.com |
| Account ID | 0043a46c95bc18954ef875323b4ee22c |

## 已部署实例

同一份代码，通过修改 `wrangler.toml` 部署了多个独立实例。每个实例有独立的域名、D1 数据库、KV 命名空间。

### 实例：router.llmailab.xyz

| 项目 | 值 |
|------|-----|
| Worker 名称 | cloud-mail |
| 前端访问地址 | https://router.llmailab.xyz |
| 邮箱格式 | xxx@router.llmailab.xyz |
| 管理员邮箱 | admin@router.llmailab.xyz |
| D1 数据库名 | email（ID: 1e974942-f0b7-4ad4-aa84-457643436f8d）|
| KV 命名空间名 | email（ID: c4c78acfaddf4222bb564b81f2938969）|

## wrangler.toml 各字段说明

```toml
name = "cloud-mail"                          # Worker 名称，对应 Cloudflare Dashboard 里的 Worker
account_id = "..."                           # CF 账号 ID，必须填，否则 wrangler 会用缓存的旧账号

[[d1_databases]]
database_name = "email"                      # D1 数据库名
database_id = "..."                          # D1 数据库 ID，从 wrangler d1 list 获取

[[kv_namespaces]]
id = "..."                                   # KV 命名空间 ID，从 wrangler kv namespace list 获取

[vars]
domain = ["router.llmailab.xyz"]             # 邮件域名，注册邮箱时 @ 后面必须是这里的值
admin = "admin@router.llmailab.xyz"          # 管理员账号，首次使用系统会创建此账号
jwt_secret = "..."                           # JWT 签名密钥，部署后不要修改，否则所有登录失效

[[routes]]
pattern = "router.llmailab.xyz"              # Web 访问域名（custom domain）
custom_domain = true
```

## 更换域名的完整步骤

1. **修改 wrangler.toml**：同步修改 `domain`、`admin`、`[[routes]].pattern` 三处
2. **确认资源 ID 正确**：`database_id` 和 KV `id` 必须是当前登录账号下的资源
   - 查 D1：`wrangler d1 list`
   - 查 KV：`wrangler kv namespace list`
3. **部署**：`wrangler deploy`
4. **配置 Email Routing**：在 Cloudflare Dashboard → Email → Email Routing 里，确认新域名的子域名已启用，且 catch-all 规则指向对应 Worker

## 注意事项

- `wrangler.toml` 里的 D1/KV ID 是账号级别的资源，不同账号下的 ID 不通用
- `account_id` 不填时 wrangler 会读本地缓存，多账号环境下容易用错账号
- Email Routing 的子域名（如 `router.llmailab.xyz`）需要在 Dashboard 里单独 Add subdomain 开启，开启后 Cloudflare 会自动创建 catch-all → Worker 的规则
- `jwt_secret` 一旦有用户登录就不能改，改了所有 token 失效
