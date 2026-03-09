## ADDED Requirements

### 需求：模块化代码架构
系统必须采用 `src/` 目录下的模块化架构组织代码，禁止使用根目录单文件 `index.js` 作为入口。

#### 场景：入口文件位于 src 目录
- **当** 检查 `wrangler.toml` 的 `main` 配置
- **那么** 入口必须为 `src/index.js`

#### 场景：核心工具函数在 core 模块中
- **当** 需要使用时间计算、农历转换等基础工具
- **那么** 必须从 `src/core/` 下的对应模块导入

### 需求：addGregorianPeriod 函数在 time 模块中导出
`src/core/time.js` 必须导出 `addGregorianPeriod` 函数，供后端所有公历周期计算场景使用。

#### 场景：后端模块导入 addGregorianPeriod
- **当** `src/data/subscriptions.js` 或 `src/services/scheduler.js` 需要计算公历月/年周期
- **那么** 必须从 `src/core/time.js` 导入 `addGregorianPeriod` 函数

### 需求：前端 adminPage 包含 addGregorianPeriod 副本
`src/views/adminPage.html` 的 JavaScript 代码中必须包含与后端功能一致的 `addGregorianPeriod` 函数副本，且通过注释标记需与后端同步。

#### 场景：前端日期计算使用 addGregorianPeriod
- **当** 用户在前端创建/编辑订阅表单中选择起始日期和周期
- **那么** 前端必须使用本地定义的 `addGregorianPeriod` 函数计算到期日

#### 场景：前端函数副本包含同步提示注释
- **当** 检查 `src/views/adminPage.html` 中的 `addGregorianPeriod` 函数定义
- **那么** 函数上方必须包含注释说明此函数需与后端 `src/core/time.js` 同步
