---
title: Diminished（已被削弱占位）说明
description: 「无效卡牌效果」削弱后的占位类别；显示红字提示且不可再被削弱
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Diminished（已被削弱占位）

> 程式类别名称：`RCG_DiminishedPlaceholder`

## 用途
**「无效卡牌效果」（`RCG_DiminishSetting`）削弱后的占位类别**。原本的叶效果（例如「攻击」）会被替换为这个 placeholder，在描述中显示**红字「已被削弱」**。

> [!IMPORTANT]
> 这个设定**通常不会由你手动建立** — 它由削弱系统在执行时自动替换进来。如果你在资料中看到它，代表这张卡曾经被削弱过。

## 编辑器中的样貌
通常不会出现在新建设定的下拉中（虽然 `AllTypes` 有列），出现时栏位空无一物 — 它就是个视觉 marker。

## 主要栏位
（无）

## 行为说明
*   触发时什么也不做（纯 placeholder）。
*   描述永远显示为**已被削弱**（红字，`RCG_Extensions.TagColors.Diminished`）。
*   作为「叶节点」存在，**不可再被削弱**（融合候选为空）。

## 注意事项
*   **不要在资料中手动建立**：除非你想故意预先标记某个位置为「已削弱」。
*   **与「Placeholder」的差别**：`RCG_PlaceholderSetting` 是**融合系统**用的中性占位（蓝色框）；`RCG_DiminishedPlaceholder` 是**削弱系统**用的（红字）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishedPlaceholder.cs`
*   **继承自**：`RCG_PlaceholderSetting`
*   **无 i18n 类别名 key**：编辑器显示 stripped name `Diminished`

### A.2 栏位对照
（无自有栏位）

### A.3 重要 Method 摘要
*   **`GetDescription / GetDescriptionShort / GetShortName`**：均回传 `UCL_LocalizeManager.Get("DiminishedPlaceholder").GetTagColor(TagColors.Diminished)`。
*   **`GetFusionCandidateSettings`** → 空清单（不能作为融合候选）。
*   **`GetFusionBaseSetting`** → `this`（保留自身为 base，**不会被进一步替换**）。

### A.4 与其他系统的互动
*   **`RCG_DiminishSetting.AddAction`**：触发此 placeholder 的替换流程。
*   **`RCG_CardBattleData.Diminish`**：执行替换的入口。
*   **`RCG_Extensions.TagColors.Diminished`**：红字色票来源。
*   **i18n key `DiminishedPlaceholder`**：显示文字（「已被削弱」之类）。
