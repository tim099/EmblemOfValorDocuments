---
title: 攻击指令 说明
description: 与「攻击」类似但可指定攻击者；用于「指挥另一个单位攻击」的情境
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻击指令

> 程式类别名称：`RCG_AttackCommandSetting`

## 用途
和「**攻击**」设定几乎一致，但**多了一个栏位指定攻击者** — 用于「**指挥另一个单位攻击**」的情境，例如：
*   召唤物代替主角发动攻击
*   队友协同攻击
*   多人连击技

> [!NOTE]
> 一般攻击请优先用「**攻击**」设定（`RCG_AttackSetting`），此设定仅在需要明确指定攻击者时才使用。

## 主要栏位

与「**攻击**」几乎相同，仅多一个 **Attacker（攻击者）** 栏位：

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Attacker** | 是 | 攻击发动者选择器（取代战斗中当前的 `User`）。 |
| **攻击目标** | 是 | 同「攻击」设定。 |
| **攻击力** | 是 | 同「攻击」设定。 |
| **攻击次数** | 是 | 同「攻击」设定。 |
| **攻击类型** | 是 | 同「攻击」设定（`Counter` 反击类型会自动继承被攻击的伤害类型）。 |
| **攻击标签** | 否 | 同「攻击」设定。 |
| **攻击特效** | 是 | 同「攻击」设定。 |
| **详细设定** | 否 | 同「攻击」设定（特攻、AttackAtOnce 等）。 |

## 行为说明
*   解析 `Attacker` 拿到实际攻击者；若拿不到（例如选择器条件无人符合）会用「全部活着的单位」第一个作为防呆。
*   `IgnoreBuff = false` 时，攻击次数会经过攻击者的 `m_UnitStatus.GetAtkTimes(...)` 修饰。
*   描述格式为「**{攻击者} 对 {目标} 造成 {攻击力} 伤害**」（i18n key `AttackDommand_Des` 套版）。
*   其他行为（预览伤害、特攻显示、AttackAtOnce 等）与「攻击」一致。

## 注意事项
*   **Attacker 防呆会踩雷**：若选择器解析不到攻击者，**会强制找一个活着的单位**顶上去 — 这虽避免 NRE，但可能让「敌人指令」误触发成「我方攻击」。请仔细设计 Attacker 选择器。
*   **与「攻击」的差别**：两者栏位几乎重复，请避免乱用 — 不需要指定攻击者的场合一律用「攻击」。
*   **Counter 类型**：未由「受到攻击」触发时会降级为 `Normal`（物伤）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackCommandSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_AttackCommandSetting` → 「攻击指令」
*   **TODO 注解**：`// TODO: 测试 QWQ 能可以跟 RCG_AttackSetting 共用逻辑?` — 两类有大量重复码，未来可能合并。

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Attacker` | `Attacker` | `RCG_SelectTargetData` | — | 攻击发动者 |
| `m_AttackTarget` | 攻击目标 | `RCG_SelectTargetData` | `AttackTarget` | |
| `m_Atk` | 攻击力 | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | 攻击次数 | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | 攻击类型 | `AttackType` | `AttackType` | |
| `m_AttackTagGenDatas` | 攻击标签 | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | |
| `m_AttackVFX` | 攻击特效 | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | 详细设定 | `RCG_AttackSetting.DetailSetting` | `DetailSetting` | **共用** `RCG_AttackSetting.DetailSetting` 巢状类别 |

### A.3 重要 Method 摘要
*   **`AddAction`**：解析 `m_Attacker` → `aUser`（防呆：取 `RCG_BattleManager.Ins.AllAliveUnits[0]`）。依 `m_DetailSetting.m_AttackAtOnce` 分流（一次砸全部 / 逐段重选），建立 `AttackData` 并 `RCG_AttackAction`。
*   **`GetDescriptionFormat`**：先套单一攻击描述（与 `RCG_AttackSetting` 共用 `LocalizeKey` 逻辑），再以 i18n key `AttackDommand_Des` 包覆攻击者叙述。

### A.4 与其他系统的互动
*   **`RCG_AttackSetting.DetailSetting / DescriptionType`** — 直接共用巢状类别与枚举。
*   **`AttackData / RCG_AttackAction`** — 与「攻击」一致的下游。
*   **`RCG_BattleField.CurActiveUnit` / `RCG_BattleManager.AllAliveUnits`** — 攻击者解析的后备来源。

### A.5 已知议题
*   与 `RCG_AttackSetting` 重复码极多（attack 力 / 次数 / VFX / DetailSetting / preview 逻辑）— 应重构为共用基底或 helper。
