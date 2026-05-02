---
title: 卡牌强化资料 (RCG_CardEnhenceData) 说明
description: 卡牌强化的「分支模板」：强化条件、加 effect、改费用、改卡牌类型、禁用使用类型
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌强化资料

> 程式类别名称：`RCG_CardEnhenceData`

## 用途

**卡牌强化的「分支模板」**。每个 Asset 描述「强化能做什么」：加新 effect、改费用、改卡牌类型、加标签、改名（如 α、β、γ）。卡牌端 `RCG_CardData.m_EnhencePool` 引用一个 `RCG_CardEnhenceDropPool`，池内掉的就是这些 `RCG_CardEnhenceData` 分支。

继承自 `RCG_Asset<RCG_CardEnhenceData>`。

## 编辑器中的样貌

```
RCG_CardEnhenceData: <ID>
    LocalizeName              ← 强化名（α / β / γ；空白用 +）
    Rarity                    ← 强化分支稀有度
    Conditions                ← AND 条件群（哪些卡能套用此强化）
    Effects                   ← 强化加上的新效果
    CardTags                  ← 强化加上的卡牌标签
    EnhenceSettings           ← 额外的强化设定（修改既有 effect 的条件式套件）
    BannedUsedType            ← 禁止此使用类型的卡套用此强化
    CostAlter                 ← 费用变化（+/-）
    SetCardType / CardType    ← 是否强制改卡牌类型（例：咏唱→普通）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **LocalizeName** | 否 | 强化命名替代（如 α / β / γ）；空白用 `+` 前缀 |
| **Rarity** | 是 | 强化分支稀有度（影响掉落权重） |
| **Conditions** | 否 | AND 条件式：套用此强化的卡需满足全部条件 |
| **Effects** | 否 | 强化会加到卡上的新效果 |
| **CardTags** | 否 | 强化加上的卡牌标签（不重复时才加） |
| **EnhenceSettings** | 否 | 进一步修改既有效果的设定（如：「OnPlay 第一个效果改伤害」） |
| **BannedUsedType** | 否 | 禁止特定使用类型的卡牌套用（例：`Unplayable` 卡禁止强化） |
| **CostAlter** | — | 费用变化值（可正可负） |
| **SetCardType** | — | 是否强制改卡牌类型 |
| **CardType** | SetCardType=true | 改成什么类型（`Default` / `Chant`） |

## 行为说明

### 条件检查 (`CheckCondition`)
1. `m_BannedUsedType` 含目标卡的 `UsedType` → 不可用。
2. `m_Conditions.CheckCondition(data)` 不通过 → 不可用。
3. 每个 `EnhenceSettings` 各自 `CheckCondition(data)` 全通过 → 可用。

### 套用 (`Enhence(card)`)
1. 把 `m_LocalizeName` 写入卡片的 `m_EnhenceLocalize`（显示用）。
2. 加入所有 enabled 的 `Effects`（深拷贝）。
3. `Cost += m_CostAlter`。
4. `SetCardType` → 套用新类型。
5. `CardTags` → 加标签（不重复）。
6. 对每个 `EnhenceSettings` 跑 `Enhence(enhenceData)`（修改既有 effect）。

## 注意事项

*   **强化是「永久套用」**：clone 过的卡片会被改名 / 加效果 / 改费用，原始卡不动。
*   **`BannedUsedType` 通常会包含 `Unplayable`**：某些「无法手动打出」的卡（系统卡、隐藏效果卡）不该被强化；曾有自动补上 `Unplayable` 的反序列化逻辑（已注解）。
*   **`EnhenceSettings` 与 `Effects` 的差异**：`Effects` 是「加新的 effect」；`EnhenceSettings` 是「修改现有 effect」（例如：把第一个伤害效果改成 +5 伤害）。
*   **空白 `LocalizeName`** 表示用预设 `+1` / `+2` 显示；非空白会被 `+α` / `α` 之类取代。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardEnhenceData.cs`
*   **继承自**：`RCG_Asset<RCG_CardEnhenceData>`
*   **AssetGroup**：`EditItems`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_LocalizeName` | LocalizeName | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Conditions` | Conditions | `RCG_CE_AND_Condition` | AND 群组 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | |
| `m_EnhenceSettings` | EnhenceSettings | `List<RCG_CardEnhenceSetting>` | |
| `m_BannedUsedType` | BannedUsedType | `List<RCG_UsedTypeTagGenData>` | |
| `m_CostAlter` | CostAlter | `int` | |
| `m_SetCardType` | SetCardType | `bool` | |
| `m_CardType` | CardType | `CardType` enum | `Conditional(SetCardType)` |

### A.3 重要 Method 摘要

*   **`CheckCondition(EnhenceData)`** — BannedUsedType + Conditions + EnhenceSettings 三层检查。
*   **`Enhence(RCG_CardData)`** — 主套用逻辑：写名 → 加 effects → 改 cost → 改 cardType → 加 cardTags → 套 EnhenceSettings。
*   **`EnhenceSettings` (property)** — 过滤 `IsEnable = true` 的 settings。

### A.4 与其他系统的互动

*   **`RCG_CardData.m_EnhencePool`** / **`GetEnhenceBranchs`** — 引用此资料的入口。
*   **`RCG_CardEnhenceDropPool`** — 强化分支随机池。
*   **`RCG_CardEnhenceCondition.EnhenceData`** — 条件检查与套用时的传递容器。
*   **`RCG_CardEnhenceSetting`** — 修改既有 effect 的子设定。
*   **`RCG_CardEnhenceGenData`** — Asset Entry 包装；预设 ID = `"Defense"`。

### A.5 已知议题

*   `DeserializeFromJson` 内有「自动补 Unplayable 到 BannedUsedType」的逻辑被注解，标示旧版自动行为已停用。
