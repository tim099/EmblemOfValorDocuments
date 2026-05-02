---
title: 计数器效果 说明
description: 修改触发来源（装备 / 单位技能）的内部计数器数值
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 计数器效果

> 程式类别名称：`RCG_CounterAlterSetting`

## 用途
**修改触发来源的计数器**。「计数器」是装备或单位技能上的内部数值（例如「使用次数」「累积层数」）。常见用途：
*   「装备：每受到 3 次伤害触发一次」 — 计数器在累积与重置间切换
*   「装备：每次战斗可用 X 次」 — 用计数器追踪剩余次数

> [!IMPORTANT]
> 此设定**只能在装备或单位技能的触发效果中使用** — 它依赖 `iData.EffectTriggerSource` 是 `RCGI_EffectCounter` 才有效。一般卡牌触发**无计数器可改**。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CounterEffectType** | 是 | 计数器所属类型：`UnitSkill`（单位技能）或 `Equipment`（装备）。 |
| **CounterId** | 是 | 计数器索引（同一装备可有多个计数器，0、1、2...）。 |
| **CounterAlter** | 是 | 变化量（变数可变）。 |
| **CounterAlterType** | 是 | 变化模式：`Add` / `Sub` / `Set`。 |

## 行为说明
*   读取 `iData.EffectTriggerSource`，确认其为 `RCGI_EffectCounter`，呼叫 `counter.AlterCounter(CounterId, CounterAlterType, CounterAlter)`。
*   描述格式：「**{CounterEffectType} 计数器 {Id+1} {CounterAlterType} {CounterAlter}**」（i18n key `CounterAlterDes`）。

## 注意事项
*   **EffectTriggerSource 不对会错误讯息**：在卡牌效果中触发此设定，会在 console 印错误但不会 crash。**请确认上层是装备或技能效果**。
*   **永远展开**：此设定有 `[AlwaysExpendOnGUI]`，Inspector 中**永远不会折叠**。
*   **CounterId 越界**：超出该装备拥有的计数器数量会触发未定义行为，请确认 ID 与装备配置一致。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterAlterSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 标记
*   **i18n 类别名 key**：`RCG_CounterAlterSetting` → 「计数器效果」
*   **同档案 enum**：`CounterEffectType { UnitSkill, Equipment }` / `CounterAlterType { Add, Sub, Set }`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CounterEffectType` | `CounterEffectType` | enum | — | |
| `m_CounterAlterType` | `CounterAlterType` | enum | — | |
| `m_CounterId` | `CounterId` | `int` | — | 预设 0 |
| `m_CounterAlter` | `CounterAlter` | `IntVariable` | — | |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：取 `iData.EffectTriggerSource`，is-check 为 `RCGI_EffectCounter`，呼叫 `counter.AlterCounter(m_CounterId, m_CounterAlterType, m_CounterAlter.GetValue(iData))`。
*   **`GetDescriptionShort`** → i18n key `RCG_CounterAlterSetting`（=「计数器效果」）。
*   旧逻辑（`m_TriggerData.m_Equipment / m_UnitSkill` 分流）已注解。

### A.4 与其他系统的互动
*   **`RCGI_EffectCounter`**：计数器介面，由装备 / 单位技能实作。
*   **`iData.EffectTriggerSource`**：触发来源；装备触发效果时会被填上对应实例。
