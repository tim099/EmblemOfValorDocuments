---
title: 统计资料资产 (RCG_StatsAsset) 说明
description: 对应「游戏内统计值」与 Steam 统计的桥梁：击杀数、伤害总量等可累加数值
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 统计资料资产

> 程式类别名称：`RCG_StatsAsset`

## 用途

**游戏内统计值的定义**。例如「累计击杀数」「累计打出卡牌数」「累计通关次数」这种**可累加的历史数值**，用 `RCG_StatsAsset` 定义；可选择是否串接 Steam Stats 自动上报。

继承自 `RCG_Asset<RCG_StatsAsset>`。

## 编辑器中的样貌

```
RCG_StatsAsset: <ID>
    HasSteamStat         ← 是否串接 Steam 统计
    SteamUserStat        (HasSteamStat=true) ← 对应的 Steam 统计 ID
    GameStatsType        ← 对应的游戏内统计枚举值（None / KillCount / ... 等）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **HasSteamStat** | — | 是否串接 Steam 统计 |
| **SteamUserStat** | HasSteamStat=true | 对应的 Steam 统计 entry |
| **GameStatsType** | 是 | 游戏内统计类型（`GameStatsType` enum） |

## 行为说明

### Editor 内 `Create BuiltIn Stats` 工具
编辑器页面上方有按钮：对 `GameStatsType` 列举内所有非 `None` 值，自动建立缺漏的 Asset（ID 用 enum 字串名）并把 `m_GameStatsType` 设好。**便于同步程式端 enum 与 Asset 端**。

## 注意事项

*   **`GameStatsType` enum 是程式控制**：新增统计类型要先改 enum，再用编辑器工具补 Asset。
*   **预设 ID `None`**（`RCG_StatsEntry.DefaultID`）：表示「无统计」；不要拿来作为实际统计的 ID。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StatsAsset.cs`
*   **继承自**：`RCG_Asset<RCG_StatsAsset>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_HasSteamStat` | HasSteamStat | `bool` | |
| `m_SteamUserStat` | SteamUserStat | `UCL_SteamUserStatAssetEntry` | `Conditional(HasSteamStat)` |
| `m_GameStatsType` | GameStatsType | `GameStatsType` enum | |

### A.3 重要 Method / 工具

*   **`CreateSelectAssetPage`** — `RCG_StatsAssetEditorPage.Create()`，编辑器分页。
*   **`Create BuiltIn Stats`**（Editor only） — 自动补齐缺漏的 stats Asset。

### A.4 与其他系统的互动

*   **`UCL_SteamUserStatAssetEntry`** — Steam SDK 串接。
*   **`GameStatsType` (enum)** — 程式端统计类型枚举。
*   **`RCG_StatsEntry`** — Asset Entry；预设 `None`。
