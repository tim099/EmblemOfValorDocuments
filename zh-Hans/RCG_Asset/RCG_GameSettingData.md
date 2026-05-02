---
title: 游戏设定 (RCG_GameSettingData) 说明
description: 不同版本（Demo / 正式版）的游戏级设定：主选单、最大解锁等级、教学重置、商店刷新成本
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 游戏设定

> 程式类别名称：`RCG_GameSettingData`

## 用途

**版本级的游戏设定**——让不同 build（Demo / 正式版 / 展览版）能套不同设定。系统会用 `Application.version` 当作 ID 从中找对应 Asset；如果没有同 ID 的，fallback 到 `Default`。

继承自 `RCG_Asset<RCG_GameSettingData>`。

## 编辑器中的样貌

```
RCG_GameSettingData: <ID = Application.version>
    GameVersion          ← Default / Demo
    MainMenu             ← 主选单设定引用
    MaxUnlockLevel       ← 最大解锁等级上限
    ResetTutorialOnNewGame ← 是否每次开新游戏重置教学
    RefreshBlessingShopPrice ← 祝福商店刷新成本基数
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **GameVersion** | 是 | `Default`（普通版）或 `Demo`（锁定部分功能 / 解锁等级） |
| **MainMenu** | 是 | 主选单设定的引用（`RCG_MainMenuEntry`） |
| **MaxUnlockLevel** | 是 | 最大解锁等级；超过后等级不再成长（预设 1000） |
| **ResetTutorialOnNewGame** | — | 每次开新游戏时重置教学，**展览机**用 |
| **RefreshBlessingShopPrice** | 是 | 刷新祝福商店成本基数；总成本 = (剩余可购买物品数 - 4) × 此值（预设 10） |

## 行为说明

### Asset 取得 (`Ins`)
`RCG_GameSettingData.Ins` 用 `Application.version` 当 ID 从 Asset 库取对应实例。**因此 Asset 命名要符合 build 版本号**（如 `0.1.0` / `0.2.0-demo`）；若没精确匹配，`Util.GetData(version, useDefaultIfMissing = true)` 会 fallback 到 `Default`。

### Demo 版限制
`GameVersion = Demo` 时游戏逻辑会自动锁定一些功能（例如：强制 `MaxUnlockLevel` 为较低值、禁用某些章节）。具体锁定逻辑散落在各 manager；本档只是个 flag。

## 注意事项

*   **ID 必须对应 Application.version**：建版号改了，旧的 Asset 不会自动套用（需新增同名 Asset 或让 Default 涵盖）。
*   **`m_GameSetting` 已被注解**：原本 `RCG_GameInitData.m_GameSetting` 引用此 Asset，现在改用 `Application.version` 自动查找。
*   **`RefreshBlessingShopPrice` 公式**：含负数 case（剩余 ≤ 4 时成本为 0 或负数），实际价格逻辑需 clamp 到 ≥ 0。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameSettingData.cs`
*   **继承自**：`RCG_Asset<RCG_GameSettingData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `GameVersion` enum | `Default` / `Demo` |
| `m_MainMenu` | MainMenu | `RCG_MainMenuEntry` | |
| `m_MaxUnlockLevel` | MaxUnlockLevel | `int` | 预设 1000 |
| `m_ResetTutorialOnNewGame` | ResetTutorialOnNewGame | `bool` | 预设 false |
| `m_RefreshBlessingShopPrice` | RefreshBlessingShopPrice | `int` | 预设 10 |

### A.3 重要 Method

*   **`Ins` (static)** — `Util.GetData(Application.version, useDefaultIfMissing = true)`。

### A.4 与其他系统的互动

*   **`RCG_MainMenuData`** — `m_MainMenu` 引用的主选单设定。
*   **`RCG_GameInitData`** — 游戏初始资料；旧版 `m_GameSetting` 栏位已注解，现由 `Application.version` 自动关联。
*   **`RCG_GameSettingGenData`** — Asset Entry；预设 ID = `"Default"`。

### A.5 已知议题

*   `Ins` 中的 `Debug.Log("Application.version: ...")` 已标记 "log!!"——应该是 dev 用，release 前要移除。
