## 为什么

本仓库是公开仓库，当前 `wrangler.toml` 中包含 KV 命名空间 ID 的占位符字段（`id` 和 `preview_id`）。一旦开发者 fork 后填入真实 ID 并提交，这些敏感信息就会暴露在 Git 历史中。需要一种方案让敏感配置完全不出现在仓库中。

## 变更内容

- **修改**：`wrangler.toml` 移除 `[[kv_namespaces]]` 中的 `id` 和 `preview_id` 字段，仅保留 `binding` 名称；KV 命名空间绑定通过 Cloudflare Dashboard 配置
- **新增**：在 `.gitignore` 中添加 `.dev.vars`，用于本地开发时存放敏感配置
- **修改**：README 中新增 Cloudflare Workers Git 集成部署说明，指引用户通过 Dashboard 绑定仓库并配置 KV 绑定

## 功能 (Capabilities)

### 新增功能
- `cloudflare-git-deploy`: Cloudflare Workers Git 集成部署方式——通过 Cloudflare Dashboard 绑定 GitHub 仓库，推送时自动部署，敏感配置在 Dashboard 中管理

### 修改功能
（无现有规范需要修改——部署方式变更不影响应用行为）

## 影响

- **文件**：`wrangler.toml`（移除敏感 ID 字段）、`.gitignore`（添加 `.dev.vars`）、`README.md`（新增部署说明）
- **部署流程**：通过 Cloudflare Workers 的 Git 集成功能绑定 GitHub 仓库，推送到 master 时 Cloudflare 自动拉取并部署；KV 绑定在 Dashboard 的 Settings > Bindings 中配置
- **兼容性**：不影响应用代码和 API 行为；一键部署按钮仍保留
