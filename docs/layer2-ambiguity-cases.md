# Layer 2: Ambiguity Cases — 清理模糊地带

系统性识别和解决单个 skill 内的模糊输入。在 Layer 1 锁定核心场景后，处理"看起来像但不确定"的 case。

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

1. 读取 `.routing-quality/config.md`
2. 读取 `.routing-quality/snapshots/v1-golden/failed-cases.json`
3. 读取 `.routing-quality/snapshots/v1-golden/summary.md`
4. **不读 skill description** — description 在 Step 5 修复时才读

---

## 执行流程

### Step 1: 读取 config + v1 快照 [串行]

加载 config（含 Baseline Intent）、Layer 1 的失败 case 列表和改动摘要。**不读 description。**

### Step 2: 分析模糊输入区域 [AI 辅助 + 用户确认]

AI 从 Baseline Intent 的模糊区域 + Layer 1 失败模式推断歧义场景草稿。用户确认这些场景是否反映真实用户行为。

**AI 自动分析：** 读取 Baseline Intent "模糊区域" 列 + Layer 1 快照中的失败 case，用两个维度构建模糊矩阵：

**维度 A: 素材类型**
- 无素材 / 单图 / 多图 / 图+视频 / 纯视频

**维度 B: 意图类型**
- 创作 / 编辑 / 探索 / 反馈 / 澄清

**AI 生成草稿：**
1. **模糊矩阵热力图草稿** — 基于素材类型 x 意图类型交叉分析
2. **每个识别出的模糊区域 3-5 个候选 ambiguity case 草稿**
3. Layer 1 失败 cases 归类分析（是否属于模糊 case）

**用户验证：** 确认这些 case 是否代表真实用户会发送的模糊输入，可调整预期路由决策。

输出格式：
```json
{
  "case_id": "L2-{skill}-{N}",
  "skill": "{skill_name}",
  "media": "{素材描述}",
  "text": "{用户文本}",
  "expected": "{skill_name}",
  "ambiguity": "{为什么这个 case 模糊}",
  "competing_skill": "{可能竞争的 skill}"
}
```

### Step 3: 准备 ambiguity test cases [串行 — 等用户]

向用户展示：
- 模糊矩阵热力图（基于 Baseline Intent 的模糊区域 + 素材×意图交叉分析）
- 拟定的 ambiguity test cases（每个 skill 3-5 个）
- Layer 1 失败 cases 的归类结果

用户确认：
- 每个 case 的预期路由
- 是否有 case 其实不模糊（应移回 Layer 1 或移到 Layer 3）
- 是否有遗漏的模糊区域

### Step 4: 运行测试 [串行]

合并 Layer 1 golden cases + Layer 2 ambiguity cases 一起跑，确保无回归。

### Step 5: 分析结果 [串行]

```
├─ < 80%  → description 边界定义有系统性问题
│          修复 description → 回到 Step 2
│
├─ 80-95% → 个别模糊区域需要加强
│           修复 description → 回到 Step 4
│
└─ >= 95% → 模糊层达标
            问用户：继续提升 or 进入下一层？
```

**先确认测试失败（RED），再读 description 定位根因并修复（GREEN）：**

**最小改动原则：** 只改让失败 case 通过所需的最少内容。不顺手重写、不扩展未被测试覆盖的区域、不"改善"无关表述。
- 用 `references/root-cause-taxonomy.md` 归类根因（A/B 类最常见）
- 遵守 `references/description-rules.md` 四规则
- 用 `references/templates.md` 的 Spec 模板记录改动
- 扩展覆盖时检查是否侵入其他 skill 的领地

### Step 5.5: REFACTOR — 精简 description [串行]

测试全绿后，回顾修复过程中对 description 的所有改动：

1. **去冗余** — 多轮修复是否引入了重复或矛盾的表述？合并
2. **查一致** — 跨 skill 的排除条款是否对称？
3. **控长度** — 在不丢失路由信号的前提下精简

**约束：REFACTOR 期间测试必须保持全绿。**

### Step 6: 保存快照 v2 [串行]

保存到 `.routing-quality/snapshots/v2-ambiguity/`：

```
v2-ambiguity/
  skills/              # 所有 skill description 完整副本
  test-cases.json      # Layer 2 ambiguity cases
  test-results.json    # 测试结果（含 L1 回归验证）
  failed-cases.json    # 失败 case 列表
  summary.md           # 改动摘要
```

### Step 7: 输出失败 cases [串行]

将 Layer 1 + Layer 2 的所有失败 case 合并整理，传递给 Layer 3。标注每个 case 是"单 skill 模糊"还是"跨 skill 竞争"。

---

## Todo 模板

进入 Layer 2 时创建：

- [ ] 读取 config（含 Baseline Intent）+ v1 快照，不读 description
- [ ] 分析每个 skill 的模糊输入区域
- [ ] 向用户确认 ambiguity case 列表
- [ ] 运行测试（含 L1 回归验证）
- [ ] 分析失败 case + 最小改动修复 description（迭代至全绿）
- [ ] REFACTOR：精简 description，保持全绿
- [ ] 保存快照 v2
- [ ] 输出失败 case 列表给 Layer 3

---

## 退出条件

- Ambiguity cases 准确率 >= 95%（含 L1 回归）
- 快照 v2 已保存
- 失败 case 已标注"单 skill 模糊" vs "跨 skill 竞争"
