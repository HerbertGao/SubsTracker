## 为什么

当订阅的到期日为月末（如 1 月 31 日）时，第一次自动续费能正确钳位到 2 月 28 日，但第二次续费会错误地设为 3 月 28 日而非 3 月 31 日。根因是自动续费和手动续订路径在计算完 anchorDay 后没有将其回写到订阅对象中，导致下一次续费时 fallback `expiryDate.getDate()` 取到了被钳位后的日期（28），而非用户期望的锚定日（31）。

## 变更内容

- 修复 `src/services/scheduler.js` 自动续费路径：将计算出的 anchorDay 回写到 `updatedSubscription` 对象中
- 修复 `src/data/subscriptions.js` 手动续订路径（`manualRenewSubscription`）：将计算出的 anchorDay 回写到更新后的订阅对象中

## 功能 (Capabilities)

### 新增功能

（无）

### 修改功能

- `month-end-date-clamp`: 补充 anchorDay 在续费路径中的持久化要求，确保自动续费和手动续订后 anchorDay 不丢失

## 影响

- **后端模块**：`src/services/scheduler.js`（自动续费 updatedSubscription 对象）、`src/data/subscriptions.js`（手动续订 subscriptions[index] 赋值）
- **数据**：老订阅数据（无 anchorDay 字段）在首次续费后会被自动补上 anchorDay，后续续费不再丢失
