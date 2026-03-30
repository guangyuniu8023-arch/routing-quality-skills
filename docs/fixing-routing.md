# Fixing Routing — 单个路由 Bug 的快速修复

当发现单个路由问题时（截图、日志、测试失败、用户反馈），按以下流程排查和修复。每个 Phase 结束后向用户汇报结论，等确认后再进入下一个 Phase。

---

## 输入

1. 读取 `.routing-quality/config.md`（如不存在，先执行 `phase0-discovery.md`）
2. 问题来源：截图 / 测试失败 / 用户反馈

---

## Phase 1: 收集证据

**目标：搞清楚"送了什么进去、出了什么结果、预期是什么"**

### 1.1 根据来源收集

| 来源 | 收集方式 |
|------|---------|
| **截图** | 读截图 → 找时间戳 → 在请求日志中按时间找对应的 plan 和 complete 记录 |
| **测试失败** | 从 test results 中取 failed_cases 详情 |
| **用户口述** | 确认：素材类型、用户文本、预期路由、实际路由 |

### 1.2 整理证据卡片 [AI 辅助]

AI 解析用户提供的日志/截图/测试结果，自动生成证据卡草稿（素材类型、用户文本、预期路由、实际路由）。用户确认或修正。

用 `references/templates.md` 的证据卡片格式，对每个问题 case 标准化记录。

### 1.3 汇报

向用户展示证据卡片，确认预期路由是否正确。

---

## Phase 2: 定位根因

**目标：找到是哪个 description 的哪一句话导致了误路由**

### 2.1 读取相关文件

从 config 获取路径，读取：
- Planner prompt
- 误路由目标 skill 的 description — 为什么它接住了？
- 正确目标 skill 的 description — 为什么它没接住？
- Tool schema — 检查是否有 schema-prompt 矛盾

### 2.2 逐条比对

对每个失败 case，回答三个问题：
1. 正确 skill 的 description 是否覆盖了这个 case？
2. 错误 skill 的 description 为什么吸引了 planner？
3. Planner prompt 是否有干扰？

### 2.3 分类根因

用 `references/root-cause-taxonomy.md` 的 A-F 分类归类每个 case。

### 2.4 合并

多个失败 case 共享同一根因 → 合并为系统性问题。如果发现系统性问题，建议用户跑 building 流程（Layer 1-4）。

### 2.5 汇报

向用户展示根因分析。

---

## Phase 3: 写 Spec

**目标：改之前先让用户审核方案**

### 3.1 撰写修改方案 [AI 辅助 + 用户审阅]

AI 根据根因分析自动生成修复 Spec 草稿（问题摘要、修改前后对比、不变项、测试计划）。用户审阅后确认或调整。

用 `references/templates.md` 的 Spec 模板，内容包括：
- 问题总结（证据卡片 + 根因分类）
- 修改方案（改前/改后/对应根因）
- 不改的部分
- 测试计划

### 3.2 四规则自检

用 `references/description-rules.md` 的四规则逐条检查修改方案。

### 3.3 影响评估

用 `references/description-rules.md` 的影响评估检查，列出可能受影响的 case。

### 3.4 汇报

向用户展示 Spec，等审核通过。

---

## Phase 4: 执行改动

**目标：按 Spec 改，不多不少**

1. 应用 description 改动，逐个 diff 确认
2. 补充测试用例（新 case ID 从现有最大值 +1）
3. 如涉及 planner prompt / schema（仅 E/F 类根因），同样 diff 确认

---

## Phase 5: 跑测试 + 回归验证

**目标：用数据证明改动是正向的**

### 5.1 运行测试（建议 3 runs）

### 5.2 对比结果

用 `references/templates.md` 的达标门槛判定：
- 总准确率不低于改动前
- 目标失败 case 改善
- 无新增完全失败 case

### 5.3 未达标处理

- Description 改动导致 → 回 Phase 3 调整 Spec
- LLM 噪声（3 次中仅 1 次失败）→ 记录但不阻塞

---

## Phase 6: 记录变更

**目标：让未来能回溯这次改动的上下文和决策**

用 `references/templates.md` 的变更记录模板，记录：
- 变更概览
- 修改的文件和内容
- 测试结果（改前 → 改后）
- 剩余失败 case 分析

---

## 与 Building 的关系

- Fixing 修完一个 bug 后，如果发现是系统性问题（多个 case 共享根因）→ 建议跑 Building（Layer 1-4）
- Building 的三层方法论覆盖了 fixing 使用的根因分类法和四规则
- 两者共享 `references/` 下的参考文件和 `.routing-quality/config.md`
