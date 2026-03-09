## MODIFIED Requirements

### 需求：所有公历日期计算入口统一
系统中所有涉及公历月/年周期计算的代码路径，必须使用统一的日期计算函数，禁止直接使用 `Date.setMonth()` 或 `Date.setFullYear()` 进行周期推算。

#### 场景：前端到期日计算
- **当** 用户在创建/编辑订阅表单中选择起始日期和周期
- **那么** 系统必须使用 `src/views/adminPage.html` 中定义的 `addGregorianPeriod` 函数计算到期日

#### 场景：后端自动续费
- **当** 定时任务检测到订阅已过期且开启自动续费
- **那么** 系统必须使用从 `src/core/time.js` 导入的 `addGregorianPeriod` 函数计算新的到期日

#### 场景：手动续订
- **当** 用户通过 API 手动续订订阅
- **那么** 系统必须使用从 `src/core/time.js` 导入的 `addGregorianPeriod` 函数计算新的到期日

#### 场景：创建订阅时自动续期
- **当** 用户创建订阅且到期日早于当前时间
- **那么** 系统必须使用从 `src/core/time.js` 导入的 `addGregorianPeriod` 函数循环推算至未来日期

#### 场景：编辑订阅时自动续期
- **当** 用户编辑订阅且到期日早于当前时间
- **那么** 系统必须使用从 `src/core/time.js` 导入的 `addGregorianPeriod` 函数循环推算至未来日期
