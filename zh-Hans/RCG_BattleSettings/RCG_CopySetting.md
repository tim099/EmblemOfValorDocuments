---
title: 复制 说明
description: 复制选取的手牌，可指定加入手牌、牌堆或弃牌堆
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 复制

> 程式类别名称：`RCG_CopySetting`

## 用途
**复制选取的手牌**，产生一张**新的副本**并依设定加入手牌、牌堆或弃牌堆。常见用途：
*   「复制这张卡到手牌」
*   「复制选中的卡牌进牌堆顶」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **SelectHandCardSetting** | 是 | 选取要复制的手牌（含 `SkipSelect` / `ThisCard` / 一般选取等模式）。 |
| **CreateCardType** | 是 | 副本加入位置：手牌 / 牌堆 / 弃牌堆 / 牌堆顶。 |

## 行为说明
*   先触发 `SelectHandCardSetting`，然后对每张选中的卡呼叫 `CopyCard()` 产生副本，并透过 `CreateAction.CreateCard(副本, CreateCardType)` 插入指定位置。
*   描述根据 `SelectType` 不同有三种变化：
    *   `SkipSelect` → 直接显示「复制」
    *   `ThisCard` → 显示「复制这张卡」
    *   其他 → 「{选取描述} 并复制」
*   若 `CreateCardType ≠ AddToHandCard`，会额外附加位置说明。

## 注意事项
*   **副本不继承咏唱状态**：复制后是新卡，原卡的咏唱进度**不会带过去**（除非设计上特别处理）。
*   **复制强化过的卡**：`CopyCard()` 会包含强化效果；副本仍是强化版（**不会降级为基础版**）。
*   **CreateCardType = AddToDeckTop**：玩家不会看到动画；可能误以为没效果。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CopySetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **无 i18n 类别名 key**：i18n key `RCG_CopySetting` 用于描述（「复制」），但 `AllTypes` 显示为 stripped name 「Copy」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_CreateCardType` | `CreateCardType` | `CreateCardType` (enum) | — | 共用 `RCG_CardCreateSetting.CreateCardType` |

### A.3 重要 Method 摘要
*   **`AddAction`**：选牌 → 对每张 `aCard.CopyCard()` → `CreateAction.CreateCard(副本, m_CreateCardType, InsertInOrder)`。
*   **`Infos`**：`RCG_CopySetting` + `CopySettingInfo` i18n。
*   **`GetDescriptionFormat`**：依 `m_SelectHandCardSetting.m_SelectType` 三分支处理。

### A.4 与其他系统的互动
*   **`RCG_CardBattleData.CopyCard()`**：实际的副本产生方法。
*   **`CreateAction.CreateCard`**：副本插入位置的 Action。
*   **`RCG_SelectHandCardSetting`**：选牌触发。
