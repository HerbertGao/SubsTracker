## 为什么

当订阅周期为"月"且起始日为月末日期（如 1 月 31 日）时，JavaScript `Date.setMonth()` 会导致日期漂移：1 月 31 日 + 1 月 = 3 月 3 日（而非 2 月 28 日），后续周期也会持续偏离。用户期望的行为是"固定日扣款"——始终在原始日期对应的每月同一天（或该月最后一天）到期。例如 1/31 → 2/28 → 3/31 → 4/30。当前代码中有 6 处公历月周期计算均使用 `setMonth()`，全部受此问题影响。

## 变更内容

- **新增**：统一的公历月/年周期计算函数，正确处理月末日期钳位（clamp to last day of month）
- **修改**：替换所有 6 处 `setMonth()` / `setFullYear()` 调用为新的统一函数：
  - 前端到期日计算 (`calculateExpiryDate`)
  - 前端续费预览 (`updateNewExpiryPreview`)
  - 创建订阅时的过期推算 (`createSubscription`)
  - 更新订阅时的过期推算 (`updateSubscription`)
  - 定时任务自动续费 (`handleScheduledTask` 中的 `addOnePeriod`)
  - 手动续订 API (`renewSubscription`)
- **保留**：农历周期计算（`addLunarPeriod`）不受影响，其已正确处理月末日期

## 功能 (Capabilities)

### 新增功能
- `month-end-date-clamp`: 公历月/年周期计算的月末日期钳位逻辑——当目标月份天数不足时，自动钳位到该月最后一天，同时记住原始订阅日以便后续月份恢复到正确日期

### 修改功能
（无现有规范需要修改）

## 影响

- **代码**：`index.js` 中 6 处公历日期计算逻辑（前端 2 处 + 后端 4 处）
- **数据模型**：可能需要在订阅对象中新增 `anchorDay` 字段，记录原始订阅日（如 31），以便在日期钳位后的下一个月恢复正确日期（如 2/28 之后的下一个月应为 3/31 而非 3/28）
- **API**：无接口变更，行为修正属于 bug fix
- **兼容性**：对现有数据向后兼容——未设置 `anchorDay` 的订阅按当前到期日的 day 作为锚定日