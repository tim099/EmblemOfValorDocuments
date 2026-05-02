---
title: 条件判断说明
description: 依条件分流的战斗设定；条件成立执行一条路径，否则执行另一条
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 条件判断

> 程式类别名称：`RCG_ConditionalSetting`

## 用途
依「**判定状态**」结果**分流**：条件成立 → 跑「符合条件则执行」；不成立 → 跑「不符合条件则执行」。常见用途：
*   「若击杀则加 1 层敏捷，否则造成额外 3 点伤害」
*   「若 HP < 50% 则自我治疗，否则加护甲」
*   「若手牌 ≥ 5 则抽 1 张，否则弃 1 张」

## 编辑器中的样貌
```
▼ ✓ [条件判断(Conditional)] 若 (条件描述) 则 ⋯ 否则 ⋯
    判定状态           ▶ (条件清单)
    符合条件则执行     ▶ (子设定清单)
    不符合条件则执行   ▶ (子设定清单)
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **判定状态** | 否（但通常要） | 条件清单。**全部** 条件都成立才视为「符合」（AND 逻辑）。空清单 = 永远符合。 |
| **符合条件则执行** | 是（至少一项） | 条件成立时要执行的子设定清单。 |
| **不符合条件则执行** | 否 | 条件不成立时要执行的子设定清单。可以为空（不符合就什么都不做）。 |

## 行为说明

### 条件如何判定
*   全部条件都通过才算「符合」（AND 结合）。
*   清单为空时 = **永远符合** → 永远走「符合条件则执行」这条路径。
*   每个条件本身有自己的设定（例如「击杀后」「HP 比例」「持有特定状态」等），请查看各自的条件类型说明。

### 战斗中发生什么
1. 触发此设定时计算条件结果。
2. 符合 → 依序执行「符合条件则执行」清单中的所有**已启用**子设定。
3. 不符合 → 改执行「不符合条件则执行」清单中的所有**已启用**子设定。
4. 子设定清单内未启用（没打勾）的会被自动跳过。

### 描述会怎么显示
*   只有「符合条件则执行」+ 有条件：「**若 {条件}，则 {符合内容}**」
*   有「不符合条件则执行」：另起一行接「**否则 {不符合内容}**」
*   多个条件之间以「**且**」串接

### 预览伤害的限制
卡片右上角的预览伤害会**先试算条件**，挑对应的路径取最大值显示。但 — 

> [!WARNING]
> 对于「**结果型条件**」（例如「**若击杀**」「**若触发暴击**」），预览时动作还没发生，无法准确得知结果。预览显示可能与实际伤害不符。**若卡片要对玩家承诺伤害数字，请改用稳定条件**（如 HP 比例、层数比较、持有状态等）。

## 注意事项

*   **判定状态为空 + 不符合条件则执行有东西** = **设计反模式**：条件永远成立，「不符合」那条路径永远不会跑，但描述会显示出来，玩家会疑惑。**至少加一条判定状态，或删掉「不符合条件则执行」的内容。**
*   **避免巢状过深**：条件判断里再放条件判断可以，但描述会变成「若 A 则：若 B 则：⋯」 — **建议用多条件 AND 取代巢状**。
*   **条件之间是 AND，不是 OR**：要 OR 逻辑目前需用两个「条件判断」分别处理（或请程式扩充 OR 条件容器）。
*   **未启用的子设定**：在「符合 / 不符合」两条路径内，没打勾的设定会被略过，但**仍会出现在描述合成中**（部分情况），请确认资料整洁。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConditionalSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ConditionalSetting` → 「条件判断」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CardConditions` | 判定状态 | `List<RCG_Condition>` | `CardConditions` | AND 结合；空清单 = 永远 true |
| `m_IfConditionFit` | 符合条件则执行 | `List<RCG_BattleSetting>` | `IfConditionFit` | |
| `m_ElseCondition` | 不符合条件则执行 | `List<RCG_BattleSetting>` | `ElseCondition` | |
| `IfConditionFit` (property) | — | `List<RCG_BattleSetting>` | — | `m_IfConditionFit.GetEnableBattleSettings()` |
| `ElseCondition` (property) | — | `List<RCG_BattleSetting>` | — | `m_ElseCondition.GetEnableBattleSettings()` |

### A.3 重要 Method 摘要
*   **`AddAction`**：在 `AddActionTrigger` 中以 `m_CardConditions.CheckConditions_AND(iData)` 判断，分流呼叫 `IfConditionFit` 或 `ElseCondition` 中各子设定的 `AddAction(iData, InsertInOrder)`。
*   **`Infos`** → 聚合 `m_CardConditions[*].Infos` + `IfConditionFit[*].Infos` + `ElseCondition[*].Infos`，全部 `AppendIfNotRepeat` 去重。
*   **`GetBattleSettings<T> / (Type)`** → 自身 + 递回两条路径（**注意 OR 都递回，不挑分支**）。
*   **`GetDescriptionFormat`**：
    1. 暂存 `iData.m_FullSentence`，设 `false`。
    2. 有 If 子设定 + 有条件 → 套 `IfCardConditions` 模板「若 {ConditionDes}」。
    3. 套 `ConditionFitThenDes` 模板「则 {IfDescription}」。
    4. 有 Else → 换行接 `ElseTriggerEffectsDes`「否则 {ElseDescription}」。
    5. 复原 `m_FullSentence`，套 `iData.GetDescription` 句尾修饰。
*   **`ConditionDes` (private property)** → 将 `m_CardConditions[*].Description` 用 `WordSeperator + ConditionAnd + WordSeperator` 串起来。
*   **`GetPreviewDamage`** → 直接以当前 `iData` 跑 `CheckConditions_AND`，挑对应分支取 `Mathf.Max`（**对结果型条件不准**）。
*   **`PreloadData`** → 两条路径各自 `await` 子设定的 `PreloadData`。
*   **`GetFusionCandidateSettings`** → 两条路径下所有 `IsEnable = true` 的子设定的候选聚合（**不含自身**）。
*   **`GetFusionBaseSetting`** → clone 自身结构，重建两条路径为 placeholder 化版本，过滤掉 disable。

### A.4 与其他系统的互动
*   **`RCG_Condition`** → 条件判断的基底；具 `Description` / `CheckCondition(iData)` 等。
*   **`CheckConditions_AND` (extension)** → `List<RCG_Condition>` 的扩充方法，全 true 才回传 true。
*   **i18n keys**：`IfCardConditions` / `ConditionFitThenDes` / `ElseTriggerEffectsDes` / `ConditionAnd`。
*   **`LocalizedStringUtils.WordSeperator()`** → 中文无空白、英文加空白的语境感知分隔符。

### A.5 已知议题
*   旧版的 `ElseTriggerEffects` 逻辑（纯串接 + 首字大写）以 `// ` 注解保留作对比。
*   `m_FullSentence` 暂存还原是**手动操作**，若中途 throw exception 会泄漏状态 — 改 `try/finally` 较稳。
*   `GetPreviewDamage` 对「结果型条件」（`OnKill` 等）的不准特性是设计取舍，请于设计时回避。
