---
title: 战斗预设组合 (RCG_BattlePresetData) 说明
description: 用来快速启动测试战斗的「全套配置」：怪物 + 玩家角色 + 牌组 + 装备 + 难度，一键 StartBattle
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗预设组合

> 程式类别名称：`RCG_BattlePresetData`

## 用途

**测试战斗用的快速配置**。把一整套「怪物 + 玩家角色 + 牌组 + 装备 + 技能 + 难度」打包成一个 Asset，按下 Editor 内的 `StartBattle` 按钮即可立刻进入战斗（跳过大地图、角色选择等流程）。**不是正式游戏流程使用**，纯粹是 QA / 设计师调整数值的调试工具。

继承自 `RCG_Asset<RCG_BattlePresetData>`。

## 编辑器中的样貌

```
RCG_BattlePresetData: <ID>
    BattleSetGenData   ← 引用的战斗组合
    Players            ← 出战角色清单
    ExtraCards         ← 起始额外卡（除了 Deck 之外）
    Items              ← 起始道具
    Deck               ← 起始牌组（替换预设）
    Equipments         ← 各角色穿戴的装备（dictionary）
    UnitSkills         ← 各角色已学的技能（dictionary）
    AllEquipments      ← 没装备到身上的装备（背包）
    EnemyType          ← 敌人类型（普通 / 精英 / Boss）
    DifficultyData     ← 全局难度资料
    Difficulty         ← 动态难度值
    [按钮] StartBattle ← Editor playing 时可一键开战
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **BattleSetGenData** | 是 | 引用的 `RCG_BattleSet`（怪物配置） |
| **Players** | 是 | 出战角色清单 |
| **ExtraCards** | 否 | 额外塞进牌组的卡（用于测试特定卡组合） |
| **Items** | 否 | 起始道具（药水、卷轴） |
| **Deck** | 否 | 起始牌组；非 `Default` 时会覆盖玩家牌组 |
| **Equipments** | 否 | 角色 → 装备清单的 dictionary（哪个角色穿哪些装备） |
| **UnitSkills** | 否 | 角色 → 技能清单的 dictionary |
| **AllEquipments** | 否 | 不装备到身上、放背包的装备 |
| **EnemyType** | 否 | 敌人类型 |
| **DifficultyData** | 否 | 全局难度设定（HP / Atk 倍率、敌人技能等级） |
| **Difficulty** | 否 | 动态难度值（叠加在大地图难度之上） |

## 行为说明

### `StartBattleAsync`
按 `StartBattle` 按钮（仅 Editor playing 时显示）会：
1. 建立 BigMapManager 并进入大地图。
2. 进入 Quest（包成 EnterQuestSetting）。
3. 套用 `m_DifficultyData` 与 `m_Difficulty` 到 `RCG_DataService`。
4. 替换玩家牌组（如果 `m_Deck.ID != Default`）。
5. 加入 ExtraCards / Items / Equipments / UnitSkills 等。
6. 进入战斗场景。

### 预览
显示底层 BattleSet 的 Preview，并提供 StartBattle 按钮。

## 注意事项

*   **仅供测试**：正式游戏不从这里进入战斗；不要依赖此资料设计正式关卡。
*   **`m_Deck.ID == DefaultID` 时不替换牌组**，会用玩家当前牌组 — 确认此 Asset 的 `m_Deck` 有指定才会生效。
*   **InitActivePowers 逻辑已注解**：原本会套用角色初始主动能力，目前由 UnitSkill 系统取代，注解掉了。
*   **Editor not playing 时无法看到 StartBattle 按钮**——必须在 Play Mode 下开 Developer 页面才能用。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattlePresetData.cs`
*   **继承自**：`RCG_Asset<RCG_BattlePresetData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_BattleSetGenData` | BattleSet | `RCG_BattleSetGenData` | |
| `m_Players` | Players | `List<RCG_CharacterGenData>` | |
| `m_ExtraCards` | ExtraCards | `List<RCG_CardGenData>` | |
| `m_Items` | Items | `List<RCG_ItemGenData>` | |
| `m_Deck` | Deck | `RCG_DeckGenData` | |
| `m_Equipments` | Equipments | `Dictionary<RCG_CharacterGenData, List<RCG_EquipmentGenData>>` | |
| `m_UnitSkills` | UnitSkills | `Dictionary<RCG_CharacterGenData, List<RCG_UnitSkillGenData>>` | |
| `m_AllEquipments` | AllEquipments | `List<RCG_EquipmentGenData>` | 背包 |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | |
| `m_DifficultyData` | DifficultyData | `RCG_DifficultyData` | |
| `m_Difficulty` | Difficulty | `int` | |

### A.3 重要 Method 摘要

*   **`StartBattleAsync(BigMap, Quest, CancellationToken)`** — 主入口；建大地图 → 进 Quest → 套难度 → 替换 Deck → 加 Cards/Items/Equipments/UnitSkills。
*   **`Preview`** — 编辑器内显示 BattleSet preview + StartBattle 按钮。

### A.4 与其他系统的互动

*   **`RCG_BigMapManager`** / **`RCG_MapManager`** — 进入流程的核心。
*   **`RCG_DataService.Ins.m_DifficultyData / Difficulty`** — 套用难度。
*   **`RCG_DataService.Ins.m_DeckData`** — 替换牌组目标。
*   **`RCG_MapEventManager.Reset`** — 进入新战斗前清除事件伫列。

### A.5 已知议题

*   `// 初始角色能力` 一段已被注解，旧版透过 ActivePower 系统灌入初始能力的逻辑废弃。
