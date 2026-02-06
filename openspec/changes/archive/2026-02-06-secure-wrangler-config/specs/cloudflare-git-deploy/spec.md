## ADDED Requirements

### 需求：wrangler.toml 不包含敏感信息
`wrangler.toml` 中禁止包含真实的 KV namespace ID 或其他敏感凭据，`[[kv_namespaces]]` 仅保留 `binding` 名称。

#### 场景：wrangler.toml 中无 KV namespace ID
- **当** 检查仓库中的 `wrangler.toml` 文件
- **那么** `[[kv_namespaces]]` 下必须仅有 `binding` 字段，禁止出现 `id` 或 `preview_id` 字段

### 需求：本地开发敏感配置隔离
开发者在本地开发时必须能通过 `.dev.vars` 文件管理敏感配置，该文件禁止提交到版本控制。

#### 场景：.dev.vars 被 .gitignore 忽略
- **当** 检查 `.gitignore` 文件
- **那么** 必须包含 `.dev.vars` 条目

### 需求：README 包含 Cloudflare Git 集成部署指引
README 必须包含通过 Cloudflare Workers Git 集成绑定仓库的部署说明，包括 KV 绑定配置步骤。

#### 场景：README 包含 Git 集成部署章节
- **当** 用户阅读 README
- **那么** 必须能找到 Cloudflare Workers 绑定 GitHub 仓库的步骤说明，以及在 Dashboard 中配置 KV Bindings 的指引
