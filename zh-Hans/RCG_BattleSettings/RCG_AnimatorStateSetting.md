---
title: AnimatorState（动画状态）说明
description: 控制单位 Animator 状态切换或一次性动画播放的战斗设定（无描述显示）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# AnimatorState（动画状态）

> 程式类别名称：`RCG_AnimatorStateSetting`

## 用途
**纯动画驱动**的战斗设定，无游戏逻辑影响：
*   **持续状态切换**（例如「进入防御姿势」「站起」）
*   **一次性动画播放**（例如「挥手」「眨眼」）

> [!NOTE]
> 此设定**不会出现在卡片描述上**（`GetDescriptionFormat` 与 `GetDescriptionShort` 都回传空字串），纯粹作为视觉呈现。

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **ActionType** | 是 | 动作类型：<br>• **SetState** — 切换持续性的 Animator bool 状态（例如进入/离开防御姿势）。<br>• **PlayAnim** — 播放一次性动画并等到结束。 |
| **AnimType** (`UnitAnimType`) | 是 | 要操作的动画类型（攻击、被击、防御…）。 |
| **Enable** | ActionType=SetState 时必填 | bool；切换持续状态时的目标值（true = 进入，false = 离开）。`PlayAnim` 模式下会自动隐藏。 |
| **Target** | 是 | 动画播放的目标单位选择器。 |

## 行为说明
*   **SetState** 模式：对每个目标的 `UnitAnimController.SetAnimState(AnimType, Enable)`。
*   **PlayAnim** 模式：对每个目标 await `PlayAnim(AnimType)` 直到动画结束（**会卡住战斗流程直到完成**）。
*   无动画控制器的目标会被静默跳过。

## 注意事项
*   **PlayAnim 会延迟后续动作**：因为是 await 式，战斗节奏会被它拉长，安排在回圈内要小心节奏感。
*   **SetState 开关务必对称**：开了一个 `SetState(true)` 却没对应的 `SetState(false)`，状态会持续到战斗结束。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AnimatorStateSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **无 i18n 类别名 key**：编辑器显示 stripped name `AnimatorState`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_ActionType` | `ActionType` | `AnimActionType` (enum) | — | `SetState` / `PlayAnim` |
| `m_AnimType` | `AnimType` | `UnitAnimType` (enum) | — | 动画类型 |
| `m_Enable` | `Enable`（=「启用」） | `bool` | `Enable` | `[Conditional(nameof(m_ActionType), false, AnimActionType.SetState)]` 条件显示 |
| `m_Target` | `Target` (=「目标」) | `RCG_SelectTargetData` | `Target` | |

### A.3 重要 Method 摘要
*   **`GetDescriptionFormat / GetDescriptionShort`** 都回传 `string.Empty` — 刻意隐藏。
*   **`AddAction`** 包 `AddAsyncActionTrigger`，依 `ActionType` 走不同分支。

### A.4 与其他系统的互动
*   **`UnitAnimController.SetAnimState / PlayAnim`**：实际的动画驱动入口。
*   **`UnitAnimType`**：动画类型 enum（与 Spine animation name 对应）。
