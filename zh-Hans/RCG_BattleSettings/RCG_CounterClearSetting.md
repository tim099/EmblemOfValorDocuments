---
title: 计数器清除 说明
description: 一次清空触发来源（装备 / 单位技能）的所有计数器
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 计数器清除

> 程式类别名称：`RCG_CounterClearSetting`

## 用途
**一次清空触发来源的所有计数器**。比「计数器效果」更强硬：直接归零**全部**而非单个。常见用途：
*   「装备能力触发后重置所有累积层」
*   「特定条件下强制清掉装备充能」

> [!IMPORTANT]
> 同样**只能在装备或单位技能的触发效果中使用**（依赖 `EffectTriggerSource` 是 `RCGI_EffectCounter`）。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CounterEffectType** | 是 | 计数器所属类型：`UnitSkill` 或 `Equipment`。**目前仅作为描述显示用**，实际清除是针对 EffectTriggerSource 全部计数器，不挑类型。 |

## 行为说明
*   呼叫 `counter.ClearAllCounters()` — 清除该触发来源的**全部**计数器。
*   描述为「**清除 {CounterEffectType} 计数器**」（i18n key `CounterClearDes`）。

## 注意事项
*   **没有 CounterId 栏位**：清除是「全部」性质，不能挑特定计数器。要清单一计数器请改用「**计数器效果**」+ `Set 0`。
*   **EffectTriggerSource 不对的错误**：与「计数器效果」相同，console 印错但不 crash。
*   **永远展开**：`[AlwaysExpendOnGUI]`。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterClearSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 标记
*   **i18n 类别名 key**：`RCG_CounterClearSetting` → 「计数器清除」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CounterEffectType` | `CounterEffectType` | enum | — | 与 `RCG_CounterAlterSetting` 共用 |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：取 `iData.EffectTriggerSource`，is-check `RCGI_EffectCounter`，`counter.ClearAllCounters()`。
*   **`GetDescriptionFormat / GetDescriptionShort`**：均为 i18n key `CounterClearDes`，含 `m_CounterEffectType.GetLocalizeName()`。

### A.4 与其他系统的互动
*   **`RCGI_EffectCounter.ClearAllCounters`**：计数器全清入口。
