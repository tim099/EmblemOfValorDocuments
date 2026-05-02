---
title: 暗雾设定 (RCG_DarkMistSettingsData) 说明
description: 暗雾机制的等级数据：不同暗雾等级下对玩家的弱化效果与对敌方的强化效果
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 暗雾设定

> 程式类别名称：`RCG_DarkMistSettingsData`

## 用途

**暗雾机制的等级数据**。暗雾是随游戏进行不断加深的环境压力，等级越高玩家越弱、敌方越强。本资料定义「在不同暗雾等级下，双方获得什么效果」——通常是以两个共用的 `CustomStatus`（`DarkMistBuff` / `DarkMistDebuff`）为载体，战斗开始时自动套用。

继承自 `RCG_Asset<RCG_DarkMistSettingsData>`。

## 编辑器中的样貌

```
RCG_DarkMistSettingsData: <ID>
    DarkMistBuffGenData      ← 给敌方的强化状态 ID
    DarkMistDebuffGenData    ← 给玩家的弱化状态 ID
    DarkMistDatas            ← 各等级资料清单（DarkMistLevelData）
        TriggerLevel
        DarkMistBuffStatusData    ← 此等级下的强化效果
        DarkMistDebuffStatusData  ← 此等级下的弱化效果
        Effects                   ← 战斗开始时触发的 BattleSetting
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DarkMistBuffGenData** | 是 | 共用的「敌方强化状态」ID（预设 `DarkMistBuff`） |
| **DarkMistDebuffGenData** | 是 | 共用的「玩家弱化状态」ID（预设 `DarkMistDebuff`） |
| **DarkMistDatas** | 是 | 各等级资料清单（按 `TriggerLevel` 排序，**最后元素优先**） |

每个 `DarkMistLevelData` 内含：

| 子栏位 | 说明 |
|---|---|
| **TriggerLevel** | 此等级触发门槛（暗雾等级 ≥ 此值才使用本笔设定） |
| **DarkMistBuffStatusData** | 此等级下的敌方强化效果（覆写到 `DarkMistBuff` Status 上） |
| **DarkMistDebuffStatusData** | 此等级下的玩家弱化效果（覆写到 `DarkMistDebuff` Status 上） |
| **Effects** | 战斗开始时触发的 `RCG_BattleSetting` 序列 |

## 行为说明

### 取得当前等级资料 (`GetCurrentDarkMistLevelData`)
从清单**末端往前**找第一个 `TriggerLevel ≤ iLevel` 的资料；因此清单必须**按 TriggerLevel 升幂排序**才能正确匹配（例如 `[0, 5, 10, 20]`）。

### 触发 (`OnTriggerEffect`)
找到对应等级资料后，把该资料的 `m_Effects` 全部加到动作伫列。

### 覆写状态 (`UpdateDarkMistEffect`)
把当前等级的 `DarkMistBuffStatusData` 写入共用 `DarkMistBuff` Status 的 `m_Effects`——这是**就地修改全局 Status Asset**，每次战斗开始前重设一次。

## 注意事项

*   **DarkMistDatas 必须升幂排序**（依 `TriggerLevel`）：取资料逻辑依赖此前提，逆序或乱序会找错级。
*   **共用 Status 是动态覆写**：`DarkMistBuff` / `DarkMistDebuff` 两个 `RCG_CustomStatusData` 的 effects 会被 runtime 覆写；不要手动修这两个 Asset。
*   **被多个来源使用**：`RCG_GameInitData.m_DarkMistSettings` 是清单，可有多套（例：难度不同套不同暗雾）。
*   **Preview 已被注解掉**：编辑器内预览用基底类预设绘制。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_DarkMistSettingsData.cs`
*   **继承自**：`RCG_Asset<RCG_DarkMistSettingsData>`
*   **AssetGroup**：`EditQuestSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DarkMistBuffGenData` | DarkMistBuffGenData | `RCG_CustomStatusGenData` | 预设 `"DarkMistBuff"` |
| `m_DarkMistDebuffGenData` | DarkMistDebuffGenData | `RCG_CustomStatusGenData` | 预设 `"DarkMistDebuff"` |
| `m_DarkMistDatas` | DarkMistDatas | `List<DarkMistLevelData>` | 各等级资料 |

### A.3 重要 Method

*   **`OnTriggerEffect(data, level)`** — 找对应等级资料 → 套用 effects。
*   **`GetCurrentDarkMistLevelData(level)`** — 从尾端往前查 `TriggerLevel ≤ level` 的元素。
*   **`UpdateDarkMistEffect(level)`** — 覆写共用 Buff Status 的 effects。
*   **`SetDarkMistBuff(status)`** — 用 JSON 序列化复写 Buff 整个 Asset。

### A.4 与其他系统的互动

*   **`RCG_CustomStatusData`** — `DarkMistBuff` / `DarkMistDebuff` 两个全局状态。
*   **`RCG_GameInitData.m_DarkMistSettings`** — 引用此资料的清单。
*   **`RCG_BattleSetting`** — `m_Effects` 的元素型别。

### A.5 已知议题

*   `Preview` 完整实作已注解；改用基底类预设绘制。
*   清单必须手动排序，缺乏自动排序保证；若资料顺序错乱会找错等级。
