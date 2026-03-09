## 上下文

上游 v2 将单文件 `index.js`（约 7700 行）重构为 `src/` 下的模块化架构：`src/core/`（基础工具）、`src/data/`（数据层）、`src/services/`（业务服务）、`src/views/`（HTML 模板）、`src/api/`（路由与处理器）。我们的 fork 之前在旧单文件架构上实现了 `addGregorianPeriod` 月末日期钳位修复，现在需要将修复移植到新架构并解决合并冲突。

## 目标 / 非目标

**目标：**
- 成功合并上游 v2 模块化代码，获取 Gotify 通知、安全加固等新功能
- 将 `addGregorianPeriod` 函数和 `anchorDay` 字段移植到新模块化架构中，保持月末日期钳位功能完整
- 保留我们 fork 的自定义配置（部署链接、Git 集成部署、KV binding-only）

**非目标：**
- 不修改上游 v2 的模块化架构设计
- 不在此变更中添加新功能（仅移植已有修复）
- 不重构上游代码中我们未涉及的部分

## 决策

### 1. `addGregorianPeriod` 放置在 `src/core/time.js` 中

**选择**：将函数定义放在 `src/core/time.js` 并 export，各模块通过 import 引用。

**替代方案**：
- 创建独立的 `src/core/gregorian.js` — 拒绝，因为该函数本质是时间计算工具，与 `time.js` 的职责一致，无需单独建文件。
- 在每个使用模块中复制函数 — 拒绝，违反 DRY 原则。

**理由**：`time.js` 已是时间工具函数的聚合点，新增一个日期计算函数是自然的扩展。

### 2. 前端 HTML 中保留独立的函数副本

**选择**：在 `src/views/adminPage.html` 的 `<script>` 中定义独立的 `addGregorianPeriod` 副本。

**理由**：前端代码以 HTML 模板字符串形式嵌入，无法使用 ES Module import。这与旧架构的处理方式一致，也是上游 v2 的前端代码组织方式。通过注释标记前后端需同步。

### 3. 合并策略：逐文件手动解决

**选择**：使用 `git merge --no-commit` 后手动处理三个冲突文件。

**替代方案**：
- `git rebase` — 拒绝，会重写历史，增加 force-push 风险。
- Cherry-pick 上游 commits — 拒绝，上游有 17 个 commit（含多个 merge commits），逐个 cherry-pick 工作量大且容易遗漏。

### 4. `wrangler.toml` 保留 KV binding 声明

**选择**：保留 `[[kv_namespaces]] binding = "SUBSCRIPTIONS_KV"` 但不含 `id`/`preview_id`。

**理由**：我们使用 Cloudflare Git 集成部署（非 `npm run setup`），需要 toml 中声明 binding 名称让 Cloudflare 识别，实际 namespace ID 在 Dashboard 中配置。

## 风险 / 权衡

- **前后端函数同步风险** → 通过注释 `// 此函数需前后端同步` 提醒开发者，修改时需同步更新两处
- **anchorDay 向后兼容** → 对无 `anchorDay` 字段的历史订阅，代码通过 `subscription.anchorDay || date.getDate()` 回退，不影响现有数据
- **上游后续更新冲突** → 由于代码已在不同位置（我们修改了 `src/core/time.js` 等），后续合并可能需要再次手动处理这些文件
