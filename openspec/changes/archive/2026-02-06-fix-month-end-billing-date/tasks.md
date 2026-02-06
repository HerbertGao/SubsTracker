## 1. 核心函数

- [x] 1.1 在 index.js 顶部工具函数区域新增 `addGregorianPeriod(date, periodValue, periodUnit, anchorDay)` 函数，实现月/年周期计算并钳位到目标月最后一天，同时利用 anchorDay 在后续月份恢复正确日期；day 类型直接 `setDate` 透传
- [x] 1.2 在前端模板字符串（adminPage）中新增同名 `addGregorianPeriod` 函数副本，逻辑与后端保持一致，添加注释标注"此函数需前后端同步"

## 2. 数据模型

- [x] 2.1 在 `createSubscription`（~L6064）中，从 startDate 或 expiryDate 的 day 提取 anchorDay 并存入订阅对象
- [x] 2.2 在 `updateSubscription` 中，当用户修改到期日时更新 anchorDay
- [x] 2.3 在所有读取订阅的逻辑中，对缺少 anchorDay 的历史数据，兜底取 expiryDate 的 day

## 3. 替换前端日期计算

- [x] 3.1 替换 `calculateExpiryDate`（~L3547）中的 `setMonth()` / `setFullYear()` 为 `addGregorianPeriod` 调用
- [x] 3.2 替换 `updateNewExpiryPreview`（~L2476）中的 `setMonth()` / `setFullYear()` 为 `addGregorianPeriod` 调用

## 4. 替换后端日期计算

- [x] 4.1 替换 `createSubscription`（~L6052）中过期推算循环里的 `setMonth()` / `setFullYear()` 为 `addGregorianPeriod` 调用
- [x] 4.2 替换 `updateSubscription`（~L6152）中过期推算循环里的 `setMonth()` / `setFullYear()` 为 `addGregorianPeriod` 调用
- [x] 4.3 替换 `renewSubscription`（~L6304）中的 `setMonth()` / `setFullYear()` 为 `addGregorianPeriod` 调用
- [x] 4.4 替换 `handleScheduledTask` 中 `addOnePeriod`（~L7172）的 `setMonth()` / `setFullYear()` 为 `addGregorianPeriod` 调用

## 5. 验证

- [ ] 5.1 手动测试：创建月订阅（起始日 1/31），验证到期日序列为 2/28(或29) → 3/31 → 4/30（需部署后验证）
- [ ] 5.2 手动测试：创建年订阅（起始日 2/29 闰年），验证到期日在非闰年为 2/28（需部署后验证）
- [ ] 5.3 手动测试：对无 anchorDay 的历史订阅进行续费，验证行为正常（需部署后验证）
