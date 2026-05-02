---
title: 怪物逃跑 说明
description: 让怪物（敌人）从战场逃跑，播放横向移动动画并从战场移除
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 怪物逃跑

> 程式类别名称：`RCG_MonsterFleeSetting`

## 用途
**让怪物从战场逃跑** — 播放单位向场外移动的动画，然后从战场上移除（**不算击杀，不触发击杀类效果**）。常见用途：
*   特定敌人的「逃跑」行为（HP 低于门槛时逃离）
*   剧情类「敌人退场」演出

> [!IMPORTANT]
> 此设定**仅在编辑怪物资料（`RCG_UnitData` / `RCG_MonsterActionData`）时**才会出现在下拉选单中。一般卡牌不会看到此选项。

## 主要栏位
（无栏位 — 纯行为设定）

## 行为说明
*   播放横向移动动画：敌方往右移 2000 单位、玩家方往左移 2000 单位（0.8 秒）。
*   动画结束后呼叫 `aUser.Flee()` 从战场移除。
*   描述格式为 i18n key `FleeActionDescription`（含完整描述）/ `FleeActionDescriptionShort`（简短）。

## 注意事项
*   **不算击杀**：与「即死」「死亡」不同，**不会触发击杀数记录、击杀类条件**。
*   **无栏位 = 行为固定**：移动方向与时长都写死；想要客制请用其他设定组合。
*   **仅怪物用**：放在玩家卡上理论上会运作，但逻辑为「敌人逃跑」设计。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterFleeSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **无 i18n 类别名 key**：编辑器显示 stripped name `MonsterFlee`
*   **限定可选范围**：`RCG_BattleSetting.s_MonsterDataTypes` 中注册（编辑怪物资料时才出现于选单）。

### A.2 栏位对照
（无自有栏位）

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 `iData.User.UnitFaction` 决定 `aX = -2000` 或 `+2000`，呼叫 `UnitAnimController.MoveUnitLocal(token, new Vector3(aX, 0), 0.8f, false)`，后 `aUser.Flee()` 从战场移除。
*   **`GetDescription / GetDescriptionShort`**：i18n key `FleeActionDescription` / `FleeActionDescriptionShort`。

### A.4 与其他系统的互动
*   **`RCG_BattleUnit.Flee`**：移除单位的入口（与「捕获」共用）。
*   **`UnitAnimController.MoveUnitLocal`**：横移动画。
