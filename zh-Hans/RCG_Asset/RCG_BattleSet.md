---
title: 战斗组合 (RCG_BattleSet) 说明
description: 一场具体战斗的怪物配置 + 标签 + 敌人类型；地图节点实际进入战斗时用此资料生成
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗组合

> 程式类别名称：`RCG_BattleSet`

## 用途

**一场具体战斗的怪物配置**。例如「3 哥布林 + 1 哥布林大厨」「Boss 鲸鱼单体」「精英遭遇：龙 + 法师」都是不同的 BattleSet。地图节点触发战斗时，从 `RCG_BattleSetDropPool` 抽出一个 `RCG_BattleSet` 来生成本场战斗。

继承自 `RCG_Asset<RCG_BattleSet>`。

## 编辑器中的样貌

```
RCG_BattleSet: <ID>
    EnemyType  ← 敌人类型（普通 / 精英 / Boss）
    Tags       ← BattleSet 标签（用于 DropPool 筛选）
    MonsterSet ← 怪物配置（站位、ID、奖励资源等）
    Preview    ← 即时预览配置
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **MonsterSet** | 是 | 完整的怪物配置（`RCG_MonsterSet`：每个站位放哪只怪物、会掉哪些奖励资源） |
| **Tags** | 否 | BattleSet 标签（章节 / 场景类型等），DropPool 用标签过滤 |
| **EnemyType** | 是 | 敌人类型（`Normal` / `Elite` / `Boss`），影响奖励、战斗音乐、UI 标题 |

## 行为说明

### 取预设
`RCG_BattleSet.DefaultBattleSet` 直接取 `RCG_BattleSetGenData.DefaultID` 对应的资料；缺漏时的 fallback。

### 预览
显示 ID + EnemyType、Tags 列表、`MonsterSet.Preview` 的怪物站位视觉化。

## 注意事项

*   **EnemyType 是 Tag 物件**而非 enum：原本是 enum (`m_EnemyType = EnemyType.Normal`)，已改用 `RCG_EnemyTypeTagGenData` 并把旧栏位注解；新增类型直接编 Tag asset 即可。
*   **MonsterSet 内的奖励资源**：曾有过 `m_MonsterSet.m_RewardResources.Clear()` 的反序列化清理（已注解掉）；若需特殊清理逻辑可解注参考。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSet.cs`
*   **继承自**：`RCG_Asset<RCG_BattleSet>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_MonsterSet` | MonsterSet | `RCG_MonsterSet` | 主要配置容器 |
| `m_Tags` | Tags | `List<RCG_BattleSetTagGenData>` | DropPool 用 |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | 取代旧 enum |

### A.3 重要 Method

*   **`Preview` / `OnGUI`** — 编辑器渲染。
*   **`DefaultBattleSet`** (static) — `Util.GetData(RCG_BattleSetGenData.DefaultID)`。

### A.4 与其他系统的互动

*   **`RCG_MonsterSet`** — 怪物站位 + 奖励的详细容器。
*   **`RCG_BattleSetDropPool`** — 战斗组合的随机池。
*   **`RCG_BattleSetGenData`** — Asset Entry 包装。

### A.5 已知议题

*   旧版 `m_EnemyType` (enum) 已注解，标示迁移成 Tag 型别的历史。
