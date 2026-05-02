---
title: 任务资料 (RCG_QuestData) 说明
description: 一段任务（小地图）的完整定义：节点配置、战斗池、奖励池、目标、暗雾、随机生成
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 任务资料

> 程式类别名称：`RCG_QuestData`

## 用途

**一段「任务」（小地图）的完整定义**。从大地图进入任务节点后，游戏会生成这段任务对应的小地图：节点配置、节点间路径、敌人配置、奖励池、暗雾规则、任务目标。任务支援多种生成方式：手动编辑、纯随机、混合随机、杀戮尖塔风格、洞窟、遗迹⋯

继承自 `RCG_Asset<RCG_QuestData>`，实作介面：`UCL.Core.UI.UCLI_FieldOnGUI`。

## 编辑器中的样貌

```
RCG_QuestData: <ID>
    Name / QuestObjective / QuestIcon / BGM / Map prefab
    QuestGoals                 ← 任务目标（主要 / 次要）
    FieldEffects               ← 场地效果（可开关）
    StoryPool                  ← 此任务的事件池
    QuestBattles               ← 预设战斗池（按敌人类型分）
    Card / Item / EquipmentDropPools  ← 各类奖励池（按敌人类型分）
    BattleScene                ← 预设战斗场景
    QuestType                  ← Default / RandomGen / MixRandom / SlayTheSpire / CaveGen / RuinGen / TestEvents / TestBattles
    [QuestType 对应的子设定]      ← 各种 RandomGenData
    DetailSetting              ← 条件 / Tag / 难度 / 暗雾 / 地图大小 / 自动进入等
    MapEditData                ← 节点编辑资料（隐藏，用 QuestEditor 编）
    [按钮] Open QuestEditor    ← Editor playing 时可开节点编辑器
    [按钮] Export Quest Info   ← 汇出此任务的怪物 / 战斗资讯到 Markdown
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name / QuestObjective** | 是 | 任务名 / 目标描述（多语系） |
| **QuestIcon** | 是 | 任务图（大地图节点显示） |
| **BGM** | 是 | 背景音乐 |
| **Map** | 是 | 对应的小地图 Prefab |
| **QuestGoals** | 否 | 主要 / 次要任务目标（达成触发奖励） |
| **FieldEffects** | 否 | 此任务的场地效果（可开关） |
| **StoryPool** | 否 | 此任务的故事事件池 |
| **QuestBattles** | 否 | 按敌人类型对应的战斗池（普通 / 精英 / Boss⋯） |
| **CardDropPools / ItemDropPools / EquipmentDropPools** | 否 | 按敌人类型对应的奖励池；缺漏时 fallback 到 EnemyType 的预设池 |
| **BattleScene** | 否 | 预设战斗场景（无指定时用此） |
| **QuestType** | 是 | 地图生成方式（决定下方哪个子设定可用） |
| **DetailSetting** | 是 | 细节（条件 / Tag / 难度 / 暗雾设定 / 地图大小 / `AutoEnterIfOnly` 等） |

`DetailSetting` 内含：

| 子栏位 | 说明 |
|---|---|
| **CompleteConditions** | 显示前置条件（AND；不符合则隐藏） |
| **IncompatibleConditions** | 互斥条件（不符合则隐藏） |
| **TopMenuState / QuestTags / QuestLv** | 上方选单状态 / Tag 列表 / 难度等级 |
| **QuestLength / QuestDarkMistChange** | 任务长度 / 暗雾变化（会被 `DifficultyData.m_DarkMistMult` 乘） |
| **DarkMistRelatedSettings** | 暗雾细节：移动时是否上升、暗雾外观、节点抗性自动分配 |
| **Difficulty / AddDifficultyAfterPass** | 任务基础难度 / 通关后的难度增量 |
| **QuestProgress / QuestEventIcons** | 进度值 / 事件图示 |
| **CustomizeMapSize / MapWidth / MapHeight** | 自订地图长宽 |
| **AutoEnterIfOnly** | 大地图只有此任务时自动进入 |

## 行为说明

### 随机生成 (`GetMapEditData`)
依 `QuestType` 分流到对应的 GenData：
*   `MixRandom` → `m_QuestMixRandomData.RandomGen` + `BalanceDistanceIteration`
*   `RandomGen` / `SlayTheSpire` / `SlsHorizontal` / `CaveGen` / `TestEvents` → 各自的 RandomGen
*   `Default` → 直接读预存的 `m_MapEditData`
*   `RuinGen` / `TestBattles` → **未实作**（log error）

### 任务目标自动生成 (`GenerateQuestGoals` static)
若任务没设目标，静态方法会依当前大地图阶段的奖励池生成：
*   **主目标**（基于 hash seed）：3 种轮换——装备奖 / 单位技能奖 / 卡牌奖；要求是「击败 Boss」（Boss tag 任务）或「击败 N 个精英」。
*   **次目标**：3 种轮换——灵魂值门槛 / 暗雾等级门槛 / 一般敌击败数；奖励是金钱 / 灵魂值 / 道具。

### 挑战目标附加 (`AppendGameChallengeGoals` static)
把 `RCG_DataService.Ins.m_ChallengeGoalDatas` 的未完成挑战目标标记为 `m_IsChallengeGoal` 并附加到任务目标。

### 通关 (`QuestComplete`)
`Difficulty += m_DetailSetting.m_AddDifficultyAfterPass`：通关此任务后永久提升难度。

### 进度 (`QuestProgress`)
*   `BigMapType.Loop`（旧版 Loop 大地图）→ 直接回 `QuestProgressToComplete`。
*   新版（按 Tag 计数）→ 含任一 `BigMapData.QuestProgressTags` 的 Tag → 回 1，否则回 0。

### 条件检查
*   **`IsPrerequirementMet(selected)`** — 已通关过 → false；`CompleteConditions` 全 AND 通过。
*   **`IsQuestAvailable(selected)`** — 已通关过 → false；`IncompatibleConditions` 全 AND 通过。

## 注意事项

*   **`QuestType` 换掉时记得清空 `MapEditData`**：否则旧随机资料会残留干扰。`RemoveRandomNodes` 可清。
*   **`AutoEnterIfOnly` 只在常驻任务 / 起始祝福场景才有效**：大地图检查只剩此任务时自动进入。
*   **目标自动生成是 deterministic**：用 quest IDs 的 hash 算 seed，**重复进入同一组任务会生成相同目标**。
*   **`RuinGen` / `TestBattles` 未实作**：选了会 LogError 而没有实际生成。
*   **`m_FieldEffects` 元素是 `ToggleableFieldEffectGenData`**：可关闭，runtime 走 `FieldEffects` property 过滤出 enabled 的。
*   **`Export Quest Info` 按钮**：汇出怪物配置 Markdown 到 `Assets/Export/Quest/<ID>.md`，方便外部审查战斗配置。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_QuestData.cs`
*   **继承自**：`RCG_Asset<RCG_QuestData>`
*   **实作介面**：`UCL.Core.UI.UCLI_FieldOnGUI`
*   **AssetGroup**：`EditQuestSetting`
*   **CurQuest** (static) — 最近建构的 RCG_QuestData，方便外部存取

### A.2 栏位对照（节选）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` / `m_QuestObjective` | Name / Objective | `RCG_LocalizeData` | |
| `m_QuestIcon` / `m_BGM` / `m_Map` | 图 / BGM / 地图 | `RCG_SpriteData` / `RCG_BGMGenData` / `RCG_PrefabResData` | |
| `m_QuestGoals` | QuestGoals | `List<RCG_QuestGoalData>` | |
| `m_FieldEffects` | FieldEffects | `List<ToggleableFieldEffectGenData>` | |
| `m_Tags` | Tags | `List<RCG_EventTagGenData>` | |
| `m_StoryPool` | StoryPool | `RCG_StoryDropPoolGenData` | |
| `m_MapEditData` | MapEditData | `MapEditData` | `[UCL_HideOnGUI]` |
| `m_QuestBattles` | QuestBattles | `Dictionary<RCG_EnemyTypeTagGenData, RCG_BattleSetDropPoolGenData>` | |
| `m_CardDropPools / m_ItemDropPools / m_EquipmentDropPools` | 各奖励池 | `Dictionary<EnemyType, *DropPoolGenData>` | |
| `m_BattleScene` | BattleScene | `RCG_BattleSceneGenData` | |
| `m_QuestType` | QuestType | `QuestType` enum | 8 种模式 |
| `m_QuestRandomGenData / m_QuestMixRandomData / m_QuestCaveGenData / m_SlayTheSpireGenData / ...` | 各 GenData | 各自类别 | `Conditional(对应 QuestType)` |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | |

### A.3 重要 Method 摘要

*   **`GetMapEditData(isLoadMap)`** — 主入口；按 QuestType 随机生成或读预存。
*   **`QuestComplete()`** — 通关后 Difficulty +=。
*   **`GenerateQuestGoals` (static)** — 自动产主 + 次目标。
*   **`AppendGameChallengeGoals` (static)** — 附加挑战目标。
*   **`GetCardDropPool / GetItemDropPool / GetEquipmentDropPool / GetBattleSet`** — 按敌人类型取对应池；缺则 fallback。
*   **`IsPrerequirementMet / IsQuestAvailable`** — 显示条件检查。
*   **`QuestProgress` (property)** — 新旧两种计算方式。
*   **`SaveGame / LoadGame`** — 随机生成的 MapEditData 序列化。
*   **`ExportQuestInfoToJson`** — 汇出怪物资讯 Markdown 到 `Assets/Export/Quest/`。
*   **`OnGUI(field, dataDic, params)`** — 自订绘制（PreviewTexture + Export 按钮）。

### A.4 与其他系统的互动

*   **`MapEditData`** — 节点 / 路径容器。
*   **`RCG_QuestDropPool`** — Quest 的随机池。
*   **`RCG_BigMapData.QuestProgressTags`** — 进度判断依据。
*   **`RCG_QuestRandomGenData / RCG_QuestMixRandomData / RCG_QuestCaveGenData / RCG_SlayTheSpireGenData / ...`** — 各 QuestType 的生成器。
*   **`RCG_QuestGoalData / QuestGoalRequirementData / GoalReward*`** — 目标 / 奖励系统。
*   **`RCG_DataService.Ins.m_ChallengeGoalDatas`** — 挑战目标来源。
*   **`UI.RCG_QuestEditorUI`** — 节点编辑器。

### A.5 已知议题

*   `RuinGen` / `TestBattles` 未实作，选了只 LogError。
*   `GenerateQuestGoals` 用 hash seed，缺乏随机种子重置机制——同一组 IDs 永远生同样目标。
*   `OnGUI` 内旧版图形绘制代码（GL.Begin / GL.LINES）大段注解，现改用 PreviewTexture。
*   多处 `// QWQ` / `// QWQ23` 注解标示待重构与旧版迁移点。
