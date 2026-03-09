## 1. 自动续费路径修复（scheduler）

- [x] 1.1 在 `src/services/scheduler.js` 的 `updatedSubscription` 对象中添加 `anchorDay: subscription.anchorDay || anchorDay`

## 2. 手动续订路径修复

- [x] 2.1 在 `src/data/subscriptions.js` 的 `manualRenewSubscription` 中，将 anchorDay 回写到 `subscriptions[index]` 赋值对象中

## 3. 创建/编辑订阅续期循环中 anchorDay 来源修复

- [x] 3.1 修复 `createSubscription` 续期循环（第 61 行）：anchorDay 优先从 startDate 提取，而非已钳位的 expiryDate
- [x] 3.2 修复 `updateSubscription` 续期循环（第 157 行）：anchorDay 优先从已有的 subscription.anchorDay 或 startDate 提取
