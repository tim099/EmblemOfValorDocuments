---
title: 道具资料 (RCG_ItemData) 说明
description: 玩家可使用的消耗 / 任务道具模板（药水、卷轴、钥匙等）：稀有度、效果、使用次数、解锁
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 道具资料

> 程式类别名称：`RCG_ItemData`

## 用途

**玩家可持有 / 使用的道具模板**。药水、卷轴、食材、任务钥匙、回复道具⋯ 全都是这个类别的实例。每个道具有自己的图示、效果、稀有度、使用次数限制、解锁条件。

继承自 `RCG_Asset<RCG_ItemData>`，实作介面：`RCGI_Item`（可加入背包）/ `RCGI_Unloackable`（可解锁）。

## 编辑器中的样貌

```
RCG_ItemData: <ID>
    Data (m_Data)              ← 主要设定（巢状类）
    Effects (m_ItemEffects)    ← 道具效果（OnPlay 触发）
    Preview                    ← 即时呈现
```

## 主要栏位（Data 内）

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 道具名（多语系） |
| **Description** | 否 | 风味描述（多语系，附加在自动描述后） |
| **TargetType** | 是 | 使用时的选目标范围（None / Friend / Enemy 等） |
| **Price** | 是 | 商人售价；预设 20，可用 `Auto Price` 按钮重算（基础 3 × 稀有度价值） |
| **Icon** | 是 | 道具图示 |
| **Rarity** | 是 | 稀有度标签 |
| **ItemType** | 是 | `Normal`（普通）或 `QuestItem`（任务道具，剧情用） |
| **ItemUseType** | 是 | `Consume`（用完即消失）/ `OncePerBattle`（每战一次）/ `OncePerTurn`（每回合一次）/ `Infinite`（无限次） |
| **UseTimes** | 视 ItemUseType | 可使用次数（Consume / OncePer* 模式有效） |
| **Unlock** | 否 | 解锁条件 |
| **HideInCodex** | — | 图鉴隐藏（测试道具） |

外层另有 **m_ItemEffects** = `List<RCG_CommonEffect>`，描述使用时触发的效果（OnPlay 是主要 trigger）。

## 行为说明

### 使用流程
1. 玩家从背包选道具 → 进入 `RCG_ItemInfoPanel`。
2. `CheckUsable(data)`：对每个 effect 跑 `CheckPlayable`，全部 true 才能使用。
3. `TriggerEffect(data)`：取所有 `OnPlay` effect 并依序触发（含 try-catch 防爆）。
4. 依 `ItemUseType` 决定是否消失 / 锁定到下回合 / 锁定到下场战斗。

### 描述生成
*   自动由 `Effects` 串接（每个 effect 一行）。
*   外加：非 Normal 的 ItemType 标记、非 Consume 的 UseType 标记。
*   `UseTimes > 1` 时加上「剩余 N/M 次」说明（runtime 才会显示剩余次数）。
*   `m_Description` 有设定时 → 在自动描述下面额外加风味文字。

### Auto Price（编辑器工具）
编辑器 `RCG_ItemDataEditorPage` 上方有 `Auto Price` 按钮：对所有 ItemData 套用 `Price = 3 × Rarity.m_Value`。**会覆盖手动设的价格**。

## 注意事项

*   **ItemUseType = Infinite 时 UseTime 回 0**：runtime 不消耗、不锁定。
*   **`m_Description` 与自动描述会叠加**：手动写的部分显示在效果之后。
*   **QuestItem 不会被一般使用流程消耗**：通常绑定剧情事件触发。
*   **Auto Price 会洗掉手动价**——按之前要确认。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ItemData.cs`
*   **继承自**：`RCG_Asset<RCG_ItemData>`
*   **实作介面**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 栏位对照（外层）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Data` | Data | `ItemData`（巢状） | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ItemData` 巢状内含 `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_Rarity` / `m_ItemType` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock` / `m_HideInCodex`。

### A.3 重要 Method 摘要

*   **`AddItem()`** — 玩家获得：`RCG_Item.Create(ID)` → `m_ItemsData.AddItem`。
*   **`CheckUsable(TriggerEffectData)`** — 对所有 enabled effects 跑 `CheckPlayable` 取 AND。
*   **`TriggerEffect(TriggerEffectData)`** — 取 `OnPlay` effects 并逐一触发（try-catch）。
*   **`Description` / `FullDescription` / `GetDescription`** — 三层描述生成。
*   **`Effects` (property)** — `m_ItemEffects.GetEnableEffects()`。
*   **`Infos` (property)** — 聚合 effect infos + ItemType 描述。
*   **`CreateSelectAssetPage`** — `RCG_ItemDataEditorPage.Create()`。

### A.4 与其他系统的互动

*   **`RCG_Item`** — runtime 道具实例。
*   **`RCG_DataService.Ins.m_ItemsData`** — 玩家背包储存。
*   **`RCG_ItemDataEditorPage`** — 编辑主画面（含 Auto Price 工具）。
*   **`RCG_ItemInfoPanel`** — 详细资讯 UI。

### A.5 已知议题

*   `m_UseTimes` 的「暂时未完成 先隐藏」注解暗示可变使用次数的功能还没完整实作。
*   `SerializeToJson` / `DeserializeFromJson` override 已被注解；曾规划过自动价格迁移逻辑。
