---
title: 增援 说明
description: 召唤增援单位（目前仅敌方）；普通战斗随机抽，精英 / Boss 战用预设模板
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 增援

> 程式类别名称：`RCG_ReinforcementSetting`

## 用途
**召唤敌方增援单位**到战场。目前**仅支援怪物增援**（不能用于玩家方）。常见用途：
*   Boss 召唤杂兵
*   特定回合自动补充敌人

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DefaultUnit** | 是 | 预设增援单位（**精英 / 魔王战专用**）。普通战斗不使用此栏位，会从 `MonsterSet` 随机抽。 |

## 行为说明
*   读取 `RCG_BattleField.Ins.MonsterSet`，取出当前敌人类型的可用单位清单。
*   **普通战斗**：从清单中随机挑一个。
*   **精英 / Boss 战**：直接用 `DefaultUnit` — 避免生成过多精英造成失衡。
*   优先在 `iData.TargetPositions` 指定的空位生成；无指定时自动找空位。
*   描述格式：「**召唤怪物增援**」（i18n key `ReinforcementDes_Monster`）。

## 注意事项
*   **目前仅怪物增援**：程式内 `bool aIsMonster = true` 写死；要召唤我方友军请用「**召唤**」设定。
*   **无空位时的行为**：未填补成功则静默跳过 — 玩家可能误以为卡片无效。
*   **精英 / Boss 预设限制**：是为了避免「增援卡 + 精英」连锁召唤一堆精英 — 此设计选择有 `// TODO: 应该移到 Condition` 注解，未来可能重构。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ReinforcementSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ReinforcementSetting` → 「增援」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_DefaultUnit` | `DefaultUnit` | `RCG_UnitGenData` | — | 精英 / Boss 战使用 |

### A.3 重要 Method 摘要
*   **`LocalizeKey` (private)** → 写死回传 `"ReinforcementDes_Monster"`（注释：目前仅敌人增援）。
*   **`AddAction`** (async)：
    1. 从 `MonsterSet.GetUnitSpawnDatas(EnemyType)` 随机抽 → `aTargetUnit`。
    2. 若敌人类型 ≠ Normal → 改用 `m_DefaultUnit.GetData()`。
    3. 尝试从 `iData.TargetPositions` 取空位；否则 `RCG_BattleField` 自动分配（档案 80 行外的后续逻辑）。

### A.4 与其他系统的互动
*   **`RCG_BattleField.Ins.MonsterSet.GetUnitSpawnDatas`**：候选怪物清单来源。
*   **`RCG_BattleManager.Ins.EnemyType`** / **`RCG_EnemyTypeTagGenData.s_EnemyType_Normal`**：战斗类型判断。
*   **`UCL_Random.Instance.Next`**：候选选择的随机来源。

### A.5 已知议题
*   `// TODO: 目前只有普通战斗获得增援 (应该移到Condition)` — 应该抽成条件判断，而非写死于增援逻辑内。
*   `bool aIsMonster = true` 写死，无法做友军增援。
