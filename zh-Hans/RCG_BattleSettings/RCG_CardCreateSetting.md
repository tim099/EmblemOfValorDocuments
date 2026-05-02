---
title: 生成卡牌 说明
description: 生成指定卡牌并加入手牌、牌堆或弃牌堆；可指定生成张数
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 生成卡牌

> 程式类别名称：`RCG_CardCreateSetting`

## 用途
**生成新卡牌**并加入指定位置。常见用途：
*   「加入 2 张烧灼卡到弃牌堆」
*   「将强化版卡放入手牌顶端」
*   「生成 1 张嘲讽卡到牌堆」

需要让玩家**从多选一**请改用「**生成卡牌(多选)**」(`RCG_CreateSelectedCardSetting`)。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CardData** | 是 | 要生成的卡牌模板（`RCG_CardGenData`）。 |
| **IsChanted** | 咏唱卡才显示 | 是否为已咏唱状态（限咏唱卡才有此栏位）。 |
| **CreateCardType** | 是 | 加入位置：<br>• **AddToHandCard** — 加入手牌<br>• **AddToDeck** — 加入牌堆随机位置（预设）<br>• **AddToDiscardPile** — 加入弃牌堆<br>• **AddToDeckTop** — 加入牌堆顶端 |
| **CreateCount** | 是 | 生成张数；支援变数绑定。 |

## 行为说明
*   描述格式为「**生成 {张数} 张 {卡牌名}**」（i18n key `CardCreateEffectDes_{CreateCardType}`）。
*   执行时为每一张呼叫 `RCG_CardBattleData.CreateCard(模板, IsChanted)`，并插入指定位置。
*   生成的卡会记入 `iData.m_TriggerData.m_CreatedCards`，后续设定可引用。

### 卡片融合
两张「生成卡牌」融合时，**生成张数会加总**（例如「生成 1 张 X」+「生成 2 张 X」→「生成 3 张 X」）。

## 注意事项
*   **CardData 指向不存在 ID**：执行时会在 `CardData.GetData()` 拿到 null，卡片名称显示空字串。请确认 ID 已注册。
*   **咏唱卡（IsChant）的 IsChanted 栏位**：只有当 CardData 是咏唱卡时才会出现此栏位（以反射 `IsChantCard()` 判断）。
*   **AddToDeckTop 的视觉反馈**：玩家不会看到动画，仅下次抽牌时冒出来；若要强烈呈现请改 AddToHandCard。
*   **与「生成卡牌(多选)」的差别**：本设定**没有让玩家选**的环节，直接生成；多选版才会出选择 UI。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCreateSetting.cs`
*   **继承自**：`RCG_BattleSetting`，实作 `UCLI_FieldOnGUI`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_CardCreateSetting` → 「生成卡牌」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CardData` | `CardData` | `RCG_CardGenData` | — | 模板 |
| `m_RuntimeCardData` | （隐藏） | `RCG_CardBattleDataPointer` | — | `[UCL_HideOnGUI]`；用于道具化引用 |
| `m_IsChanted` | `IsChanted` | `bool` | — | `[Conditional(nameof(IsChantCard))]` 咏唱卡才显示 |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (enum) | — | 4 种加入位置 |
| `m_CreateCount` | `CreateCount` | `IntVariable` | — | `[UCL_FieldOnGUI]` |

### A.3 重要 Method 摘要
*   **`CardData` (property)**：优先使用 `m_RuntimeCardData`，否则 fallback `m_CardData.GetData()`。
*   **`OnGUI`**：自绘栏位，呼叫 `UCL_GUILayout.DrawField(this, ..., RCG_StaticFunctions.LocalizeFieldName)`。
*   **`AddAction`**：依 `m_CreateCount` 回圈呼叫 `RCG_CardBattleData.CreateCard(aCardData, m_IsChanted)`，记录到 `iData.m_TriggerData.m_CreatedCards`，加入 `CreateAction.CreateCard(aCard, m_CreateCardType)`。
*   **`Fusion(other)`**：clone + `IntVariable.FuseAdd` 合并 `m_CreateCount`；不检查 `m_CardData` 是否相同（**注意可能融合不同卡？**）。
*   **`IsChantCard()` (private)** → for reflection；用于 `Conditional` 判断 `IsChanted` 栏位是否显示。
*   **`SerializeToJson / DeserializeFromJson`**：保留 `m_RuntimeCardData` 序列化逻辑（已注解，目前走 base）。

### A.4 与其他系统的互动
*   **`RCG_CardBattleData.CreateCard(...)`**：产生卡片实例。
*   **`CreateAction.CreateCard`**：实际的 Action class。
*   **`RCG_CardBattleDataPointer`**：runtime 卡片参照器，用于物品化（道具中嵌套生成卡牌）。
*   **`RCG_StaticFunctions.LocalizeFieldName`**：栏位名 i18n callback。

### A.5 已知议题
*   `m_RuntimeCardData` 的序列化逻辑被注解掉（走 base 预设），确认此栏位是否仍需序列化保存。
*   `Fusion` 不检查两者 `m_CardData` 是否相同，理论上可能让「生成 X」+「生成 Y」融合成「生成 X 两次」（取前者的 CardData）— 须验证设计意图。
