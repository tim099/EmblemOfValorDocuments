---
title: Placeholder（融合占位）说明
description: 卡牌融合系统使用的占位类别；保留巢状结构，融合确认后会被替换为实际效果
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Placeholder（融合占位）

> 程式类别名称：`RCG_PlaceholderSetting`

## 用途
**卡牌融合系统内部使用的占位类别**。在融合预览阶段保留巢状结构（例如「条件判断里有 placeholder」），让玩家看出**哪里会被填入新效果**。融合确认后会被替换为真正的子设定。

> [!IMPORTANT]
> 这个设定**通常不会由你手动建立** — 它由融合系统在执行时自动产生。如果你在资料中看到它，代表这张卡是融合过程中暂存的中介状态。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **PlaceholderKey** | 是 | i18n key，用来显示提示文字（预设 `CardFusionPlaceholder`，会带上 PlaceholderIndex 编号）。 |
| **PlaceholderIndex** | 是 | 占位编号（多个 placeholder 用编号区分）。 |
| **PreviewDescription** | 否 | 自订的预览描述；非空则覆写 i18n 结果。 |

## 行为说明
*   触发时什么也不做（纯占位）。
*   描述优先使用 `PreviewDescription`，否则显示 i18n key `PlaceholderKey` 的结果。
*   作为**叶节点**存在 — `GetFusionCandidateSettings()` 为空（不可作为融合候选），`GetFusionBaseSetting()` 回传自身（不会被进一步替换）。

## 注意事项
*   **不要在资料中手动建立**：除非你想**故意**标记某个位置为「待融合的洞」。
*   **与「DiminishedPlaceholder」的差别**：本类是**融合系统**的中性占位（蓝框）；`RCG_DiminishedPlaceholder` 是**削弱系统**的（红字「已被削弱」）。
*   **PreviewDescription 优先**：UI 显示测试时常用此栏位手动 override 提示文字。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_PlaceholderSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **无 i18n 类别名 key**：编辑器显示 stripped name `Placeholder`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_PlaceholderKey` | `PlaceholderKey` | `string` | — | 预设 `"CardFusionPlaceholder"` |
| `m_PlaceholderIndex` | `PlaceholderIndex` | `int` | — | i18n 模板用 `{0}` 替换 |
| `m_PreviewDescription` | `PreviewDescription` | `string` | — | 预设空字串；非空则覆写描述 |

### A.3 重要 Method 摘要
*   **`GetDescription`**：`m_PreviewDescription` 非空 → 直接返回；否则 `UCL_LocalizeManager.Get(m_PlaceholderKey, m_PlaceholderIndex)`。
*   **`GetDescriptionShort / GetShortName / GetDescriptionFormat`**：均代理到 `GetDescription`。
*   **`GetFusionCandidateSettings`** → 空清单。
*   **`GetFusionBaseSetting`** → `this`。
*   **`AddAction`** → 空实作。

### A.4 与其他系统的互动
*   **融合系统的 `GetFusionBaseSetting`**：父类 `RCG_BattleSetting.GetFusionBaseSetting` 预设回传 `new RCG_PlaceholderSetting()`，所有未覆写的子类在融合预览时都会生成 placeholder。
*   **`RCG_DiminishedPlaceholder`**：继承自此类；削弱系统的特化。
