# Phase 0: 项目发现

所有层和 fixing-routing 共享的初始化步骤。首次使用时执行一次，生成 `.routing-quality/config.md`，后续层直接读取。

---

## 触发条件

- `.routing-quality/config.md` 不存在
- 项目结构发生重大变化（新增/删除 skill、迁移目录）
- Baseline Intent 需要更新（新增 skill、用户对 skill 定位有新理解）

---

## 执行流程

### Step 1: 搜索项目结构

用文件搜索工具定位以下内容：

| 搜索目标 | 搜索方式 | 记录内容 |
|----------|---------|---------|
| Skill 目录 | glob `**/skills/*/SKILL.md` | 每个 skill 的路径、name、type、description 首行 |
| Planner prompt | glob `**/planner_prompt*.txt` | 文件路径 |
| Replan prompt | glob `**/replan_prompt*.txt` | 文件路径 |
| Tool schema | grep `skill_name` 或 `CreatePlanParams` | 定义 skill_name enum 的文件路径 |
| 测试文件 | glob `**/routing_test*.py` | 文件路径、test case 数量 |
| 测试结果 | glob `**/routing_test*_results.json` | 最新结果文件路径 |

### Step 2: 展示发现结果

向用户展示结构化摘要：

```
## 项目结构发现

### Skills ({N} 个)
| Skill | 路径 | Type | Description 首行 |
|-------|------|------|-----------------|
| ...   | ...  | ...  | ...             |

### 关键文件
- Planner prompt: {path}
- Replan prompt: {path}
- Tool schema: {path}
- 测试文件: {path} ({N} cases)
- 最新结果: {path} (准确率 {X}%)

### 当前状态
- 总 skill 数: {N}
- 最新测试准确率: {X}%
- 失败 case 数: {N}
```

### Step 3: 用户确认

等用户确认或修正以下内容：
- skill 列表是否完整
- 关键文件路径是否正确
- 是否有遗漏的配置或约束

### Step 3.5: 收集 Baseline Intent [串行 — 等用户]

对每个 skill，用 `references/templates.md` 的 Baseline Intent Spec 模板，请用户从产品/用户视角回答：

- 用户想做什么时应该走这个 skill？（一句话）
- 3-5 个最典型的输入场景
- 2-3 个绝对不该走这里的场景
- 已知的模糊区域和竞争 skill

**关键：不读 description，只从用户意图出发。**

将结果写入 `.routing-quality/config.md` 的新增 `Baseline Intent` section。

### Step 4: 写入 config

创建 `.routing-quality/config.md`：

```markdown
# Routing Quality Config

Generated: {date}

## Skills
| Name | Path | Type |
|------|------|------|
| ... | ... | ... |

## Files
- planner_prompt: {path}
- replan_prompt: {path}
- tool_schema: {path}
- test_file: {path}
- test_results: {path}

## Baseline
- accuracy: {X}%
- total_cases: {N}
- failed_cases: {N}

## Baseline Intent
[Phase 0 Step 3.5 收集的每个 skill 的意图定义]

## Competition Pairs
[Layer 3 完成后填充]
```

---

## 输出

- `.routing-quality/config.md` — 后续所有层和 fixing-routing 的共享输入
- 用户已确认的项目结构快照

---

## 注意事项

- Config 是快照，不是实时数据。项目结构变化后需重新运行 Phase 0
- 不修改任何项目文件，只读取和记录
- Competition Pairs 部分在 Phase 0 留空，由 Layer 3 填充
