# Routing Quality Engineering

Planner+Skill 路由系统的质量工程方法论。包含调优（Building）和被动修复（Fixing）两套互补流程。

---

## 选择哪个流程？

| 场景 | 使用 | 文档 |
|------|------|------|
| 首次接入 / 项目结构变化 | Phase 0 发现 | [phase0-discovery.md](phase0-discovery.md) |
| 系统性提升路由准确率 | Building（Layer 1→4） | 见下方 |
| 单个路由 bug 快速修复 | Fixing | [fixing-routing.md](fixing-routing.md) |

**从哪里开始？**

- 已有 Agent 骨架和 skills 但没跑过路由质量 → 从 Phase 0 开始
- Phase 0 已完成（有 config 文件）→ 直接从 Layer 1 开始
- 只是修一个 bug → 用 `/fixing-routing`

---

## Building: 三层 + 集成

按顺序执行，每层独立文档、独立会话、靠文件衔接：

```
Phase 0 → Layer 1 → Layer 2 → Layer 3 → Layer 4
发现       锁核心     清模糊     裁边界     全量验证
```

**TDD 原则：**
- **Iron Law：** 不读 description 就不改 description。不跑测试就不知道该改什么。
- **RED：** Phase 0 收集用户意图（Baseline Intent），作为所有层 test cases 的来源。
- **GREEN：** Description 是为了让测试通过而修改的"代码"，不是 test cases 的来源。只做让失败 case 通过的最小改动。
- **REFACTOR：** 全绿后精简 description（去冗余、查跨 skill 一致性、控长度），保持全绿。
- **粒度适配：** 每轮批量定义 3-5 个 case（非逐个），因为 LLM 推理成本高于代码编译。这是有意识的取舍，不是对 TDD 的违反。

| 层 | 目标 | 文档 | 退出条件 |
|----|------|------|---------|
| **Phase 0** | 生成项目 config | [phase0-discovery.md](phase0-discovery.md) | config 已确认 |
| **Layer 1** | Golden cases 100% | [layer1-golden-cases.md](layer1-golden-cases.md) | >= 95% + 快照 v1 |
| **Layer 2** | 清理模糊输入 | [layer2-ambiguity-cases.md](layer2-ambiguity-cases.md) | >= 95% + 快照 v2 |
| **Layer 3** | 裁决竞争边界 | [layer3-boundary-cases.md](layer3-boundary-cases.md) | >= 95% + 快照 v3 |
| **Layer 4** | 全量回归验证 | [layer4-integration.md](layer4-integration.md) | >= 95% + 快照 v4 |

### 层间衔接

每层完成后保存快照到 `.routing-quality/snapshots/`，下一层从快照读取输入。新会话只需加载当前层文档 + config + 上一层快照，上下文干净。

### 可以跳层吗？

- Phase 0 必须先跑（生成 config）
- Layer 1 必须先跑（建立 golden baseline）
- Layer 2/3 可按需跳过，但 Layer 4 集成验证建议始终执行

---

## Fixing: 被动修复

6 个 Phase 的固定流程，每步等用户确认：

```
收集证据 → 定位根因 → 写 Spec → 执行 → 测试 → 记录
```

详见 [fixing-routing.md](fixing-routing.md)

**与 Building 的关系：** Fixing 发现系统性问题时，建议切换到 Building 流程。

---

## 参考文件

| 文件 | 用途 | 使用时机 |
|------|------|---------|
| [references/root-cause-taxonomy.md](references/root-cause-taxonomy.md) | A-F 根因分类 | 定位根因时 |
| [references/description-rules.md](references/description-rules.md) | 四规则 + 反模式 | 修改 description 时 |
| [references/templates.md](references/templates.md) | 标准模板 | 记录证据、写 Spec、保存快照时 |

---

## 产出目录

```
.routing-quality/
  config.md              ← Phase 0 生成
  snapshots/
    v1-golden/           ← Layer 1 产出
    v2-ambiguity/        ← Layer 2 产出
    v3-boundary/         ← Layer 3 产出
    v4-integration/      ← Layer 4 产出
```

---

## 约束

- 总 test cases <= 400
- 每层文档 < 800 词，自包含
- 平台无关：本目录下的文档不依赖任何特定工具
