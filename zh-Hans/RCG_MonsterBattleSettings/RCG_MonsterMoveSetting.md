---
title: 怪物移动 说明
description: 让怪物在战场上移动位置（前后排切换），由 AI 行为使用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 怪物移动

> 程式类别名称：`RCG_MonsterMoveSetting`

## 用途
**让怪物在战场上移动位置** — 通常是前后排切换。由怪物 AI 行为使用，常见用途：
*   敌人主动退到后排避战
*   特殊单位的位移技能

> [!IMPORTANT]
> 此设定**仅在编辑怪物资料时**才会出现在下拉选单中。一般卡牌不会看到此选项。

## 主要栏位
（无栏位 — 纯行为设定）

## 行为说明
*   呼叫 `CreateAction.MoveAction(iData.User)` 触发单位移动 Action。
*   实际移动逻辑由 `MoveAction` 的内部规则决定（一般是前后排切换）。
*   描述为 i18n key `UnitAction_MoveActionDes`，精简描述显示移动 sprite。

## 注意事项
*   **与「移动」设定的差别**：「移动」(`RCG_MoveSetting`) 是**通用**移动设定，可指定方向与目标；本设定是**怪物 AI 专用**，行为固定，无栏位可调。
*   **无栏位 = 行为固定**：移动行为由 `CreateAction.MoveAction` 决定；想客制请改用「**移动**」。
*   **无 User 跳过**：`iData.User == null` 时什么也不做。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterMoveSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **无 i18n 类别名 key**：编辑器显示 stripped name `MonsterMove`
*   **限定可选范围**：`RCG_BattleSetting.s_MonsterDataTypes` 中注册（编辑怪物资料时才出现于选单）。

### A.2 栏位对照
（无自有栏位）

### A.3 重要 Method 摘要
*   **`AddAction`**：`iData.User != null` 时 `iData.AddAction(CreateAction.MoveAction(iData.User), iAddActionMode)`。
*   **`GetDescription`** → i18n key `UnitAction_MoveActionDes`。
*   **`GetDescriptionShort`** → 移动 sprite (`EffectIcon.Move`)。

### A.4 与其他系统的互动
*   **`CreateAction.MoveAction(unit)`**：实际的移动 Action 建构工具（与通用「移动」共用？或专属怪物 AI）。
