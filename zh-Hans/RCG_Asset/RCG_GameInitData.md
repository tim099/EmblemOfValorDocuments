---
title: 游戏初始资料 (RCG_GameInitData) 说明
description: 游戏全局单例设定：起始道具/装备/技能、卡牌系统设定、Pooling、编辑器设定、暗雾
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 游戏初始资料

> 程式类别名称：`RCG_GameInitData`

## 用途

**游戏的全局唯一设定资料**（单例 Asset，ID 固定为 `Default`）。包含：
*   起始物品 / 装备 / 主动能力 / 资源
*   卡牌系统设定（`RCG_CardGameSetting`）
*   Pooling 设定（物件池）
*   Prefab 资源设定
*   编辑器设定
*   暗雾设定清单
*   装备类型对应的图示

继承自 `RCG_Asset<RCG_GameInitData>`。

## 编辑器中的样貌

```
RCG_GameInitData: Default
    GameVersion / DefaultLanguage
    CardGameSetting       ← 卡牌系统的全局设定
    PrefabResSetting      ← 通用 prefab 路径
    PoolingGenDataSetting ← 物件池设定
    DarkMistSettings      ← 暗雾设定清单
    EditorSetting
    InitItems / InitActivePowers / InitEquipments / InitResources
    EquipmentTypeIcon     ← 装备类型 → 图示对应
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **GameVersion** | 是 | 游戏版本（`RCG_GameVersionGenData`） |
| **DefaultLanguage** | 是 | 预设语系 |
| **CardGameSetting** | 是 | 卡牌系统设定（手牌数、抽牌规则等） |
| **PrefabResSetting** | 是 | 通用 prefab 路径设定 |
| **PoolingGenDataSetting** | 是 | 物件池设定（哪些 prefab 要 pool / 初始量） |
| **DarkMistSettings** | 否 | 暗雾设定清单（多个 `RCG_DarkMistSettingsData` 引用） |
| **EditorSetting** | 否 | 编辑器相关设定 |
| **InitItems / InitActivePowers / InitEquipments / InitResources** | 否 | 开新游戏时的起始物品 / 主动能力 / 装备 / 资源 |
| **EquipmentTypeIcon** | 否 | `EquipmentType → RCG_SpriteData` 对应，UI 显示用 |

## 行为说明

### `GameInit()`
开新游戏时呼叫，把 `m_InitItems` / `m_InitEquipments` / `m_InitActivePowers` 全部加到玩家身上（个别 Asset 自带 `AddItem` 逻辑）。

### `GetEquipmentIcon(EquipmentType)`
查 `m_EquipmentTypeIcon` 字典；找不到对应 key 时 LogError 并 fallback 到第一个 entry。

### Static 入口
*   `RCG_GameInitData.Ins` — 取 ID `Default` 的 Asset。
*   `CreateInstance()` — 从 Asset 重新生成一份起始资料（`useDefaultIfMissing = false`）。

## 注意事项

*   **ID 固定为 `Default`**：本类别预期只有一个 Asset。新增其他 ID 不会被系统自动读取（除非透过 `Util.GetData` 手动呼叫）。
*   **`m_GameSetting` 已被注解**：曾规划让 GameInitData 引用 GameSettingData，现在改由 `Application.version` 自动关联。
*   **`m_UnlockSetting` 已被注解**：解锁设定改由 `RCG_UnlockData` 处理。
*   **`m_InitEquipments` 标 TODO**：「准备移到角色设定资料中 不同角色自带不同装备」——未来可能搬到 `RCG_CharacterData`。
*   **装备类型图示找不到时的 fallback**：用第一个 entry，可能不是想要的；漏设会看到错误的图示。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameInitDatas/RCG_GameInitData.cs`
*   **继承自**：`RCG_Asset<RCG_GameInitData>`
*   **AssetGroup**：`EditGameSetting`
*   **常数**：`DefaultID = "Default"`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `RCG_GameVersionGenData` | |
| `m_DefaultLanguage` | DefaultLanguage | `UCL_LanguageCodeEntry` | |
| `m_CardGameSetting` | CardGameSetting | `RCG_CardGameSetting` | |
| `m_PrefabResSetting` | PrefabResSetting | `RCG_PrefabResSetting` | |
| `m_PoolingGenDataSetting` | PoolingGenDataSetting | `RCG_PoolingGenDataSetting` | |
| `m_DarkMistSettings` | DarkMistSettings | `List<RCG_DarkMistSettingsGenData>` | |
| `m_EditorSetting` | EditorSetting | `RCG_EditorSetting` | |
| `m_InitItems` | InitItems | `List<RCG_ItemGenData>` | |
| `m_InitActivePowers` | InitActivePowers | `List<RCG_ActivePowerGenData>` | |
| `m_InitEquipments` | InitEquipments | `List<RCG_EquipmentGenData>` | |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_EquipmentTypeIcon` | EquipmentTypeIcon | `Dictionary<EquipmentType, RCG_SpriteData>` | |

### A.3 重要 Method 摘要

*   **`Ins` / `CreateInstance` (static)** — 两种取得入口（前者用 cache，后者强制重读）。
*   **`GameInit()`** — 开新游戏时加入起始物品 / 装备 / 能力。
*   **`GetEquipmentIcon(type)`** — 图示查询；找不到 fallback 第一个。
*   **`CardGameSetting / PoolingGenDataSetting / PrefabResSetting` (static properties)** — 快捷存取。

### A.4 与其他系统的互动

*   **`RCG_DataService`** — runtime 玩家资料；`InitItems` 等加入此处。
*   **`RCG_DarkMistSettingsData`** — 暗雾设定清单元素。
*   **`RCG_CardGameSetting`** — 卡牌系统设定。
*   **`UCL_LanguageCodeEntry`** — 语系设定。

### A.5 已知议题

*   `m_GameSetting` / `m_UnlockSetting` / `m_CardClassBackgroundImages` 等多处已注解，标示旧版设计改动。
*   `m_InitEquipments` 待搬移到角色资料的 TODO。
