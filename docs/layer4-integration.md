# Layer 4: Integration — 全量回归验证

所有层的 case 合在一起跑，确保无回归。这是发布前的最终验证。

---

## 输入

1. 读取 `.routing-quality/config.md`
2. 读取 `.routing-quality/snapshots/v1-golden/test-cases.json`
3. 读取 `.routing-quality/snapshots/v2-ambiguity/test-cases.json`
4. 读取 `.routing-quality/snapshots/v3-boundary/test-cases.json`
5. 读取所有 skill 的当前 description

---

## 执行流程

### Step 1: 合并全部 test cases [串行]

将 L1 golden + L2 ambiguity + L3 boundary 的 test cases 合并为一个完整测试集。

检查：
- **去重** — 是否有跨层重复的 case？合并时保留更高层的版本
- **总数上限** — 验证总 case 数 <= 400
- **覆盖度** — 每个 skill 至少有 golden + ambiguity + boundary 三类 case

### Step 2: 运行全量测试 [串行]

用合并后的完整测试集运行测试（建议 3 runs 取多数）。

### Step 3: 分析结果 [串行]

#### 3.1 整体指标

| 指标 | 值 | 判定 |
|------|-----|------|
| 总准确率 | X% | >= 95% |
| Golden cases 准确率 | X% | >= 98% |
| Ambiguity cases 准确率 | X% | >= 90% |
| Boundary cases 准确率 | X% | >= 90% |

#### 3.2 回归检测

对比每层快照，识别回归：

```
对 v1 快照中通过的 case:
  现在仍通过 → ✅
  现在失败   → ⚠️ 回归 — 归因到哪层改动导致

对 v2 快照中通过的 case:
  同上

对 v3 快照中通过的 case:
  同上
```

#### 3.3 回归处理

```
├─ 有回归 → 用 references/root-cause-taxonomy.md 归类
│          回到对应层修复时遵守 references/description-rules.md
│          修复后重跑 Layer 4
│
└─ 无回归 → 进入 Step 4
```

最终报告格式参考 `references/templates.md`。

### Step 4: 保存快照 v4 [串行]

保存到 `.routing-quality/snapshots/v4-integration/`：

```
v4-integration/
  skills/              # 所有 skill description 最终版本
  test-cases.json      # 完整合并测试集
  test-results.json    # 全量测试结果
  failed-cases.json    # 最终失败 case 列表
  summary.md           # 最终摘要：各层准确率、回归情况、总结
```

### Step 5: 生成最终报告 [串行]

```markdown
# Integration Report

## 总结
- 总 case 数: {N}
- 总准确率: {X}%
- 各层准确率: L1 {X}% / L2 {X}% / L3 {X}%

## 各 Skill 表现
| Skill | Golden | Ambiguity | Boundary | 总计 |
|-------|--------|-----------|----------|------|
| ... | .../... | .../... | .../... | X% |

## 回归情况
[无 / 有（已修复）]

## 已知限制
[不可裁决 case 的兜底结果统计]

## Description 变更总结
[从 v1 到 v4，每个 skill description 的累计变更摘要]
```

---

## Todo 模板

进入 Layer 4 时创建：

- [ ] 合并 L1+L2+L3 全部 test cases（验证 <= 400）
- [ ] 运行全量测试
- [ ] 分析结果 + 回归检测
- [ ] 处理回归（如有）
- [ ] 保存快照 v4
- [ ] 生成最终报告

---

## 退出条件

- 总准确率 >= 95%
- Golden cases >= 98%
- 无未处理的回归
- 快照 v4 已保存
- 最终报告已生成
