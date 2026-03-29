# Root Cause Taxonomy

路由失败的根因分类。用于定位问题后归类，指导修复方向。

---

## 分类表

| 类型 | 含义 | 修复方向 |
|------|------|---------|
| **A. 覆盖不全** | 正确 skill 的适用场景/判断维度没覆盖这个 case | 扩展正确 skill 的 description |
| **B. 排除过宽** | 正确 skill 的"不适用/不触发"条件误排了这个 case | 缩窄正确 skill 的排除条件 |
| **C. 竞争吸引** | 错误 skill 的某个条款（如豁免、兜底）过宽，吸引了本不属于它的 case | 收紧错误 skill 的 description |
| **D. 优先级不明** | 两个 skill 都能匹配，缺少优先级信号 | 在正确 skill 加高优先级标记，或在错误 skill 加排除条件 |
| **E. Schema-Prompt 矛盾** | tool schema 的 description/default 与 prompt 冲突，LLM 优先听 schema | 对齐 schema 与 prompt |
| **F. Planner 干扰** | planner prompt 的架构约束或示例误导了路由决策 | 修改 planner prompt |

---

## 使用方法

### 1. 逐条比对

对每个失败 case，回答三个问题：

1. **正确 skill 的 description 是否覆盖了这个 case？**
   - 适用场景是否包含？
   - 判断维度/公式能否匹配？
   - 是否被"不适用"或"不触发"条件误排除？

2. **错误 skill 的 description 为什么吸引了 planner？**
   - 哪个条款让 planner 觉得它更匹配？
   - 是否存在过宽的豁免条款？

3. **planner prompt 是否有干扰？**
   - 架构约束是否误触发？
   - Few-shot 示例是否有误导？

### 2. 归类

根据上述分析，将每个 case 归入 A-F 中的一类。

### 3. 合并

多个失败 case 是否共享同一个根因？如果是，合并为一个系统性问题，统一修复。

---

## 修复优先级

- **A/B/C**（description 问题）：最常见，修 description 即可，风险低
- **D**（优先级）：需要在竞争 skill 对之间建立裁决规则
- **E**（schema）：必须修，schema 矛盾会持续误导 LLM
- **F**（planner）：谨慎修改，planner prompt 影响所有 skill 的路由
