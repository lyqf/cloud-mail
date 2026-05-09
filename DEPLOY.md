# 部署配置说明

## 账号信息

| 项目 | 值 |
|------|-----|
| Cloudflare 账号（主力） | sdjnhmx@gmail.com |
| Account ID | 0043a46c95bc18954ef875323b4ee22c |
| Cloudflare 账号（备用） | yunhuinfo4@gmail.com |
| Account ID | dfb771ccbe430a8a29dbdc0cd2afd439 |

## API Token

| Token 名称 | Token 值 | 过期时间 | 权限 |
|---|---|---|---|
| flat-recipe-8a73 | 见本地私密记录（勿提交到 git） | 2027-05-09 | Zone DNS Write + Email Routing Rules Write（All zones） |

> Token 用于 CF API 直接操作，主要场景：删除 DNS 记录、配置 Email Routing catch-all 规则

---

## 已部署实例

同一份代码，通过不同 wrangler.toml 部署了多个独立实例。每个实例有独立的 Worker、D1 数据库、KV 命名空间。

### sdjnhmx@gmail.com 账号（D1: email `1e974942`, KV: email `c4c78ac`）

| 域名 | Worker 名称 | 管理员邮箱 |
|---|---|---|
| lgbt.llmailab.xyz | cloud-mail | admin@lgbt.llmailab.xyz |
| willsigil.com | cloud-mail-willsigil | admin@willsigil.com |
| gancaopu.com | cloud-mail-gancaopu | admin@gancaopu.com |

> `cloud-mail` 使用 `wrangler.toml`；独立域名使用各自的 `wrangler-<domain>.toml`

### yunhuinfo4@gmail.com 账号（D1: email `ed730418`, KV: `6ecc5e6f`）

| 域名 | Worker 名称 | 管理员邮箱 |
|---|---|---|
| kaka.locationindoor.cn | cloud-mail | admin@kaka.locationindoor.cn |

---

## 为新域名部署邮箱服务（完整步骤）

### 前提条件

- 域名已托管到 Cloudflare（NS 已指向 CF）
- 已登录正确的 wrangler 账号：`wrangler whoami`

### 第一步：创建独立 D1 和 KV

```bash
wrangler d1 create email-<domain>
wrangler kv namespace create email-<domain>
```

记录输出的 `database_id` 和 KV `id`。

### 第二步：创建 wrangler-<domain>.toml

参考 `wrangler-willsigil.toml` 或 `wrangler-gancaopu.toml`，修改以下字段：

```toml
name = "cloud-mail-<domain>"                 # Worker 名称，每个域名唯一
account_id = "0043a46c95bc18954ef875323b4ee22c"

[[d1_databases]]
database_name = "email-<domain>"
database_id = "<第一步获取的 D1 ID>"

[[kv_namespaces]]
id = "<第一步获取的 KV ID>"

[vars]
domain = ["<domain>"]
admin = "admin@<domain>"
jwt_secret = "5e10cdad698de1f32fdb331e68b88cef04d2f980cbec123e49e6dc3c123818c7"

[[routes]]
pattern = "<domain>"
custom_domain = true
```

### 第三步：处理 DNS 冲突（如有）

部署时若报错 `Hostname already has externally managed DNS records`，需先删除冲突记录：

```bash
TOKEN="<flat-recipe-8a73 token 值>"
ZONE_ID="<zone_id>"   # 从下面命令获取

# 查 zone_id
curl -s "https://api.cloudflare.com/client/v4/zones?name=<domain>" \
  -H "Authorization: Bearer $TOKEN" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['result'][0]['id'])"

# 查冲突 DNS 记录
curl -s "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records?per_page=100" \
  -H "Authorization: Bearer $TOKEN" | python3 -c "
import json,sys
for r in json.load(sys.stdin).get('result') or []:
    print(r['id'], r['type'], r['name'], r.get('content',''))
"

# 删除记录（替换 <record_id>）
curl -s -X DELETE "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/<record_id>" \
  -H "Authorization: Bearer $TOKEN" | python3 -c "import json,sys; print(json.load(sys.stdin)['success'])"
```

### 第四步：部署

```bash
wrangler deploy --config wrangler-<domain>.toml
```

### 第五步：初始化数据库

```bash
curl -s "https://<domain>/api/init/5e10cdad698de1f32fdb331e68b88cef04d2f980cbec123e49e6dc3c123818c7"
# 返回 success 即可
```

### 第六步：配置 Email Routing（接收邮件）

```bash
TOKEN="<flat-recipe-8a73 token 值>"
ZONE_ID="<zone_id>"
WORKER_NAME="cloud-mail-<domain>"

# 启用 Email Routing（会自动添加 MX/SPF/DKIM 记录）
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/email/routing/enable" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print('enable:', d['success'])"

# 添加 DKIM 记录
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d "{\"type\":\"TXT\",\"name\":\"cf2024-1._domainkey.<domain>\",\"content\":\"v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAiweykoi+o48IOGuP7GR3X0MOExCUDY/BCRHoWBnh3rChl7WhdyCxW3jgq1daEjPPqoi7sJvdg5hEQVsgVRQP4DcnQDVjGMbASQtrY4WmB1VebF+RPJB2ECPsEDTpeiI5ZyUAwJaVX7r6bznU67g7LvFq35yIo4sdlmtZGV+i0H4cpYH9+3JJ78km4KXwaf9xUJCWF6nxeD+qG6Fyruw1Qlbds2r85U9dkNDVAS3gioCvELryh1TxKGiVTkg4wqHTyHfWsp7KD3WQHYJn0RyfJJu6YEmL77zonn7p2SRMvTMP3ZEXibnC9gz3nnhR6wcYL8Q7zXypKTMD58bTixDSJwIDAQAB\",\"ttl\":1}" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print('dkim:', d['success'])"

# 设置 catch-all 规则 → Worker
curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/email/routing/rules/catch_all" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d "{\"matchers\":[{\"type\":\"all\"}],\"actions\":[{\"type\":\"worker\",\"value\":[\"$WORKER_NAME\"]}],\"enabled\":true,\"name\":\"catch-all\"}" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print('catch-all:', d['success'])"
```

---

## 更换现有实例域名（快速步骤）

适用于 `wrangler.toml`（`cloud-mail` worker）的域名切换：

1. 确认当前登录账号与 wrangler.toml 中 `account_id` 一致
2. 修改 wrangler.toml 三处：`domain`、`admin`、`[[routes]].pattern`
3. `wrangler deploy`
4. 初始化数据库（新域名首次部署需要）
5. 配置 Email Routing（第六步）

---

## 注意事项

- `account_id` 必须填，否则 wrangler 会用缓存的旧账号 ID
- D1/KV ID 是账号级别资源，切换账号时必须重新查询
- `jwt_secret` 一旦有用户登录就不能修改，改了所有 token 失效
- llmailab.xyz 子域名的 Email Routing 共用同一个 CF zone，catch-all 规则指向最新部署的 Worker
- 独立域名（willsigil.com、gancaopu.com）各有独立的 wrangler-*.toml，部署时用 `--config` 指定
