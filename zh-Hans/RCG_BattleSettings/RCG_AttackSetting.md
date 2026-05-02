---
title: 攻击设定说明
description: 对目标造成伤害的战斗设定；含攻击力、攻击次数、类型、标签、特效与进阶详细设定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻击

> 程式类别名称：`RCG_AttackSetting`

## 用途
对**指定目标**造成伤害。游戏中最常见、栏位最多的战斗设定，几乎所有「会打怪」的卡片、敌人技能、反击状态都用它。

## 编辑器中的样貌
```
▼ ?  ✓  [攻击(Attack)] 给予 1 物伤
    ▶ 攻击目标（目标）
      攻击力    [数值] 1
      攻击次数  [数值] 1
      攻击类型  [物伤(Normal)]
    ▶ 攻击标签(0)
    ▶ 攻击特效  [普通攻击特效(打击)(AttackEffectPlain)]
    ▶ 详细设定
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **攻击目标** | 是 | 目标选择器。展开后可设定范围（单体 / AOE / 全敌 / 全友 …）与目标类型。 |
| **攻击力** | 是 | 一次攻击造成的伤害（支援变数绑定，如「等于护甲值」「等于某状态层数」）。**负值会被自动夹到 0**。 |
| **攻击次数** | 是 | 攻击段数。每段独立结算，搭配「详细设定」可决定是否每段重选目标。 |
| **攻击类型** | 是 | 影响伤害结算方式。详见下方对照表。 |
| **攻击标签** | 否 | 攻击上附带的标签（吸血、火、爆击…）。标签本身可带进阶规则，例如「吸血」会把伤害的 X% 转为治疗。展开可加多个。 |
| **攻击特效** | 是 | 命中动画资料；至少要选一个基础攻击特效，否则玩家看不到挥砍动画。 |
| **详细设定** | 否 | 进阶参数，见下方 §进阶设定。 |

### 攻击类型对照
| 显示 | 内部代号 | 物理意义 |
|---|---|---|
| **物伤** | Normal | 受护甲与物伤减免影响 |
| **法伤** | Magic | 受魔抗与法伤减免影响 |
| **生命削减** | Status | 无视护甲与一切减免（状态伤害不视为一般攻击） |
| **反伤** | Counter | 反击用；类型会与被攻击的伤害类型一致 |
| **伤害** | Any | 任意攻击类型；用于泛用触发条件 |

## 进阶设定（展开「详细设定」）

| 编辑器显示 | 预设 | 说明 |
|---|---|---|
| **特攻** (`AntiAttack`) | （空） | 对特定怪物标签造成加倍伤害。例如选「龙族」即「对龙特攻」。空清单代表无特攻。 |
| **AttackAtOnce** | ✗ | 打勾 = 同一段攻击一次砸到全部目标（同步动画）；不打勾 = 逐个目标依序攻击。 |
| **StopAttackAfterDeath** | ✓ | 多段攻击时，目标若被击杀则停止后续段数（避免「砍空气」）。 |
| **CounterAttackVFX** | ✗ | 反击时使用受到的攻击动画（沿用对方的挥砍特效）。 |
| **IgnoreBuff** | ✗ | 忽略攻击者所有 Buff/Debuff（显示与结算都会无视）。 |
| **描述类型** (`DescriptionType`) | Default | 影响卡片描述句型；选 `InflictDamageOn` 会用「对 X 造成 Y 伤害」的句法。 |

## 行为说明

### 预览伤害（卡片右上角小数字）
*   **仅在「攻击目标」设为「攻击选中目标」**时计算（其他范围模式如 AOE 或自身回传「非攻击牌」）。
*   会把当前已套用的 Buff/Debuff/特攻 都计入。
*   每段伤害独立预览 — **不乘以攻击次数**，玩家看到的是单段值。

### 攻击力显示（描述中的数字）
*   未勾选 IgnoreBuff 时：显示「**已套 Buff 后**」的攻击力（含 +X 调整值的格式）。
*   勾选 IgnoreBuff 时：显示原始 `攻击力` 数值。

### 卡片资讯（悬停 Tooltip）
会自动列出：
1. 攻击类型（物伤 / 法伤 …）
2. 每个攻击标签（吸血、火 …）
3. 若有「特攻」：列出加倍规则 + 对哪些怪物有效
4. 「攻击目标」带来的额外资讯

## 注意事项

*   **多段 + AOE 是双倍重选**：「攻击次数 = 3 + AOE = 全敌」预设每段都会**重新解析全敌**（敌人阵亡后就少一个目标）。要做「全敌各打 3 次」请改用「**重复触发(回圈)**」包一个 AOE 攻击。
*   **特攻和描述要对齐**：如果「特攻」设了「对龙 1.5 倍」但卡片描述自己写「对龙 2 倍」，玩家会看到不一致 — 两处都是手动维护，**请仔细核对**。
*   **攻击特效不可空**：一定要选一个 `AttackEffectXxx`，否则战斗中无动画播放会 NRE。即使只想做「无视觉反击」也请选 `AttackEffectPlain`。
*   **变数型攻击力的显示**：`攻击力` 用变数时，描述会自动串接 `(变数名)`。如果想要干净的显示，请挑「隐藏变数」型态，描述只会出现最终数字。

---

## 附录：程式人员参考 (Programmer Reference)

> 此段以下使用程式内部术语，受众转为程式人员与 AI agent。前半段内容请优先采信。

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AttackSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 标记
*   **i18n 类别名 key**：`RCG_AttackSetting` → 「攻击」

### A.2 栏位对照（主类别）

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_AttackTarget` | 攻击目标 | `RCG_SelectTargetData` | `AttackTarget` | 取代旧的 `AttackRange` enum |
| `m_Atk` | 攻击力 | `IntVariable` | `Atk` | `[UCL_FieldOnGUI]` |
| `m_AtkTimes` | 攻击次数 | `IntVariable` | `AtkTimes` | `[UCL_FieldOnGUI]` |
| `m_AttackType` | 攻击类型 | `AttackType` (enum) | `AttackType` | 列举 i18n：`AttackType_Normal` (物伤) / `_Magic` (法伤) / `_Status` (生命削减) / `_Counter` (反伤) / `_Any` (伤害) |
| `m_AttackTagGenDatas` | 攻击标签 | `List<RCG_AttackTagGenData>` | `AttackTagGenDatas` | 同 key 也覆盖 `AttackTags` / `AttackTag` 等别名 |
| `m_AttackVFX` | 攻击特效 | `RCG_AttackVFXGenData` | `AttackVFX` | |
| `m_DetailSetting` | 详细设定 | `DetailSetting` | `DetailSetting` | 巢状类别，栏位见 A.3 |

### A.3 栏位对照（`DetailSetting` 子类别）

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_AntiAttack` | 特攻 | `List<RCG_MonsterTagGenData>` | `AntiAttack` | 对特定怪物标签加倍伤害 |
| `m_AttackAtOnce` | `AttackAtOnce`（未本地化） | `bool` | — | 一次砸全部目标 vs 逐个 |
| `m_StopAttackAfterDeath` | `StopAttackAfterDeath`（未本地化） | `bool` | — | 预设 `true` |
| `m_CounterAttackVFX` | `CounterAttackVFX`（未本地化） | `bool` | — | 反击用对方动画 |
| `m_IgnoreBuff` | `IgnoreBuff`（未本地化） | `bool` | — | 忽略攻击者 Buff/Debuff |
| `m_DescriptionType` | 描述类型 | `DescriptionType` (enum) | `DescriptionType` | `Default` / `InflictDamageOn` |

### A.4 重要 Method 摘要
*   **`Infos`** → 组合 `[AttackType, ...AttackTagGenDatas, AntiAttakDes(若有特攻), ...AttackTarget.Infos]`，最后一段去重。
*   **`GetAtkStr(iUser, iIsShowPoint, iData)`**：
    *   非 `IgnoreBuff` 且 user 非 null → 透过 `iUser.p_BattleUnit.m_UnitStatus.GetAtkDescription(...)` 套上 Buff 修饰。
    *   `m_VariableType` 为 `Value` 或 `HiddenVariable` 时 `iHasModification = false`（不显示调整值格式）。
*   **`GetAttackData(iData, iUser, iAttackTargets)`** → 预览用 `AttackData`（不含反击处理）。
*   **`GetAtk(iData)`** → `Mathf.Max(0, m_Atk.GetValue(iData))`。
*   **`GetPreviewDamage`** → 仅 `m_AttackTarget.m_SelectTargetType == Range && m_TargetPos.m_UnitRange == Target` 时计算，否则回传 -1；其余走 `iTarget.GetDamage(FinalAtk, AttackData)`。
*   **`DeserializeFromJson(JsonData)`** → 呼叫 base，无客制栏位转换（旧转换逻辑已注解）。

### A.5 与其他系统的互动
*   **`AttackData`** → 攻击资料封装，含 `FinalAtk` 计算。
*   **`RCG_PlayerAtkAction`** 系列 → 实际的攻击 Action（透过 `AddAction` 触发；本档未直接呼叫，继承父类流程）。
*   **`RCG_SelectTargetData` / `SelectTargetType` / `UnitRange`** → 通用目标选择器；`AttackRange` enum 是淘汰前的旧型态。
*   **`IntVariable`** → 攻击力 / 攻击次数的数值容器。
*   **`RCG_MonsterTagGenData`** → 特攻判定用的怪物标签。

### A.6 已知议题
*   `AttackRange` enum 已被 `RCG_SelectTargetData` 取代，但仍保留枚举定义 — 反序列化旧 JSON 用。
*   旧的 `CanEnhence` / `Enhence(RestSetting)` 已注解（强化系统重构中）。
*   `GetCommentLine 注解区块` 中保留了 4 处被弃用的描述逻辑（`//public DescriptionType m_DescriptionType` 等）。
