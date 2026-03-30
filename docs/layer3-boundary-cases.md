# Layer 3: Boundary Cases — 裁决边界 + 定义兜底

对竞争 skill 对建立裁决规则。不可裁决的定义可接受兜底，确保"错得可控"。

---

## Iron Law

```
不读 description 就不改 description。不跑测试就不知道该改什么。
```

- Step 1-5：只读 config + Baseline Intent + 快照，不读 SKILL.md
- Step 6：只有测试失败才读 description，只为让失败 case 通过而改
- 违反顺序？回到 Step 1 重新开始

---

## 输入

1. 读取 `.routing-quality/config.md`
2. 读取 `.routing-quality/snapshots/v1-golden/failed-cases.json`
3. 读取 `.routing-quality/snapshots/v2-ambiguity/failed-cases.json`
4. 读取 `.routing-quality/snapshots/v2-ambiguity/summary.md`
5. **不读 skill description** — description 在 Step 6 修复时才读

---

## 执行流程

### Step 1: 读取 config + v1/v2 快照 [串行]

加载 config（含 Baseline Intent）、Layer 1+2 的所有失败 case。**不读 description。**

### Step 2: 识别竞争 skill 对 [AI 辅助 + 用户确认]

AI 从 config 的竞争skill 列自动提取竞争对，生成判断信号分析和 fallback 建议。用户审阅后确认或调整。

**AI 自动提取竞争对：** 从 config.md 的"竞争skill"列 + Layer 1+2 失败 case 中识别竞争关系，呈现列表供用户确认。

1. **Baseline Intent 声明的竞争对** — AI 从 config 自动提取，用户确认
2. **失败 case 中发现的新竞争对** — AI 从 Layer 1+2 失败 case 中发现 Baseline Intent 未预见的竞争
3. **AI 生成判决信号表草稿** — 每对竞争 skill，AI 分析区分它们的关键信号，用户验证
4. **标注不可裁决** — 哪些 case 两个 skill 都有合理性，无法通过 description 区分？

输出格式：
```markdown
## 竞争对: {skill-A} vs {skill-B}

误路由频率: {N} 次（L1: {x}, L2: {y}）

### 判决信号
| 信号 | → skill-A | → skill-B |
|------|-----------|-----------|
| ... | ... | ... |

### 不可裁决 cases
| case_id | 素材 | 文本 | 为什么不可裁决 |
|---------|------|------|---------------|
```

### Step 3: 验证并完善兜底策略 [AI 建议 + 用户审批]

**AI 生成 fallback 建议：** 基于竞争对分析，AI 为每对竞争 skill 生成兜底方向建议，附带 UX 影响推理：

- **AI 分析哪个错误结果对用户体验损失更小**
  - 例：误路由到 selection-clarification（多问一次）vs 误路由到 image-editing（直接执行错误操作）→ 前者损失更小
- **AI 建议兜底方向** — 不可裁决时默认路由到哪个 skill，附推理
- **目标** — 95% 的"错误"结果是用户体验损失可控的

**用户审批或覆盖：** 用户审阅 AI 的 fallback 建议，确认或调整兜底方向。

### Step 4: 准备 boundary test cases [AI 辅助 + 用户确认]

**AI 生成 boundary case 草稿：** 为每对竞争 skill 生成 3-5 个候选 boundary case，来源为判决信号分析：
- 覆盖已识别的判决信号
- 包含不可裁决 case（预期为兜底方向）
- 包含 Layer 1+2 中标注为"跨 skill 竞争"的失败 case

**用户验证：** 确认每个 case 是否反映真实边界场景，可增删编辑。

### Step 5: 运行测试 [串行]

合并 L1 golden + L2 ambiguity + L3 boundary 一起跑，确保无回归。

### Step 6: 分析结果 + 迭代 [串行]

```
├─ 可裁决 case 失败 → 修复 description，加强判决信号
├─ 不可裁决 case 未走兜底方向 → 调整兜底 skill 的 description 吸引力
└─ 回归 → 定位是哪层改动导致，回到对应层修复
```

**最小改动原则：** 只改让失败 case 通过所需的最少内容。不顺手重写、不扩展未被测试覆盖的区域。

修复时：
- 用 `references/root-cause-taxonomy.md` 归类根因（C/D 类最常见）
- 遵守 `references/description-rules.md` 四规则
- 用 `references/templates.md` 的 Spec 模板记录改动

### Step 6.5: REFACTOR — 精简 description [串行]

测试全绿后，回顾修复过程中对 description 的所有改动：

1. **去冗余** — 多轮修复是否引入了重复或矛盾的表述？合并
2. **查一致** — 跨 skill 的排除条款是否对称？
3. **控长度** — 在不丢失路由信号的前提下精简

**约束：REFACTOR 期间测试必须保持全绿。**

### Step 7: 保存快照 v3 [串行]

保存到 `.routing-quality/snapshots/v3-boundary/`：

```
v3-boundary/
  skills/              # 所有 skill description 完整副本
  test-cases.json      # Layer 3 boundary cases
  test-results.json    # 测试结果（含 L1+L2 回归验证）
  failed-cases.json    # 失败 case 列表
  summary.md           # 改动摘要
```

### Step 8: 更新竞争对裁决表 [串行]

将确认的竞争对信息写入 `.routing-quality/config.md` 的 Competition Pairs 部分：

```markdown
## Competition Pairs

| Skill A | Skill B | 判决信号 | 兜底方向 |
|---------|---------|---------|---------|
| ... | ... | ... | ... |
```

---

## Todo 模板

进入 Layer 3 时创建：

- [ ] 读取 config（含 Baseline Intent）+ v1/v2 快照，不读 description
- [ ] 从 Baseline Intent + 失败 case 识别竞争 skill 对
- [ ] 验证并完善兜底策略
- [ ] 准备 boundary test cases
- [ ] 运行测试（含 L1+L2 回归验证）
- [ ] 分析结果 + 迭代
- [ ] REFACTOR：精简 description，保持全绿
- [ ] 保存快照 v3
- [ ] 更新竞争对裁决表到 config

---

## 退出条件

- 可裁决 boundary cases 准确率 >= 95%
- 不可裁决 cases 兜底方向正确率 >= 90%
- L1+L2 无回归
- 竞争对裁决表已写入 config
- 快照 v3 已保存
