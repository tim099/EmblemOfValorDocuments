---
title: 战斗资讯资产 (RCG_BattleInfoAsset) 说明
description: 战斗日志左侧显示的「即时统计资讯」设定（本回合打出手牌数、伤害总量等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 战斗资讯资产

> 程式类别名称：`RCG_BattleInfoAsset`

## 用途

**战斗日志左侧显示的即时统计资讯**。例如「本回合打出手牌：3」「累积伤害：120」「剩余抽卡格：2」。每个统计项目是一个 `RCG_BattleInfoAsset` 实例；战斗 UI 会依 `m_Order` 排序并逐一显示。

继承自 `RCG_Asset<RCG_BattleInfoAsset>`。

## 编辑器中的样貌

```
RCG_BattleInfoAsset: <ID>
    Enable
    InfoType   ▾ IntVariable
    Variable   ← InfoType=IntVariable 时显示
    Order
    HideIfZero
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Enable** | 是 | 是否启用此资讯（false 直接从显示中拿掉） |
| **InfoType** | 是 | 资讯类型，目前只有 `IntVariable` |
| **Variable** | InfoType=IntVariable | 要显示的整数变数（`IntVariable`），可绑定战斗计数器 |
| **Order** | 是 | 显示顺序，越小越前面 |
| **HideIfZero** | — | 为 0 时隐藏（避免一堆 0 洗版） |

## 行为说明

### `ShowInfo(data)`
*   `m_HideIfZero = true` 且 `Variable.GetValue() == 0` → 不显示。
*   否则显示。

### `GetInfo(data)`
依 `InfoType` 取得字串：
*   `IntVariable` → `m_Variable.GetDescription(iData)`（已格式化，含标签前缀等）。

## 注意事项

*   **`InfoType` 目前只支援 `IntVariable`**：未来可能新增 String / Float 类型，但目前 enum 只有一个值。
*   **`m_Order` 冲突时**：两个 Asset 同 Order 的显示顺序不保证；建议手动分开值。
*   **HideIfZero 对非 IntVariable 无效**：逻辑目前只 cover IntVariable case。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleInfoAsset.cs`
*   **继承自**：`RCG_Asset<RCG_BattleInfoAsset>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Enable` | Enable | `bool` | 预设 `true` |
| `m_InfoType` | InfoType | `InfoType` enum | 目前只有 `IntVariable` |
| `m_Variable` | Variable | `IntVariable` | `Conditional(IntVariable)` |
| `m_Order` | Order | `int` | 预设 0；越小越前 |
| `m_HideIfZero` | HideIfZero | `bool` | 预设 `false` |

### A.3 重要 Method

*   **`ShowInfo(TriggerEffectData)`** — 是否要显示；依 `m_HideIfZero` 与 Variable 值判断。
*   **`GetInfo(TriggerEffectData)`** — 取格式化后的字串。

### A.4 与其他系统的互动

*   **`IntVariable`** — 变数绑定来源。
*   **战斗日志左侧 UI** — 显示这些 Asset 的消费端。

### A.5 已知议题

*   `InfoType` enum 只有一个值，预示未来扩充计划但尚未实作。
