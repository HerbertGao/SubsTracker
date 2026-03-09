## 上下文

`addGregorianPeriod` 函数依赖外部传入的 `anchorDay` 参数来正确恢复月末日期。当前创建和编辑订阅时会将 anchorDay 持久化到订阅对象中，但自动续费（scheduler.js）和手动续订（subscriptions.js manualRenewSubscription）两条路径只在局部变量中使用 anchorDay，不回写到订阅对象。对于无 anchorDay 字段的 v1 老订阅，第一次续费 fallback 到 `expiryDate.getDate()` 正确，但续费后 expiryDate 被钳位，第二次 fallback 就取到了错误值。

## 目标 / 非目标

**目标：**
- 在自动续费和手动续订路径中，将 anchorDay 回写到持久化的订阅对象中
- 确保 v1 老订阅（无 anchorDay 字段）在首次续费后自动补上正确的 anchorDay

**非目标：**
- 不做数据迁移脚本，老数据由用户手动修补
- 不修改创建/编辑订阅路径（已正确持久化）
- 不修改前端代码（前端只读不写 anchorDay）

## 决策

### 使用 `subscription.anchorDay || anchorDay` 而非无条件覆盖

在 `updatedSubscription` 对象中使用 `anchorDay: subscription.anchorDay || anchorDay`，而非直接写 `anchorDay: anchorDay`。

**理由**：如果订阅已有正确的 anchorDay（新建订阅或已编辑过的订阅），应保留原值。仅在字段缺失时（v1 老数据），才用当次 fallback 计算的值补上。这避免了因 newStartDate 不同（如手动续订指定了 paymentDate）而意外覆盖用户期望的锚定日。

**替代方案**：每次续费时从 expiryDate 重新计算 anchorDay — 拒绝，因为续费后 expiryDate 可能已被钳位，重新计算会得到错误值。

## 风险 / 权衡

- **风险**：v1 老订阅如果已经经历过多次续费导致 expiryDate 漂移（如 31→28），首次应用此修复时 fallback 仍会取到 28 → 需要用户手动修正数据中的 expiryDate 或 anchorDay
- **缓解**：用户已表示愿意手动修改数据，此修复确保修正后不再漂移
