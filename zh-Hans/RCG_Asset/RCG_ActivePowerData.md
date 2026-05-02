---
title: 主动能力资料 (RCG_ActivePowerData) 说明
description: 角色身上的「主动能力」：可手动触发的能力（类似道具但绑定角色而非背包）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 主动能力资料

> 程式类别名称：`RCG_ActivePowerData`

## 用途

**角色身上的「主动能力」模板**。介于「装备」与「道具」之间：玩家可在战斗中**手动触发**的能力（不像被动），但**绑定在特定角色身上**（不像道具放在共用背包）。例如「治疗师：每战可释放一次群体治疗」「剑士：每回合可手动引发暴击」。

继承自 `RCG_Asset<RCG_ActivePowerData>`，实作介面：`RCGI_Item` / `RCGI_Unloackable`。

## 编辑器中的样貌

```
RCG_ActivePowerData: <ID>
    Data (m_Data)               ← 主要设定（Name / Description / Icon / TargetType / Price / Unlock / ItemUseType）
    ItemEffects                 ← 主动能力的效果（OnPlay 触发）
    Preview
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name / Description / Icon** | 是 | 名称、风味描述、图示 |
| **TargetType** | 是 | 使用时的选目标范围 |
| **Price** | 是 | 商店价（如果可购买） |
| **ItemUseType** | 是 | `Consume` / `OncePerBattle` / `OncePerTurn` / `Infinite` |
| **UseTimes** | 视 ItemUseType | 可使用次数 |
| **Unlock** | 否 | 解锁条件 |
| **ItemEffects** | 否 | 触发效果（`RCG_CommonEffect` list） |

## 行为说明

### 与道具的差异
*   **道具**：放共用背包，任何角色都能用。
*   **主动能力**：绑定在特定角色身上（`RCG_DataService.Ins.m_ActivePowersData`），通常随角色加入队伍而带来。

### 使用 / 触发
`CheckUsable(data)` → 所有 effect 的 `CheckPlayable` 全 true 才能用。
`TriggerEffect(data)` → 取 `OnPlay` effects 依序触发（含 try-catch）。

### 描述生成
与 `RCG_ItemData` 类似：自动由 effects 串接 + UseType 标记 + UseTimes 显示。

## 注意事项

*   **`ShowInfo()` 是空壳**：原本要走 `RCG_ItemInfoPanel`，但被注解掉（`// QWQ`），目前无作用。实际 UI 由其他位置显示。
*   **没有 ItemType 栏位**（与 `RCG_ItemData` 不同）：因为已是「主动能力」分类，不再细分 Normal / QuestItem。
*   **`InitActivePowers` 的旧注解**：原本角色加入时会自动套用 InitActivePowers，但已转用 UnitSkill 系统取代（程式内有大段注解标示）。
*   **无 Auto Price 工具**：与 ItemData / EquipmentData 不同，编辑器页面没做这个。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ActivePowerData.cs`
*   **继承自**：`RCG_Asset<RCG_ActivePowerData>`
*   **实作介面**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`

### A.2 栏位对照（外层）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Data` | Data | `ActivePowerData`（巢状） | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ActivePowerData` 巢状含 `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock`。

### A.3 重要 Method 摘要

*   **`AddItem()`** — `RCG_ActivePower.Create(ID)` → `m_ActivePowersData.AddActivePower`。
*   **`CheckUsable(TriggerEffectData)`** — 全 effect AND check。
*   **`TriggerEffect(TriggerEffectData)`** — `OnPlay` effects 触发。
*   **`Description` / `FullDescription` / `GetDescription`** — 描述生成（`RCG_ActivePower` 可选参数带 runtime 剩余次数）。
*   **`UseTime`** — 依 `ItemUseType` 回传次数（Infinite → 0）。

### A.4 与其他系统的互动

*   **`RCG_ActivePower`** — runtime 主动能力实例。
*   **`RCG_DataService.Ins.m_ActivePowersData`** — 角色绑定储存。
*   **`RCG_CommonEffect`** — 触发效果单位。

### A.5 已知议题

*   `ShowInfo()` 已被注解（`// QWQ`），UI 入口需确认。
*   旧版 `InitActivePowers` 自动加成逻辑已弃用（被 UnitSkill 取代）。
