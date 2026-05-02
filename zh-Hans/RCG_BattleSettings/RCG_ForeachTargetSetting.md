---
title: 每个目标分别触发 说明
description: 对选取的每个目标分别执行内含的组合效果（拆分 AOE 为单体回圈）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 每个目标分别触发

> 程式类别名称：`RCG_ForeachTargetSetting`

## 用途
**对每个目标分别触发**内含效果。把 AOE 拆成「**对每个敌人各做一次**」的回圈。常见用途：
*   「对每个血量低于 10 的敌人即死」（AOE 即死必须拆开判定才正确）
*   「对全敌各回 5 点 HP（同时播放动画）」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Targets** | 是 | 要分别触发的目标选择器（通常是 AOE）。 |
| **Content** | 是 | 对每个目标执行的「**组合效果**」。 |

## 行为说明
*   解析 `Targets` 取得目标清单。
*   对每个目标：建立 child `TriggerEffectData`，将 `Targets` 设为该目标单体，再呼叫 `Content.AddAction`。
*   描述格式：「**对每个 {Targets} 分别 {Content 描述}**」（i18n key `ForeachTargetDes`）。

## 注意事项
*   **与「重复触发」的差别**：「重复」是「**对同一目标做 N 次**」；本设定是「**对 N 个不同目标各做一次**」。
*   **预览伤害不适用**：本设定**不覆写 `GetPreviewDamage`**，会走父类预设值 `-1`（非攻击牌）。要显示伤害请在外层用其他方式表达。
*   **巢状 Foreach**：Foreach 内再放 Foreach 是 N×M 的回圈 — 描述会非常复杂，请避免。
*   **空目标清单**：合法，但什么也不会发生。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ForeachTargetSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_ForeachTargetSetting` → 「每个目标分别触发」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Targets` | `Targets` | `RCG_SelectTargetData` | — | |
| `m_Content` | `Content` | `RCG_CombineSetting` | — | 子效果 |

### A.3 重要 Method 摘要
*   **`AddAction`**：对 `m_Targets.GetTargets(iData)` 中每个 target，`iData.CreateChildData(target.BattleName)` → `data.Targets = [target]` → `m_Content.AddAction(data, InsertInOrder)`。
*   **透传到 `m_Content`**：`Infos` / `HasTerm` / `GetCollaborators` / `GetAtk`。
*   **`GetBattleSettings<T> / (Type)`**：自身 + 递回 `m_Content`。
*   **`GetFusionCandidateSettings`** → 直接代理 `m_Content` 的候选。
*   **`GetFusionBaseSetting`**：clone 自身结构，内容换为 placeholder 化的 Combine。

### A.4 与其他系统的互动
*   **`TriggerEffectData.CreateChildData / Targets`**：每目标独立的 child data，避免子效果共用 Targets 干扰。
*   **`LocalizedStringUtils.CapitalizeString`**：暂时的首字大写处理（与 LoopSetting 共用 hack）。
