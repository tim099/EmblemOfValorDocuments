---
title: 资源类型 (RCG_ResourceTypeData) 说明
description: 游戏内各种资源（金币 / 补给 / 灵魂 / 祝福）的定义：名称、描述、图示
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 资源类型

> 程式类别名称：`RCG_ResourceTypeData`

## 用途

**游戏内可累积资源类型的定义**。Gold（金币）、Supply（补给 / 暗雾减少道具）、Spirit（灵魂）、Blessing（祝福）等都是不同 ID 的 `RCG_ResourceTypeData`。本资料定义它们的显示名、描述、图示。

继承自 `RCG_Asset<RCG_ResourceTypeData>`。

## 编辑器中的样貌

```
RCG_ResourceTypeData: <ID>
    Name(多国语言)
    Description(多国语言)
    IconSprite      ← 对应的 RCG_IconSprite 引用
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 资源名（多语系） |
| **Description** | 否 | 资源描述（用于 tooltip） |
| **IconSprite** | 是 | 图示引用（`RCG_IconSpriteGenData`） |

## 行为说明

### LocalizeName 切换
*   `RCG_BattleSetting.IsShowOnUI = false` → 回 `m_Name.Name`（纯文字名）。
*   `IsShowOnUI = true` → 回 `m_IconSprite.TMPKey`（给 TextMeshPro 用的 sprite tag）。

### 预设四种资源（程式内常数）
*   `Gold` (`ResourceType_Gold`)
*   `Supply` (`ResourceType_Supply`) — 也对应到「暗雾」相关用途
*   `Spirit` (`ResourceType_Spirit`)
*   `Blessing` (`ResourceType_Blessing`)

## 注意事项

*   **ID 命名规则**：`ResourceType_<EnumName>`（例如 `ResourceType_Gold`）。
*   **`enum ResourceType` (档内)** 只列出三种（Gold / Supply / Spirit）；Blessing 直接用字串而没进 enum。
*   **与 `RCG_DifficultyData.m_PriceMult / m_SoulPriceMult`** 等难度倍率的对应：价格倍率影响 Gold / Spirit 两种资源的商店价。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ResourceTypeData.cs`
*   **继承自**：`RCG_Asset<RCG_ResourceTypeData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |

### A.3 重要 Property

*   **`LocalizeName`** — UI 模式回 TMPKey；其他回 Name。
*   **`Description`** — `m_Description.Name`。
*   **`Icon`** — `m_IconSprite.GetData().IconSprite`。

### A.4 与其他系统的互动

*   **`RCG_ResourceTypeGenData`** — Asset Entry；含 4 个 static 预设（Gold / Supply / Spirit / Blessing）。
*   **`RCG_IconSpriteGenData`** — 图示资源引用。
*   **`RCG_DataService.Ins.GetResource(...)`** — runtime 取资源量的入口。
*   **`RCG_BattleSetting.IsShowOnUI`** — 控制 LocalizeName 显示模式。
