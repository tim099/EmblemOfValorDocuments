---
title: 游戏标签 说明
description: 触发游戏层级的标签效果（成就、玩家事件等）；薄包装层
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 游戏标签

> 程式类别名称：`RCG_GameTagBattleSetting`

## 用途
**在战斗中触发游戏层级的标签事件** — 是 `RCG_GameTagSetting` 的薄包装层，将其行为转接到战斗动作伫列。常见用途：
*   触发成就条件（例如「击败第一个 Boss」）
*   标记玩家战斗行为纪录

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **GameTagSetting** | 是 | 嵌套的 `RCG_GameTagSetting` — 实际的游戏标签设定。 |

## 行为说明
*   触发时呼叫 `m_GameTagSetting.Trigger(iData)`。
*   描述、卡片资讯全部代理到 `m_GameTagSetting`。

## 注意事项
*   **此设定本身只是包装**：实际逻辑都在 `RCG_GameTagSetting` — 配置栏位请查该类别说明。
*   **不影响战斗数值**：纯粹是事件回报 / 纪录类效果。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_GameTagBattleSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_GameTagBattleSetting` → 「游戏标签」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_GameTagSetting` | `GameTagSetting` | `RCG_GameTagSetting` | — | 嵌套；包装层的唯一栏位 |

### A.3 重要 Method 摘要
*   **`AddAction`**：`m_GameTagSetting.Trigger(iData)`。
*   **`Info / GetDescriptionParams / GetDescriptionFormat`**：全部代理到 `m_GameTagSetting`。

### A.4 与其他系统的互动
*   **`RCG_GameTagSetting`**：实际逻辑所在；包装层只负责 Battle Setting 介面对接。
