## 上下文

当前部署有两种路径：
1. **一键部署按钮**（Deploy to Cloudflare Workers）— 通过 Cloudflare Dashboard 完成
2. **手动部署** — 用户直接在 Dashboard 粘贴 `index.js` 代码

`wrangler.toml` 当前硬编码了占位符 KV namespace ID（`id` 和 `preview_id`）。开发者如果 fork 并填入真实 ID，这些 ID 会进入 Git 历史。

## 目标 / 非目标

**目标：**
- 敏感配置（KV namespace ID）完全不出现在仓库文件中
- 利用 Cloudflare Workers 的 Git 集成功能，绑定 GitHub 仓库后推送自动部署
- KV 绑定在 Cloudflare Dashboard 中配置，不依赖 `wrangler.toml` 中的 `id` 字段
- 保留一键部署按钮（不受影响）

**非目标：**
- 不修改应用代码逻辑

## 决策

### D1：使用 Cloudflare Workers Git 集成

**选择**：通过 Cloudflare Dashboard 将 Worker 绑定到 GitHub 仓库，推送时由 Cloudflare 自动拉取代码并部署。

**理由**：
- 原生集成，无需维护额外的 CI/CD 工作流文件
- KV 绑定、环境变量等敏感配置全部在 Dashboard 中管理，不需要出现在代码仓库中
- Cloudflare 读取 `wrangler.toml` 获取 Worker 名称、兼容性日期、Cron 等非敏感配置；KV 绑定可在 Dashboard 的 Settings > Bindings 中覆盖

**替代方案：**
- `wrangler.toml` 加入 `.gitignore` + 提供 `.example`：打破一键部署按钮流程

### D2：wrangler.toml 中移除 KV namespace ID

**选择**：从 `[[kv_namespaces]]` 中移除 `id` 和 `preview_id` 字段，仅保留 `binding = "SUBSCRIPTIONS_KV"`。

**理由**：
- Cloudflare Git 集成部署时，KV 绑定通过 Dashboard Settings > Bindings 配置，不需要 `wrangler.toml` 中的 `id`
- `binding` 字段保留，确保代码中 `env.SUBSCRIPTIONS_KV` 的引用名称一致
- 本地 `wrangler dev` 开发时，用户需自行在本地 `wrangler.toml` 添加 `id` 或通过 `--local` 模式

## 风险 / 权衡

| 风险 | 缓解措施 |
|------|----------|
| 本地 `wrangler dev` 缺少 KV ID 可能报错 | README 中说明本地开发需自行配置或使用 `--local` 模式 |
| 用户不熟悉 Cloudflare Git 集成 | README 中提供清晰的绑定步骤说明 |
| `wrangler.toml` 无 `id` 字段时 `wrangler deploy` CLI 会报错 | 仓库方案不推荐 CLI 直接部署，推荐 Git 集成或一键部署 |
