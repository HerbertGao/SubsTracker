## 1. 配置文件修改

- [x] 1.1 修改 `wrangler.toml`：移除 `[[kv_namespaces]]` 中的 `id` 字段，仅保留 `binding = "SUBSCRIPTIONS_KV"`
- [x] 1.2 确认 `.gitignore` 中已包含 `.dev.vars`

## 2. 文档更新

- [x] 2.1 在 README.md 中新增 Cloudflare Workers Git 集成部署章节，说明绑定仓库步骤和 Dashboard KV 配置方式
