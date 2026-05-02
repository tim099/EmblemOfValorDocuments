---
title: 单位技能资料 (RCG_UnitSkillData) 说明
description: 角色学会的「技能」——类似装备但不占装备槽；提供被动效果、领袖技、学习特典
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 单位技能资料

> 程式类别名称：`RCG_UnitSkillData`

## 用途

**角色学会的技能模板**。类似装备，但不占装备槽。每个技能可以：
*   提供被动效果（战斗中各种 trigger 触发）
*   标记为「**领袖技**」（只在角色站第一位时生效）
*   学习时触发一次性事件（例：永久 +1 HP、解锁额外抽卡格）

继承自 `RCG_Asset<RCG_UnitSkillData>`，实作介面：`RCGI_Status`（战斗状态系统）/ `RCGI_Unloackable`（解锁）。

## 编辑器中的样貌

```
RCG_UnitSkillData: <ID>
    Name(多国语言)
    Icon                          ← 技能图
    Effects                       ← 战斗中触发的效果（OnPlay / OnTurnStart / ...）
    AcquireSkillEvents            ← 学会时触发的一次性事件
    SkillTags                     ← 需要的专精（角色须符合才能学）
    Tags                          ← 一般标签
    UnitSkillDescription / Template ← 描述方式（Auto / Manually）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 技能显示名（多语系） |
| **Icon** | 是 | 技能图示 |
| **Effects** | 否 | 战斗中各种 trigger 上要跑的效果（OnPlay / OnTurnStart / OnAttack…） |
| **AcquireSkillEvents** | 否 | 学会技能时触发的事件（例：扩充手牌数、永久强化） |
| **SkillTags** | 否 | 必要专精——角色拥有的专精须**全包含**这里列的才能学（AND 关系） |
| **Tags** | 否 | 一般物品/技能标签（用于分类、条件判断） |
| **CanLearnRepeatedly** | — | 是否可重复学习（叠加效果） |
| **HideThisSkill** | — | 学会后**不显示在技能栏**（适合「只触发学习特典」这种隐藏增益） |
| **IsLeaderSkill** | — | 是否为**领袖技**：只在角色站第一位时触发 Effects |
| **CanDrop** | — | 是否能从掉落池抽到（false = 只能透过特定途径取得） |
| **InitCounters** | 否 | 起始计数器值（给 Effects 中的计数器类效果使用） |
| **Unlock** | 否 | 解锁条件 |
| **UnitSkillDescriptionType** | 是 | `Auto`（自动由 Effects 合成）或 `Manually`（手动撰写） |
| **UnitSkillDescription** | Manually 时 | 手动描述（多语系） |
| **DescriptionTemplate** | Manually 时 | 含 `{(OnPlay.0)}` 等占位符的范本字串 |

## 行为说明

### 触发判定
`OnUnitState(triggerOn, data)` 在每个 trigger 上：
1. 从 `Effects` 取出该 trigger 的所有效果。
2. 若 `IsLeaderSkill = true` 且拥有者**不是队伍第一位**（与 `RCG_BattleField.LeadUnit` 比对 ID），跳过。
3. 否则逐一触发。

### 学会时 (`OnAquireSkill`)
所有 `AcquireSkillEvents` 加入 `RCG_MapEventManager`，套到该角色身上（会立刻或在合适时机 fire）。

### 描述生成
*   **Auto**：依「领袖技标签 → AcquireEvents 描述 → Effects 描述」顺序自动串接。
*   **Manually**：以 `UnitSkillDescription` 为本体，透过 `GetDescriptionParams()` 替换 `(OnPlay.0)` 之类的占位符；可按 `Generate Description Template` 按钮从现有 Effects 自动产出范本。

### Tooltip Infos
`Infos` 属性聚合所有 effect 的 `CardInfoData`；领袖技会在最前插入 `BattleTag_LeadAbility` 的解说。

## 注意事项

*   **`SkillTags` 是 AND 关系**：列了多个专精表示「全部都要有」才能学；要做「择一」需开两个技能。
*   **`HideThisSkill = true`** 适合纯粹做「学会时加 max HP」这类**永久效果而无被动**的技能；玩家看不到 buff icon 但效果已生效。
*   **`IsLeaderSkill`** 与队伍切换领袖机制绑定；领袖切换时不会自动重触发 Effects（`OnPlay` 不会重 fire）。
*   **`CanDrop = false`** 的技能不会被 DropPool 抽到，常用于剧情解锁技能。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitSkillData.cs`
*   **继承自**：`RCG_Asset<RCG_UnitSkillData>`
*   **实作介面**：`RCGI_Status` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`
*   **预设 Icon 路径**：`RCG_SpriteData.SpriteFolder + "/UnitSkills"`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` |
| `m_Icon` | Icon | `RCG_SpriteData` | `Icon` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | `Effects` |
| `m_AcquireSkillEvents` | AcquireSkillEvents | `List<RCG_MapEvent>` | `AcquireSkillEvents` |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | `Tags` |
| `m_CanLearnRepeatedly` | CanLearnRepeatedly | `bool` | `CanLearnRepeatedly` |
| `m_HideThisSkill` | HideThisSkill | `bool` | `HideThisSkill` (Header `HideThisSkillDes`) |
| `m_IsLeaderSkill` | IsLeaderSkill | `bool` | `IsLeaderSkill` |
| `m_CanDrop` | CanDrop | `bool` | `CanDrop` |
| `m_InitCounters` | InitCounters | `List<int>` | `InitCounters` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` |
| `m_UnitSkillDescriptionType` | DescriptionType | `CardDescriptionType` | — |
| `m_UnitSkillDescription` | Description | `RCG_LocalizeData` | `Conditional(Manually)` |
| `m_DescriptionTemplate` | Template | `string` | `Conditional(Manually)` |

### A.3 重要 Method 摘要

*   **`OnUnitState(RCG_EffectTriggerOn, TriggerEffectData)`** — 主要触发入口；含领袖技判断。
*   **`TriggerOnUnitState(RCG_EffectTriggerOn)`** — 是否在此 trigger 有 effect（给 trigger 系统做 quick check）。
*   **`OnAquireSkill(RCG_CharacterData)`** — 学会时把 `AcquireSkillEvents` 加入 `RCG_MapEventManager`。
*   **`CheckRequireSkill(HashSet<RCG_SkillTagGenData>)`** — 角色当前 skills 是否包含全部 `m_SkillTags`。
*   **`Description` (property)** — Auto / Manually 的描述逻辑。
*   **`GetDescriptionTemplate / GetDescriptionParams`** — Manually 模式下的范本生成 / 参数抽取。
*   **`Status` (property)** — 回 `new RCG_StatusGenData(StatusType.UnitSkill, ID)`，作为 RCGI_Status 介面实作。

### A.4 与其他系统的互动

*   **`RCG_CommonEffect`** — 触发效果单位。
*   **`RCG_MapEvent` / `RCG_MapEventManager`** — 学习特典事件系统。
*   **`RCG_BattleField.LeadUnit`** — 领袖技判断。
*   **`RCG_BattleTag.Util.GetData("BattleTag_LeadAbility")`** — 领袖技标签。
