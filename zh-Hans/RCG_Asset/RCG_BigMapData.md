---
title: 大地图资料 (RCG_BigMapData) 说明
description: 一张完整大地图的设定（章节骨架）：任务生成模式、状态机、暗雾、解锁、起始装备、进度规则
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 大地图资料

> 程式类别名称：`RCG_BigMapData`

## 用途

**一张完整大地图的章节骨架设定**。每个 BigMap 是一段完整的游戏历程：从起点走到 Boss、中间踩各种任务节点、依任务 tag 推进进度。本资料定义：
*   任务生成模式（预设规则 / 任务池 / 池+混合）
*   多阶段状态机（`MapState`：每阶段有自己的任务池、奖励池、进入下阶段条件）
*   暗雾设定、解锁条件、起始装备、可用挑战目标
*   常驻 + 最终任务池

继承自 `RCG_Asset<RCG_BigMapData>`，实作介面：`RCGI_Unloackable`。

## 编辑器中的样貌

```
RCG_BigMapData: <ID>
    Name / Description / QuestIcon / BGM / BigMap (prefab)
    BigMapBackground
    TopMenuState
    QuestGenerateType         ← Default / QuestPool / QuestPoolMix
    MaxShowingQuestCount      ← 同时最多显示多少个任务（Default 模式）
    QuestProgressToComplete   ← 解锁最终关所需任务进度（0 = 忽略）
    QuestPoolSetting          ← (QuestPool/QuestPoolMix) 多阶段状态机
    PermanentQuests           ← (QuestPoolMix) 常驻任务
    FinalQuests               ← (QuestPoolMix) 最终任务
    QuestProgressTags         ← Boss 任务的 Tag 标签（Forest/Mountain/Boss⋯）
    DetailSetting             ← 隐藏旗标 / 教学设定 / 解锁 / 起始装备等
```

## 主要栏位（节选）

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **QuestGenerateType** | 是 | `Default`（预设规则）/ `QuestPool`（纯任务池）/ `QuestPoolMix`（池+常驻+最终混合） |
| **MaxShowingQuestCount** | Default 时 | 同时可选任务数（预设 3） |
| **QuestProgressToComplete** | — | 解锁最终关所需任务进度数；> 0 才会显示进度指标 |
| **QuestPoolSetting** | QuestPool/Mix | 多阶段状态机（每个 `MapState` 有自己的任务池 / 奖励池 / 进入条件） |
| **PermanentQuests** | QuestPoolMix | 常驻任务（如休息点）：总是出现且**不计算显示数量限制** |
| **FinalQuests** | QuestPoolMix | 最终任务（达成 Final 条件后才出现） |
| **QuestProgressTags** | Default/Mix | 哪些 Tag 算「进度任务」（通常是 Boss 标签） |
| **DetailSetting** | 是 | 细节设定：HideThisBigMap / RandomQuestOrder / Difficulty / EnterBigMapTutorial / ShowMode / IsTutorial / BigMapType (Default/Loop) / Unlock / OverridingCharacters / StartingEquipments / AvailableChallenges 等 |

`MapState` 内含每阶段资料：
*   `MaxShowingQuestCount` / `QuestDropSettings`（含条件式权重的任务池清单）
*   `EquipmentRewardPools / UnitSkillRewardPools / CardRewardPools / ItemRewardPools`：阶段专属奖励池
*   `NextStateConditions`（OR）/ `FinalConditions`（OR）

## 行为说明

### `EnterBigMap(newLoop)`
新一局时重置进度与通过任务记录。

### `GenerateQuests()`
依 `QuestGenerateType` 分流：
*   **Default**：不在这里，由其他 manager 处理。
*   **QuestPool**：纯抽当前 MapState 的任务池。
*   **QuestPoolMix**：复杂逻辑——
    1. 起始祝福事件（首次进入时）
    2. 上个任务若为常驻/特殊 → 生成常驻任务 + 跳过选项
    3. 上个任务为 Boss → 重新随机生成（按 ProgressTag 筛选）
    4. 满足最终条件 → 只显示最终任务
    5. 一般情况：依环境 Tag 去重、Boss 任务 + 一般任务按比例混合

### `QuestStart(questData)`
进入任务时若 `CurState.CheckFinal()` 为 true → 标记为最终地图（让胜利后弹通关 UI）。

### `PassedQuest(questId, progress)`
通关任务时：
1. `m_PassedQuests` 加入此 ID。
2. 写入 `MapStatePassQuestCount` / `PassQuestCount` Tag（GameTag 系统）。
3. 检查 `CurState.CheckTransition()` → 进入下一个 MapState（并重置 MapStatePassQuestCount）。

### `QuestCompleteAsync()`
若是最终地图 → 显示 `RCG_VictoryUI`，由玩家选择是否继续（继续 = 进入下一轮 Loop）。

## 注意事项

*   **`QuestGenerateType.Default`** 是旧版规则，新地图建议用 `QuestPoolMix`。
*   **`PermanentQuests` 不计数量限制**：要做休息点 / 永久商店这种「总是显示」的任务节点放这里。
*   **`m_QuestProgressToComplete` 与 `FinalQuests`** 一起使用：未达进度时 FinalQuests 不会出现。
*   **`HideThisBigMap`** 真值不止来自 `m_HideThisBigMap`：还有「`OverrideAvailableGameModes` 不含当前 GameVersion」也会隐藏。Demo 版锁部分章节用此机制。
*   **`ShowMode`** 控制教学模式下的显示：`ShowIfEnableTutorial` / `ShowIfDisableTutorial` 对应教学状态切换。
*   **`Tutorial` 写死 ID**：`BigMap_Challenge_Card`，是教学专用大地图。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_BigMapData.cs`
*   **继承自**：`RCG_Asset<RCG_BigMapData>`
*   **实作介面**：`RCGI_Unloackable`
*   **AssetGroup**：`EditQuestSetting`
*   **常数**：`TutorialName = "BigMap_Challenge_Card"`

### A.2 栏位对照（节选）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` / `m_Description` | Name / Description | `RCG_LocalizeData` | |
| `m_QuestIcon` | QuestIcon | `RCG_SpriteData` | |
| `m_BGM` | BGM | `RCG_BGMGenData` | |
| `m_BigMap` | BigMap | `RCG_PrefabResData` | 大地图 prefab |
| `m_BigMapBackground` | BigMapBackground | `RCG_SpriteData` | |
| `m_TopMenuState` | TopMenuState | `TopMenuState` enum | |
| `m_QuestGenerateType` | QuestGenerateType | `QuestGenerateType` enum | |
| `m_MaxShowingQuestCount` | MaxShowingQuestCount | `int` | `Conditional(Default)` |
| `m_QuestProgressToComplete` | QuestProgressToComplete | `int` | |
| `m_QuestPoolSetting` | QuestPoolSetting | `QuestPoolSetting`（巢状） | `Conditional(QuestPool / QuestPoolMix)` |
| `m_PermanentQuests` / `m_FinalQuests` | 常驻 / 最终 | `MapState` | `Conditional(QuestPoolMix)` |
| `m_QuestProgressTags` | QuestProgressTags | `List<RCG_QuestData.QuestTag>` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting`（巢状） | |

### A.3 重要 Method 摘要

*   **`EnterBigMap(newLoop)`** — 进入大地图；新局重置进度。
*   **`GenerateQuests()`** — 主任务生成入口。
*   **`GenerateQuestsQuestPool()`** / **`GenerateQuestsQuestPoolMix()`** — 两套生成实作。
*   **`QuestStart(questData)`** — 任务开始；标记最终地图旗标。
*   **`PassedQuest(questId, progress)`** — 通关时更新 Tag、进入下个 MapState。
*   **`QuestCompleteAsync(token, map)`** — 任务完成；最终地图会弹通关 UI。
*   **`AutoTriggerQuests()`** — 起始祝福事件 / 自动进入单一任务。
*   **`CurState`** — `m_QuestPoolSetting.m_MapStates[saveData.m_State]`。
*   **`HideThisBigMap`** / **`ShowThisBigMap`** — 多重判断可见性。
*   **`MapState.GetQuestDropPool`** — 按条件式权重随机抽池。
*   **`MapState.CheckTransition` / `CheckFinal`** — 状态切换 / 最终判定（OR）。

### A.4 与其他系统的互动

*   **`RCG_BigMapManager`** — runtime 主入口。
*   **`RCG_QuestData`** / **`RCG_QuestDropPool`** — 任务系统。
*   **`RCG_BigMapGenData`** — Asset Entry；预设 `BigMap_Challenge_Card`。
*   **`BigMapSaveData`** — runtime 进度储存。
*   **`RCG_DifficultyData.m_AdditionalQuestProgress`** — 难度附加进度。
*   **`RCG_GameSettingData.m_GameVersion`** — Demo 版本锁定判断。
*   **`RCG_TutorialService`** — 教学模式可见性判断。

### A.5 已知议题

*   `// QWQ23` 标记散落多处，标示旧版资料格式迁移与待确认逻辑。
*   `GenerateQuestsQuestPoolMix` 逻辑极长（300+ 行单一方法），多处注解掉的 LogError，维护困难。
