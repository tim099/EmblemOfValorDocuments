---
title: 地图事件资料 (RCG_MapEventData) 说明
description: 小地图节点上可触发的事件包：对话、过场、给予物品、战斗前奏等
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 地图事件资料

> 程式类别名称：`RCG_MapEventData`

## 用途

**小地图节点上可触发的事件包**。例如踩到「神秘节点」会跳出对话 → 选择 → 给予奖励；「篝火」直接给回血选单；「商人」开店。每个 `RCG_MapEventData` 是一组 `RCG_MapEvent` 的容器，触发时依序加入事件管理器。

继承自 `RCG_Asset<RCG_MapEventData>`。

## 编辑器中的样貌

```
RCG_MapEventData: <ID>
    Name                     ← 事件显示名（多语系）
    Icon                     ← 自订图示（空白时用第一个 Event 的预设图示）
    CanTriggerRepeatedly     ← 是否可重复触发
    Events                   ← 实际事件清单（RCG_MapEvent）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 否 | 显示名（多语系）；空白时 fallback 到第一个 Event 的 ShortName |
| **Icon** | 否 | 图示；空白时用第一个 Event 的 `DefaultIcon` |
| **CanTriggerRepeatedly** | — | 是否可重复触发；false → 触发后节点上的事件会清除 |
| **Events** | 是 | 实际的事件清单（`List<RCG_MapEvent>`） |

## 行为说明

### 触发 (`StartEvent(node)`)
逐一 clone 每个 `RCG_MapEvent` 并加入 `RCG_MapEventManager`。Clone 而非直接用是为了让同一个 Asset 多次触发时，每次都是独立 instance。

### 图示与名称的 fallback
*   `Icon`：`m_Icon` 空 → `m_Events[0].DefaultIcon` → null。
*   `LocalizedName`：`m_Name` 有 → 用；否则 → `m_Events[0].GetShortName()`。
*   `EventHoverTips`：`LocalizedName`（滑鼠在节点上时的提示）。

## 注意事项

*   **`CanTriggerRepeatedly = false`** 触发后节点上的事件会清除：适合「一次性事件」如剧情关键点、独特奖励。
*   **Events 为空时**：图示与名称会 fallback 失败（回 null / 空字串），UI 上看起来像没事件。
*   **预设 ID `Start`**：通常作为起点节点专用事件。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MapEventData.cs`
*   **继承自**：`RCG_Asset<RCG_MapEventData>`
*   **AssetGroup**：`EditQuestSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | Addressable 预设 |
| `m_CanTriggerRepeatedly` | CanTriggerRepeatedly | `bool` | 预设 `false` |
| `m_Events` | Events | `List<RCG_MapEvent>` | |

### A.3 重要 Method

*   **`StartEvent(RCG_MapNode)`** — 主触发入口；clone 每个 event 加入 manager。
*   **`Icon` (property)** — 含 fallback 逻辑。
*   **`LocalizedName / EventHoverTips`** — 显示用属性。

### A.4 与其他系统的互动

*   **`RCG_MapEvent`** — 实际事件单位（对话 / 选择 / 奖励 / 战斗前奏）。
*   **`RCG_MapEventManager`** — runtime 事件伫列管理。
*   **`RCG_MapNode`** — 触发来源节点。
*   **`RCG_MapEventGenData`** — Asset Entry；预设 ID = `"Start"`。
