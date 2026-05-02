---
title: 重复触发(回圈) 说明
description: 把指定的组合效果重复执行 N 次的控制流战斗设定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 重复触发(回圈)

> 程式类别名称：`RCG_LoopSetting`

## 用途
把一段「**组合效果**」**重复执行 N 次**。常见用途：
*   「造成 1 点伤害，重复 5 次」（连击类）
*   「每剩一张手牌就触发一次」（变数绑定的连击）
*   「给予 1 层敏捷，重复 3 次」（叠状态层数）

## 编辑器中的样貌
```
▼ ✓ [重复触发(回圈)(Loop)] (LoopContent) × N
    要重复的次数         [数值] 1
    要重复执行的效果     ▶ (展开后是「组合效果」容器)
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **要重复的次数** | 是 | 重复次数；支援变数绑定（例如「等于剩余卡片数」「等于某状态层数」）。 |
| **要重复执行的效果** | 是 | 一个「组合效果」容器，内含实际要重复的设定。 |

> [!NOTE]
> 为什么内容栏位是「组合效果」而不是直接放一串设定？因为「组合效果」已经处理了多 Setting 的描述合成、Tag 去重、预览合计等逻辑，包一层省事。**单一效果的回圈也请放一个只有一项子设定的「组合效果」**。

## 行为说明

### 战斗中发生什么
1. 系统解析「要重复的次数」拿到 N。
2. 连续执行 N 次「要重复执行的效果」中的内容。
3. 每圈动作都依序插入战斗伫列，与外层的设定串接执行。

### 描述会怎么显示
格式为「**重复 N 次：{内容描述}**」，会自动把内容首字大写。

精简描述（敌方意图列）：直接显示 `{内容} × {N}`。

### 预览伤害不乘 N
卡片预览伤害显示的是「**单圈**」的伤害，不会自动乘以重复次数。例如「重复 3 次造成 4 点伤害」预览显示 4，**不显示 12**。

> [!IMPORTANT]
> 这是设计选择 — 玩家看「每次伤害」较直观。若你要强调总伤害，请在卡片描述中明写「**3 × 4 = 12 点**」。

## 注意事项

*   **重复次数 = 0 或负值**：技术上不会 crash，但等于「卡牌没效果」，玩家会以为 bug。请在公式上夹 `Mathf.Max(1, ...)` 或设计上避开负值情境。
*   **巢状回圈**：可以「重复触发」内再包「重复触发」，但描述会变成「重复 M 次：重复 N 次：⋯」**对玩家极不友善**。请改用 `M*N` 摊平。
*   **多目标 + 回圈**：「攻击全敌 + 重复 3 次」≠「对每个敌人各 3 次」。前者是「3 次 AOE，每次重新解析目标」；后者请用「**回圈每个目标(Foreach)**」设定。
*   **AOE 攻击放在回圈里**：每圈会重新解析目标，所以中途死亡的敌人会在后续圈被跳过，符合直觉。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_LoopSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_LoopSetting` → 「重复触发(回圈)」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_LoopTimes` | 要重复的次数 | `IntVariable` | `LoopTimes` | 支援变数型 |
| `m_LoopContent` | 要重复执行的效果 | `RCG_CombineSetting` | `LoopContent` | 包一层 Combine 为了重用其聚合逻辑 |

### A.3 重要 Method 摘要
*   **`AddAction`**：
    ```csharp
    int aLoopTimes = m_LoopTimes.GetValue(iData);
    for (int i = 0; i < aLoopTimes; i++)
        m_LoopContent.AddAction(iData, AddActionMode.InsertInOrder);
    ```
    **重点**：每圈用 `InsertInOrder`，保证动作顺序与外层串接。
*   **`GetDescriptionFormat`**：
    1. 暂存 `iData.m_FullSentence`，设为 `false` 取干净内容描述。
    2. `LocalizedStringUtils.CapitalizeString` 把首字大写（暂时 hack）。
    3. 复原 `m_FullSentence = true`，套 i18n key `LoopSettingDes`。
*   **`GetDescriptionShort`** → i18n key `LoopSettingDesShort`，格式为 `{Content} × {Times}`。
*   **透传到 `m_LoopContent`**：`Infos` / `HasTerm` / `GetCollaborators` / `GetAtk` / `GetPreviewDamage` / `PreloadData`。
*   **`GetBattleSettings<T> / (Type)`** → 自身 + 递回 `m_LoopContent`。
*   **`GetFusionCandidateSettings`** → 直接代理 `m_LoopContent.GetFusionCandidateSettings()`（**Loop 自身不作为候选**）。
*   **`GetFusionBaseSetting`** → clone 自身结构，内容换为 placeholder 化的 Combine。

### A.4 与其他系统的互动
*   **`RCG_CombineSetting`** → `m_LoopContent` 的容器型态；Loop 透过 Combine 重用「多 Setting 描述合成」逻辑。
*   **`AddActionMode.InsertInOrder`** → 确保多圈动作维持插入顺序，与 `PushBack` 不同。
*   **`IntVariable.GetValue`** → 解析重复次数；不夹值，所以 0 / 负数会直接 0 圈。

### A.5 已知议题
*   `LocalizedStringUtils.CapitalizeString` 是「暂时特殊处理」（程式注解 `//暂时特殊处理`），i18n 重构时应有正规方案。
*   `GetPreviewDamage` 不乘 N 是设计选择而非 bug；若需求转变请在此明文修改。
*   旧版 `CanEnhence` / `Enhence` 注解保留（强化系统旧逻辑）。
