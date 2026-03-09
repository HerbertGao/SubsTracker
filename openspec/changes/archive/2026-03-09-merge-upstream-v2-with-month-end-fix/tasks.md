## 1. 合并上游代码与冲突解决

- [x] 1.1 执行 `git fetch upstream` 并 `git merge upstream/master --no-commit` 触发合并
- [x] 1.2 解决 `wrangler.toml` 冲突：保留 KV binding-only 配置，接受 `main: "src/index.js"` 和每小时 cron
- [x] 1.3 解决 `README.md` 冲突：接受上游新文档结构，保留一键部署按钮（指向 fork）和 Git 集成部署章节
- [x] 1.4 解决 `index.js` modify/delete 冲突：接受上游删除，`git rm index.js`

## 2. 移植 addGregorianPeriod 到模块化架构

- [x] 2.1 在 `src/core/time.js` 中添加 `addGregorianPeriod` 函数定义并加入 export 列表
- [x] 2.2 在 `src/data/subscriptions.js` 中导入 `addGregorianPeriod`，替换 `createSubscription` 中的内联周期计算
- [x] 2.3 在 `src/data/subscriptions.js` 中替换 `updateSubscription` 中的内联周期计算
- [x] 2.4 在 `src/data/subscriptions.js` 中替换 `manualRenewSubscription` 中的内联周期计算
- [x] 2.5 在 `src/services/scheduler.js` 中导入 `addGregorianPeriod`，替换 `checkExpiringSubscriptions` 中的内联周期计算

## 3. 移植 anchorDay 字段支持

- [x] 3.1 在 `createSubscription` 中从起始日/到期日提取 anchorDay，添加到新建订阅对象
- [x] 3.2 在 `updateSubscription` 中从编辑后到期日更新 anchorDay，添加到更新后的订阅对象
- [x] 3.3 在 `manualRenewSubscription` 中使用 `subscription.anchorDay || newStartDate.getDate()` 回退
- [x] 3.4 在 `checkExpiringSubscriptions` 中使用 `subscription.anchorDay || expiryDate.getDate()` 回退

## 4. 前端代码移植

- [x] 4.1 在 `src/views/adminPage.html` 中添加 `addGregorianPeriod` 函数副本及同步注释
- [x] 4.2 替换 `updateNewExpiryPreview` 中的内联周期计算为 `addGregorianPeriod` 调用
- [x] 4.3 替换表单提交（公历推算）中的内联周期计算为 `addGregorianPeriod` 调用

## 5. 验证与提交

- [ ] 5.1 暂存所有已修改文件并提交合并 commit
