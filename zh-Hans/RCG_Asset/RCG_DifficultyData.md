---
title: 难度资料 (RCG_DifficultyData) 说明
description: 难度等级的全局倍率设定：HP / 攻击 / 商店价 / 暗雾 / 场地效果 / 解锁前置难度
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 难度资料

> 程式类别名称：`RCG_DifficultyData`

## 用途

**难度等级的设定模板**——简单 / 普通 / 困难 / 噩梦 / 教学等都是不同 ID 的 `RCG_DifficultyData`。本资料决定该难度下：
*   HP / 攻击 / 商品价的倍率
*   敌人技能基础等级加成
*   起始资源
*   篝火减少率
*   道具使用次数限制
*   永久套用的场地效果
*   进入此难度的前置条件（须完成哪些难度）

继承自 `RCG_Asset<RCG_DifficultyData>`。

## 编辑器中的样貌

```
RCG_DifficultyData: <ID>
    Name / IconSprite / IsHidden / SortOrder
    BaseLevel              ← 怪物基础等级（再经 UnitLevelData 曲线换算）
    HealthMult / AtkMult / PriceMult / SoulPriceMult / DarkMistMult
    SellPriceMult          ← 卖物品/装备时的价格倍率
    InitResources          ← 起始额外资源
    CampFireDecreaseRate / CampFireHealPercentageMult
    ItemUsageLimit         ← 道具使用次数限制
    AdditionalLength / AdditionalQuestProgress
    EnemySkillLevel        ← 敌人基础技能等级
    FieldEffects           ← 永久场地效果
    RequiredCompletedDifficulty  ← 前置条件（OR：完成任一即可解锁此难度）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name / IconSprite** | 是 | 显示名与图示 |
| **IsHidden** | — | 不在难度选单显示（测试用） |
| **SortOrder** | 是 | 选单排序 |
| **BaseLevel** | 是 | 怪物基础等级（会交给 `RCG_UnitLevelData` 曲线换算成最终等级） |
| **HealthMult / AtkMult** | 是 | HP / 攻击倍率（套到 `UnitLevelData.GetMaxHP / GetAtkMult` 结果上） |
| **PriceMult / SoulPriceMult** | 是 | 商人 / 灵魂商店价格倍率 |
| **DarkMistMult** | 是 | 暗雾累积速度倍率 |
| **SellPriceMult** | — | 玩家出售物品的倍率（预设 0.5） |
| **InitResources** | 否 | 开局额外资源 |
| **CampFireDecreaseRate** | — | 篝火使用次数的衰减率（每次使用后减的比例） |
| **CampFireHealPercentageMult** | — | 篝火治疗百分比倍率 |
| **ItemUsageLimit** | — | 每场战斗使用道具的上限（0 = 无限） |
| **AdditionalLength** | — | 大地图长度加成 |
| **AdditionalQuestProgress** | — | Quest 进度加成 |
| **EnemySkillLevel** | — | 敌人基础技能等级加成（影响 `MonsterLevelActionData.GetAction` 的 level） |
| **FieldEffects** | 否 | 永久套用的场地效果清单 |
| **RequiredCompletedDifficulty** | 否 | 前置难度（**OR**：完成任一即可解锁此难度） |

## 行为说明

### 全局倍率作用点
这些 mult 值会在不同地方被乘上：
*   `UnitLevelData.GetMaxHP / GetAtkMult` 套用 `HealthMult / AtkMult`。
*   商店 UI 显示售价时套用 `PriceMult / SoulPriceMult`。
*   出售物品计算售价时套用 `SellPriceMult`。
*   暗雾累积逻辑套用 `DarkMistMult`。
*   `MonsterLevelActionData.GetAction` 加上 `EnemySkillLevel`。

### `OnVictory()`
胜利并选择继续游戏时呼叫，自动 `++m_EnemySkillLevel`——让「无限模式」每胜一轮敌人技能升级一次。

### 描述
`LocalizedDescription` 从本地化系统取 key `<ID>_Description`，所以**描述要写在 zh-Hant.txt / en.txt 里**，而不是直接编在 Asset 上。

## 注意事项

*   **`RequiredCompletedDifficulty` 是 OR 关系**：满足任一即解锁。设计多阶解锁时要列出所有可接受的前置。
*   **`OnVictory` 永久修改 Asset**：`m_EnemySkillLevel` 是序列化栏位，胜利后会持久化。**这意味着每次胜利下次选同难度时都会更难**——这是「无限模式」设计，不是 bug。
*   **`Difficulty_Tutorial`** (`TutorialDifficultyID`) 是教学专用 ID，游戏一开始强制使用。
*   **`m_FieldEffects` 永久套用**：每场战斗开始时都会套上，无法移除。
*   **`Description` 用 i18n key**：直接编 Asset 上的 description 栏位无效，必须加到语言档。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DifficultyData.cs`
*   **继承自**：`RCG_Asset<RCG_DifficultyData>`
*   **AssetGroup**：`EditGameSetting`
*   **预设 ID**：`Difficulty_Normal`（也是建构式预设）

### A.2 栏位对照（节选）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_SpriteData` | |
| `m_IsHidden` | IsHidden | `bool` | |
| `m_SortOrder` | SortOrder | `int` | 预设 1 |
| `m_BaseLevel` | BaseLevel | `int` | 预设 1 |
| `m_FieldEffects` | FieldEffects | `List<RCG_FieldEffectGenData>` | |
| `m_HealthMult` / `m_AtkMult` / `m_PriceMult` / `m_SoulPriceMult` / `m_DarkMistMult` | 各 Mult | `float` | 预设 1f |
| `m_SellPriceMult` | SellPriceMult | `float` | 预设 0.5f |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_CampFireDecreaseRate` / `m_CampFireHealPercentageMult` | 篝火相关 | `float` | |
| `m_ItemUsageLimit` | ItemUsageLimit | `int` | |
| `m_AdditionalLength` / `m_AdditionalQuestProgress` | 进度加成 | `float / int` | |
| `m_EnemySkillLevel` | EnemySkillLevel | `int` | |
| `m_RequiredCompletedDifficulty` | RequiredCompletedDifficulty | `List<RCG_DifficultyGenData>` | OR 关系 |

### A.3 重要 Method

*   **`OnVictory()`** — `++m_EnemySkillLevel`（无限模式递增）。
*   **`LocalizedName / LocalizedDescription`** — i18n 对应；description 走 `<ID>_Description` key。
*   **建构式预设 `ID = "Difficulty_Normal"`**。

### A.4 与其他系统的互动

*   **`RCG_UnitLevelData.GetMaxHP / GetAtkMult`** — 套用 HealthMult / AtkMult。
*   **`RCG_MonsterLevelActionData.GetAction`** — 加 EnemySkillLevel。
*   **`RCG_DataService.Ins.m_DifficultyData`** — runtime 套用点。
*   **`RCG_DifficultyGenData`** — Asset Entry；`TutorialDifficulty` 为教学专用。

### A.5 已知议题

*   `OnVictory` 永久递增 `EnemySkillLevel` 会写回 Asset；玩家在不同存档玩同难度时会看到对方累积过的等级加成（**这是有意设计的「无限挑战」机制**）。
