# Layer 1: Golden Cases — 锁定核心场景

每个 skill 的"主场"case 100% 正确。这是路由质量的基础层。

---

## Iron Law

```
不读 description 就不改 description。不跑测试就不知道该改什么。
```

- Step 1-4：只读 config + Baseline Intent，不读 SKILL.md
- Step 5：只有测试失败才读 description，只为让失败 case 通过而改
- 违反顺序？回到 Step 1 重新开始

---

## 输入

1. 读取 `.routing-quality/config.md`（含 Baseline Intent）
2. **不读 skill description** — description 在 Step 5 修复时才读

---

## 执行流程

### Step 1: 读取 config（含 Baseline Intent）[串行]

从 config 获取 skill 列表和 Baseline Intent。**不读 description。**

### Step 2: 从 Baseline Intent 生成 golden cases [AI 辅助 + 用户确认]

AI 从 Baseline Intent 的必须通过场景自动生成 golden case 草稿，用户逐个确认是否符合真实用户行为。

每个 skill 3-5 个 golden case。**来源是 config 中的 Baseline Intent，不是 description（Iron Law）：**

1. **AI 自动生成草稿** — 读取 Baseline Intent 的"必须通过的场景"，为每个 skill 生成 3-5 个 golden case 草稿（含 case_id, media, text, expected_skill, reason）
2. **AI 从"绝对不该走这里的场景"生成反向 case 草稿**（预期路由为其他 skill）
3. **用户验证** — 逐个确认每个 case 是否代表真实用户行为，可增删编辑
4. **AI 不读 SKILL.md description 来生成 cases（Iron Law）**

Golden case 标准：
- 来自用户/产品视角，不依赖 description 的措辞
- 代表该 skill 最典型的使用场景
- 只有一个 skill 能合理匹配（无歧义）

输出格式：
```json
{
  "case_id": "L1-{skill}-{N}",
  "skill": "{skill_name}",
  "media": "{素材描述}",
  "text": "{用户文本}",
  "expected": "{skill_name}",
  "reason": "{为什么这是 golden case}"
}
```

### Step 3: 向用户展示 golden case 列表 [串行 — 等用户]

按 skill 分组展示，用户确认：
- 每个 case 的预期路由是否正确
- 是否有遗漏的核心场景
- 是否有 case 其实有歧义（应移到 Layer 2）

### Step 4: 运行测试 [串行]

将确认后的 golden cases 写入测试文件，运行测试（建议 3 runs 取多数）。

### Step 5: 读 description + 分析失败 + 修复 [串行]

**现在才读 description。**

**最小改动原则：** 只改让失败 case 通过所需的最少内容。不顺手重写、不扩展未被测试覆盖的区域、不"改善"无关表述。每个改动必须对应一个失败 case。

对每个失败 case：

1. 读正确 skill 和错误 skill 的 description
2. 用 `references/root-cause-taxonomy.md` 归类根因
3. 用 `references/description-rules.md` 四规则约束修复方案
4. 用 `references/templates.md` 的 Spec 模板记录改动
5. 修复 description → 回到 Step 4 重新测试

```
├─ < 80%  → description 有系统性问题，继续修复（不回 Step 2，cases 来自用户意图）
├─ 80-95% → 个别 description 覆盖不全，修复后回 Step 4
└─ >= 95% → 基础层达标，问用户：继续提升 or 进入下一层？
```

### Step 5.5: REFACTOR — 精简 description [串行]

测试全绿后，回顾修复过程中对 description 的所有改动：

1. **去冗余** — 多轮修复是否引入了重复或矛盾的表述？合并
2. **查一致** — 跨 skill 的排除条款是否对称？（A 排除的 = B 包含的）
3. **控长度** — description 是否因修复变得过长？在不丢失路由信号的前提下精简

**约束：REFACTOR 期间测试必须保持全绿。** 每次精简后重新跑测试，失败则回退。

### Step 6: 保存快照 v1 [串行]

保存到 `.routing-quality/snapshots/v1-golden/`：

```
v1-golden/
  skills/              # 所有 skill description 完整副本
  test-cases.json      # golden test cases
  test-results.json    # 测试结果
  failed-cases.json    # 失败 case 列表
  summary.md           # 改动摘要（用 templates.md 的格式）
```

### Step 7: 输出失败 cases [串行]

将 `failed-cases.json` 中的失败 case 整理为证据卡片格式（见 `references/templates.md`），传递给 Layer 2 分析。

---

## Todo 模板

进入 Layer 1 时创建：

- [ ] 读取项目 config（含 Baseline Intent），不读 description
- [ ] 从 Baseline Intent 定义 golden cases（3-5 个/skill）
- [ ] 向用户确认 golden case 列表
- [ ] 运行基线测试
- [ ] 分析失败 case + 最小改动修复 description（迭代至全绿）
- [ ] REFACTOR：精简 description，保持全绿
- [ ] 保存快照 v1
- [ ] 输出失败 case 列表给 Layer 2

---

## 退出条件

- Golden cases 准确率 >= 95%
- 快照 v1 已保存
- 失败 case 列表已整理
