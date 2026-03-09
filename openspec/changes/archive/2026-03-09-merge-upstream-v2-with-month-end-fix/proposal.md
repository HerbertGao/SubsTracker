## 为什么

上游仓库 (wangwangit/SubsTracker) 发布了 v2 模块化重构，将单文件 `index.js`（约 7700 行）拆分为 `src/` 目录下的多模块架构，同时新增了 Gotify 通知、安全加固、deploy-safe 工作流等功能。我们的 fork 分支 (herbertgao) 之前独立实现了月末日期钳位修复 (`addGregorianPeriod` + `anchorDay`) 和安全配置优化，需要合并上游最新代码以获取新功能，同时将我们的修复移植到新的模块化架构中。

## 变更内容

- **BREAKING** 项目架构从单文件 `index.js` 迁移到 `src/` 模块化目录结构（来自上游 v2）
- 将 `addGregorianPeriod()` 月末日期钳位函数从旧 `index.js` 移植到 `src/core/time.js`，并在 `src/data/subscriptions.js`、`src/services/scheduler.js`、`src/views/adminPage.html` 中替换所有内联周期计算
- 在新模块化代码的订阅对象中保留 `anchorDay` 字段支持
- 合并 `wrangler.toml` 冲突：接受上游的 `main: "src/index.js"` 入口和每小时 cron，保留我们的 KV binding-only 配置
- 合并 `README.md` 冲突：接受上游精简后的文档，保留我们的一键部署按钮和 Git 集成部署说明
- 接受上游新增的 Gotify 通知渠道、安全修复（secrets masking、/debug 保护、crypto RNG）、deploy-safe 工作流等

## 功能 (Capabilities)

### 新增功能

- `v2-modular-architecture`: 上游 v2 模块化架构适配，将 `index.js` 拆分为 `src/` 下的 core、data、services、views、api 等模块

### 修改功能

- `month-end-date-clamp`: 将已有的月末日期钳位逻辑（`addGregorianPeriod` + `anchorDay`）从旧单文件架构移植到新的 `src/` 模块化架构中，函数定义迁移至 `src/core/time.js`，调用点分布在 `src/data/subscriptions.js`、`src/services/scheduler.js`、`src/views/adminPage.html`
- `cloudflare-git-deploy`: 在上游重写的 README 中保留 Git 集成部署说明和一键部署按钮（指向 fork 仓库），`wrangler.toml` 保持 binding-only 模式

## 影响

- **代码结构**：根目录 `index.js` 被删除，所有逻辑迁移到 `src/` 目录，`wrangler.toml` 入口改为 `src/index.js`
- **新增文件**：`src/` 下约 30 个模块文件、`scripts/setup-kv.js`、`package.json` 更新
- **后端模块**：`src/core/time.js`（新增 `addGregorianPeriod` 导出）、`src/data/subscriptions.js`（3 处内联计算替换 + anchorDay 字段）、`src/services/scheduler.js`（1 处内联计算替换）
- **前端模板**：`src/views/adminPage.html`（新增前端 `addGregorianPeriod` 副本 + 2 处内联计算替换）
- **部署配置**：`wrangler.toml` 入口、cron 频率变更；`package.json` 新增 npm scripts
- **通知系统**：新增 Gotify 通知渠道（来自上游）
