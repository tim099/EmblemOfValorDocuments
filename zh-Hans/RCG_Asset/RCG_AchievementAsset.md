---
title: 成就资料 (RCG_AchievementAsset) 说明
description: 玩家可解锁的成就：条件达成后自动解锁；可串 Steam 成就，可有前置成就
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 成就资料

> 程式类别名称：`RCG_AchievementAsset`

## 用途

**玩家可解锁的成就模板**。条件达成后自动解锁（并可同步到 Steam 成就）。支援前置成就链、职业 / 角色相关标记、自订描述覆写。

继承自 `RCG_Asset<RCG_AchievementAsset>`。

## 编辑器中的样貌

```
RCG_AchievementAsset: <ID>
    IsClassAchievement / SkillTag             ← 职业相关旗标（true 时显示对应职业）
    IsCharacterAchievement / Character        ← 角色相关旗标
    OverrideDescription                       ← 覆盖自动描述
    OrderIndex                                ← 显示排序
    HasSteamAchievement / SteamAchievement    ← Steam 成就串接
    Conditions                                ← 达成条件清单（AND）
    HasPrerequisiteAchievement / PrerequisiteAchievement  ← 前置成就
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **IsClassAchievement** | — | 是否为「职业成就」（决定是否显示 SkillTag 栏位） |
| **SkillTag** | IsClassAchievement=true | 对应的职业（`RCG_SkillTagGenData`） |
| **IsCharacterAchievement** | — | 是否为「角色成就」 |
| **Character** | IsCharacterAchievement=true | 对应的角色（`RCG_CharacterGenData`） |
| **OverrideDescription** | 否 | 覆盖自动产生的描述（多语系，含参数高亮） |
| **OrderIndex** | — | 排序索引 |
| **HasSteamAchievement** | — | 是否串接 Steam 成就 |
| **SteamAchievement** | HasSteamAchievement=true | Steam 成就 ID |
| **Conditions** | 否 | 达成条件（`RCG_Condition` list；**AND** 关系） |
| **HasPrerequisiteAchievement** | — | 是否有前置成就 |
| **PrerequisiteAchievement** | HasPrerequisiteAchievement=true | 必须先达成的成就 |

## 行为说明

### 条件检查 (`CheckAchievementCondition`)
*   `m_Conditions` 为空 → 回 false（没条件视为「不达成」，与 GameChallenge 的「没条件视为通过」相反）。
*   `m_Conditions.CheckConditions_AND` → 所有条件都满足才回 true。
*   有前置成就且本成就条件已通过 → 递回检查前置条件。

### 解锁 (`CheckAchievement`)
1. 若有 Steam 成就且**已达成**（`steamAchievement.GetStat() == true`）→ 直接回 true（避免重复触发）。
2. 跑 `CheckAchievementCondition`。
3. 达成 + 有 Steam 成就 → `steamAchievement.SetStat(true)` 上报 Steam。

### 描述生成 (`RequirementDes`)
*   `OverrideDescription` 有设 → 用它（含参数高亮 `Term` 颜色 tag）。
*   否则 → 串接 `m_Conditions` 各自的 `GetShortName()`。

## 注意事项

*   **`m_Conditions` 为空回传 false**：与直觉相反——空条件不会自动达成。**此设计与 GameChallenge 的 `IsEmpty || Unlocked` 不一致**，要小心。
*   **Steam 成就一旦达成不会重置**：本档不提供「取消成就」流程。
*   **OverrideDescription 含参数高亮**：用 `RCG_Extensions.TagColors.Term` 对参数上色，编辑时要把参数写成 `{0}` 之类的占位符（具体规则见 `RCG_LocalizeData.GetName`）。
*   **前置成就链的条件检查**：`CheckAchievementCondition` 会递回跑前置的条件——前置条件满足才能解锁本成就，但**前置成就本身是否已解锁（GetStat == true）这层不影响本成就逻辑**。
*   **注解 `// TODO: use as condition QWQ?`** 标示：`m_SkillTag` / `m_Character` 目前只是「分类旗标」，没当条件用；未来可能会作为自动条件。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AchievementAsset.cs`
*   **继承自**：`RCG_Asset<RCG_AchievementAsset>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_IsClassAchievement` | IsClassAchievement | `bool` | |
| `m_SkillTag` | SkillTag | `RCG_SkillTagGenData` | `Conditional(IsClassAchievement)` |
| `m_IsCharacterAchievement` | IsCharacterAchievement | `bool` | |
| `m_Character` | Character | `RCG_CharacterGenData` | `Conditional(IsCharacterAchievement)` |
| `m_OverrideDescription` | OverrideDescription | `RCG_LocalizeData` | |
| `m_OrderIndex` | OrderIndex | `int` | |
| `m_HasSteamAchievement` | HasSteamAchievement | `bool` | |
| `m_SteamAchievement` | SteamAchievement | `UCL_SteamAchievementEntry` | `Conditional(HasSteamAchievement)` |
| `m_Conditions` | Conditions | `List<RCG_Condition>` | AND |
| `m_HasPrerequisiteAchievement` | HasPrerequisiteAchievement | `bool` | |
| `m_PrerequisiteAchievement` | PrerequisiteAchievement | `RCG_AchievementEntry` | `Conditional(HasPrerequisiteAchievement)` |

### A.3 重要 Method

*   **`CheckAchievement(triggerEffectData)`** — 入口：Steam 已达成快检 → 条件 → 前置 → 上报 Steam。
*   **`CheckAchievementCondition(triggerEffectData)`** — 纯条件检查（不上报）；递回前置。
*   **`RequirementDes` (property)** — 自动 / 覆写描述。

### A.4 与其他系统的互动

*   **`UCL_SteamAchievementEntry`** — Steam SDK 串接。
*   **`RCG_Condition`** — 条件元素。
*   **`RCG_AchievementEntry`** — Asset Entry；预设 `CreatorAchievement`。
*   **`RCG_SkillTagGenData / RCG_CharacterGenData`** — 分类用。

### A.5 已知议题

*   `m_SkillTag` / `m_Character` 标 `// TODO: use as condition QWQ?`，目前只是分类，未自动转成条件。
*   空 `m_Conditions` 回 false 的行为与直觉相反（与 GameChallenge 不一致）。
