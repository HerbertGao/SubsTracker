## 上下文

当前系统使用 JavaScript 原生 `Date.setMonth()` 进行公历月周期计算。`setMonth()` 在月末日期上存在已知缺陷：

```
new Date("2024-01-31").setMonth(1)  → 2024-03-02（溢出到 3 月）
```

代码中共有 6 处受影响：
| 位置 | 函数 | 类型 |
|------|------|------|
| ~L3547 | `calculateExpiryDate()` | 前端：创建/编辑时计算到期日 |
| ~L2476 | `updateNewExpiryPreview()` | 前端：续费预览 |
| ~L6052 | `createSubscription()` | 后端：创建时过期推算 |
| ~L6152 | `updateSubscription()` | 后端：更新时过期推算 |
| ~L6304 | `renewSubscription()` | 后端：手动续订 |
| ~L7172 | `addOnePeriod()` in `handleScheduledTask` | 后端：自动续费 |

农历路径（`lunarBiz.addLunarPeriod`）已正确处理此问题（通过 `Math.min(day, maxDay)` 钳位），无需修改。

## 目标 / 非目标

**目标：**
- 引入统一的 `addGregorianPeriod(date, periodValue, periodUnit, anchorDay)` 函数
- 替换所有 6 处内联日期计算为统一函数调用
- 使用 `anchorDay`（锚定日）机制：记住用户原始订阅日，每次计算时尝试恢复到该日，若目标月份天数不足则钳位到月末
- 对现有数据向后兼容

**非目标：**
- 不修改农历周期计算逻辑
- 不改变订阅模式（cycle / reset）的语义
- 不修改 UI 布局或新增用户可见配置项

## 决策

### D1：使用 anchorDay 而非简单钳位

**选择**：在订阅对象中存储 `anchorDay` 字段

**理由**：简单钳位（如 `Math.min(day, lastDay)`）在连续计算时会丢失信息。例如 1/31 → 2/28 后，如果基于 2/28 再加 1 月，得到 3/28 而非期望的 3/31。`anchorDay` 保留了"用户期望每月 31 日到期"这一意图。

**替代方案**：
- 仅钳位不存储：连续续费时日期逐步漂移（1/31 → 2/28 → 3/28 → 4/28），不符合用户预期
- 每次从原始起始日重新计算：需要追踪已过去的周期数，且与 reset 模式冲突

### D2：anchorDay 的来源

**选择**：
- 新创建的订阅：从 `startDate` 或 `expiryDate` 的 day 提取
- 现有订阅（无 anchorDay 字段）：从当前 `expiryDate` 的 day 提取作为默认值
- 用户编辑到期日时：自动更新 anchorDay

**理由**：向后兼容，无需数据迁移。

### D3：函数位置

**选择**：在 `index.js` 顶部工具函数区域定义 `addGregorianPeriod`，前端模板字符串中定义同名函数副本

**理由**：当前架构为单文件，前后端代码在同一文件但前端代码在模板字符串内，无法直接共享函数。需要维护两份相同逻辑的副本（与 `lunarBiz` 等工具函数的现有模式一致）。

**替代方案**：
- 仅在后端定义，前端通过 API 调用：增加不必要的网络请求，影响交互体验
- 使用注入变量：当前无此机制，改造成本过高

### D4：年周期也使用 anchorDay

**选择**：`setFullYear()` 同样存在 2/29 → 非闰年 3/1 的问题，一并使用 `addGregorianPeriod` 处理

**理由**：保持一致性，避免遗漏边界情况。

## 风险 / 权衡

| 风险 | 缓解措施 |
|------|----------|
| 前后端两份函数副本可能不同步 | 函数逻辑简单（<15 行），添加注释标注"此函数需前后端同步" |
| 现有订阅无 anchorDay 字段 | 兜底逻辑：无 anchorDay 时从 expiryDate 提取 day，行为与修复前基本一致 |
| 自动续费循环补齐多个周期时 anchorDay 行为 | 每次 addOnePeriod 都传入 anchorDay，钳位后下个周期仍可恢复 |
