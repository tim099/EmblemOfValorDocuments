---
title: 卡牌转换 说明
description: 把指定的卡牌转换为另一种卡牌（手牌、选中、本卡三种来源）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌转换

> 程式类别名称：`RCG_CardConvertSetting`

## 用途
把指定的卡牌**整批转换**成另一种卡牌。常见用途：
*   「将手牌全部变为亡者卡」
*   「将这张卡变为强化版」
*   「将选中的卡转为废牌」

也用于系统内部 — 角色死亡时自动把不能再打出的卡转为「亡者卡」。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **ConvertType** | 是 | 要转换的来源：<br>• **SelectedCards** — 玩家选择的手牌<br>• **AllHandCards** — 全部手牌<br>• **ThisCard** — 触发此效果的卡片本身 |
| **ConvertTarget** | 是 | 要转换成的目标卡牌（`RCG_CardGenData`）。 |

## 行为说明
*   描述格式依 `ConvertType` 套不同 i18n key（`CardConvert_SelectedCards` / `CardConvert_AllHandCards` / `CardConvert_ThisCard`）。
*   执行时：
    1. 依 ConvertType 收集要转换的卡牌清单。
    2. 对每张卡 `aDeck.Convert(旧卡, 新卡)` 替换。
    3. **手牌中的卡**会播放转换动画（每张间隔 0.2 秒），其他位置（牌堆）的直接替换无动画。

## 注意事项
*   **ThisCard 只在卡牌触发时有效**：若被状态 / 战斗事件触发，没有「这张卡」的概念，会找不到目标。
*   **SelectedCards 必须事先选好**：通常前面要接一个「**选取手牌**」设定来填入 `iData.SelectedHandCards`。
*   **转换目标不存在 ID 会 NRE**：请确认 `ConvertTarget` 指向的卡片 ID 已注册。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardConvertSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_CardConvertSetting` → 「卡牌转换」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_ConvertType` | `ConvertType` | `ConvertType` (enum) | — | `SelectedCards` / `AllHandCards` / `ThisCard` |
| `m_ConvertTarget` | `ConvertTarget` | `RCG_CardGenData` | — | 目标卡牌模板 |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：
    1. 依 `m_ConvertType` 收集 `aCards`（手牌 / 选中 / 本卡）。
    2. 对每张 clone 出新 `RCG_CardBattleData`，呼叫 `aDeck.Convert(旧, 新)`。
    3. 手牌类的卡片播放 `CardConvertAnim` 动画（间隔 0.2 秒），其余直接替换。
*   **`RemoveDeadUnitCards(triggerEffectData)` (static)**：角色死亡 hook — 找出 `!HaveRequireSkills(PlayerSkillTags)` 的卡，全部转成 `RCG_CardData.DeadCardID`。
*   **`LocalizeKey`** → `"CardConvert_" + m_ConvertType.ToString()`，描述依此查表。

### A.4 与其他系统的互动
*   **`RCG_Player.Ins.Deck.Convert`**：实际执行卡片替换。
*   **`RCG_CardBattleData.CreateCard(target)`**：clone 出新卡实例。
*   **`RCG_Card.CardConvertAnim`**：手牌转换动画。
*   **`RCG_CardData.DeadCardID`**：死亡时自动转换的目标。
