# Description Rules

Skill description 修改的规则与反模式。所有涉及 description 的改动必须通过这些检查。

---

## 四规则

| # | 规则 | 检查点 |
|---|------|--------|
| 1 | **只改 description** | 改动是否仅限于 skill 的 description 字段和 body？是否涉及了 planner prompt？只有 F 类根因才需要改 planner prompt |
| 2 | **不交叉引用其他 skill 名称** | description 中是否出现了其他 skill 的名称（如"不属于 image-editing"）？应改为语义描述（如"不属于基于模板的创作"） |
| 3 | **不把 badcase 加到示例** | 是否把失败 case 原封不动加到了示例中？示例应展示正确执行流程，不是路由判断 |
| 4 | **泛化判断规则，不列关键词** | 判断维度/公式中是否列举了具体用户表达（如"做同款""拍同款"）？应改为语义概念（如"复刻整体效果"） |

---

## 反模式清单

### 1. 列举具体用户表达
- **错误**：当用户说"做同款"时 → 走本 skill
- **正确**：当用户意图为复刻整体效果时 → 走本 skill

### 2. 引用其他 skill 名称
- **错误**：不属于 image-editing 的场景
- **正确**：不属于基于模板的创作的场景

### 3. 用 badcase 当示例
- **错误**：示例中展示"用户说 X → 应该路由到本 skill"
- **正确**：示例展示 replan 阶段的正确执行流程

### 4. 改 planner prompt 来修路由
- **错误**：在 planner prompt 加"当用户说 X 时走 Y skill"
- **正确**：planner prompt 只管架构约束，路由决策由 skill description 驱动

### 5. 排除条件一刀切
- **错误**：单图生成视频 → 不属于本 skill
- **正确**：仅对原图添加动态效果时 → 不属于本 skill

### 6. 不触发条件过于激进
- **错误**：素材齐全 → 不触发
- **正确**：区分"模板创作（素材齐全可直接执行）"和"主体替换（即使有替换素材也需要走 assignment 流程）"

### 7. Schema default 与 prompt 矛盾
- **错误**：schema default="" + prompt "禁止留空"
- **正确**：删除 schema default，description 改为"必填，禁止留空"
- **原因**：LLM 优先听 schema（离 action 更近），schema 矛盾会覆盖 prompt 指令

---

## 职责分离原则

| 层级 | 职责 | 内容 |
|------|------|------|
| **Skill description** | 定义"我是什么" | 正向、自包含的适用场景和判断维度 |
| **Planner prompt** | 架构约束 + 关系裁决 | I2I 约束、澄清上限、竞争 skill 对的优先级规则 |
| **Tool schema** | 参数定义 | 必须与 prompt 对齐，不可矛盾 |

---

## 影响评估检查

修改 description 后，逐条检查：

- **扩展了覆盖范围** → 是否会误吸引原本不属于这个 skill 的 case？
- **缩窄了排除条件** → 是否会让原本正确排除的 case 被误收？
- **收紧了竞争 skill** → 是否会导致该 skill 原本正确处理的 case 失败？

列出可能受影响的 test case ID。
