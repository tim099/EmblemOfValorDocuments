---
title: 自定义状态 (RCG_CustomStatusData) 说明
description: 战斗中可加在单位身上的状态（buff / debuff / DoT 等）：层数变化、抵销、免疫、抗性、攻防 buff
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 自定义状态

> 程式类别名称：`RCG_CustomStatusData`

## 用途

**战斗中可加在单位身上的状态（buff / debuff）模板**。例如「中毒：每回合扣 5 血」「力量+2」「免疫负面状态」「闪避：层数 ≥ 攻击力时免伤」。本类别把所有状态相关属性集中一处：层数变化规则、抵销关系、免疫 / 抗性、攻防 buff 加成、单位特殊状态（晕眩、咏唱⋯）。

继承自 `RCG_Asset<RCG_CustomStatusData>`，实作介面：`RCGI_Status`。

## 编辑器中的样貌

```
RCG_CustomStatusData: <ID>
    Name / StatusEffectType / StatusClass
    AtkBuff / DefBuff           ← 攻防加成设定
    IconSprite / Tags
    LayerIncreaseVFX / StatusPersistantVFX
    StatusOffsets / OffsetTags  ← 互相抵销的状态
    StatusAlterOn               ← 哪些 trigger 上层数会变
    Effects                     ← 触发效果
    UnitStates                  ← 对应的特殊状态（晕眩、闪避⋯）
    StopDecayStatus             ← 使目标状态不衰减
    ImmuneStatus                ← 免疫状态（不再获得）
    Resistance                  ← 抗性（每层 1% 机率使状态无效）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 状态名（多语系） |
| **StatusEffectType** | 是 | `Buff` / `Debuff` / 其他类型；影响显示色与抵销判断 |
| **StatusClass** | — | `Normal` / `AtkBuff` / `DefBuff` / `Immune`（免疫所有负面） |
| **AtkBuff / DefBuff** | 否 | 多个 atk / def buff 设定（伤害倍率、护甲加成⋯） |
| **IconSprite** | 是 | 状态图示（`RCG_IconSprite` 引用） |
| **Tags** | 否 | 状态分类标签（Buff / Debuff / DoT / Feature⋯） |
| **LayerIncreaseVFX** | — | 层数增加时的特效 |
| **StatusPersistantVFX** | — | 常驻单位特效（持有此状态时恒显示） |
| **StatusOffsets** | 否 | 互相抵销的具体状态（例：力量 ↔ 虚弱） |
| **OffsetTags** | 否 | 与哪些**标签类型**抵销（例：所有 Debuff 都被某 buff 抵销） |
| **StatusAlterOn** | 否 | 哪些 trigger 上层数变化（dictionary：`trigger → DecreaseType`） |
| **Effects** | 否 | 各 trigger 触发的效果 |
| **UnitStates** | 否 | 对应的单位特殊状态列举值（`Stun` / `Guard` / `Dodge`⋯） |
| **StopDecayStatus** | 否 | 使目标状态不衰减的清单 |
| **ImmuneStatus** | 否 | 免疫的状态（不再获得） |
| **Resistance** | 否 | 每层 1% 机率使目标状态无效 |

## 行为说明

### 层数变化 (`GetDecreaseType`)
查 `m_StatusAlterOn` 表，依 trigger 取得对应的 `eStatusDecreaseType`：
*   `None` 不变 / `DecreaseOneLayer` -1 / `Clear` 全清 / `DecreaseHalf` 减半
*   `AddOneLayer` +1 / `ClearImmediately` 立刻清 / `Eliminate` 消除（不触发结束效果）

### 抵销判断 (`CheckIsOffset`)
*   `StatusClass = Immune` → 所有 Tag 含 `StatusDebuff` 的状态都会被抵销。
*   `OffsetTags` 命中对方 Tag → 抵销。
*   `StatusOffsets` 含对方 ID → 抵销。

### 描述生成
分两段（中间空行）：
1. **抵销段**：列出此状态会抵销的对象。
2. **效果段**：UnitStates 描述 / Atk-DefBuff 描述 / StopDecay / Immune / Resistance / Effects 触发描述。

### IconTMPKey
`RCG_BattleSetting.IsShowOnUI = true` 时回传 IconSprite 的 TMPKey（给 TextMeshPro 图示用）；否则回 LocalizedName。

### 触发效果 (`OnTriggerEffect`)
从 `m_Effects` 取对应 trigger 并依序触发。

## 注意事项

*   **`Immune` 类型自动抵销所有 Debuff**：不需手动列 `OffsetTags = StatusDebuff`，逻辑内建。
*   **`m_AtkBuff / m_DefBuff` 是 list**：可叠多个（不同条件下不同加成）；旧版 `m_AtkBuffData / m_DefBuffData` 已被取代并注解。
*   **`UnitStates` 直接对应 enum**：晕眩、闪避、格挡等是写死在 enum 内的 13 种特殊状态；无法新增（要改程式）。
*   **Resistance 是机率制**：每层 1%，10 层 = 10% 抗性，效果不叠到 100%。
*   **StatusFeature Tag**：含此 tag 的状态名称前面会加「特性: 」前缀。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CustomStatusData.cs`
*   **继承自**：`RCG_Asset<RCG_CustomStatusData>`
*   **实作介面**：`RCGI_Status`
*   **AssetGroup**：`EditCharacter`

### A.2 栏位对照（节选）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_StatusEffectType` | StatusEffectType | `StatusEffectType` enum | `Buff` / `Debuff` |
| `m_StatusClass` | StatusClass | `StatusClass` enum | `Normal` / `AtkBuff` / `DefBuff` / `Immune` |
| `m_AtkBuff` | AtkBuff | `List<RCG_AtkBuffData>` | |
| `m_DefBuff` | DefBuff | `List<RCG_DefBuffData>` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |
| `m_Tags` | Tags | `List<RCG_StatusTagGenData>` | |
| `m_LayerIncreaseVFX` | LayerIncreaseVFX | `RCG_CommonVFXGenData` | 预设 `VFX_StatusLayerID` |
| `m_StatusPersistantVFX` | StatusPersistantVFX | `RCG_CommonVFXGenData` | |
| `m_StatusOffsets` | StatusOffsets | `List<RCG_CustomStatusGenData>` | |
| `m_OffsetTags` | OffsetTags | `List<RCG_StatusTagGenData>` | |
| `m_StatusAlterOn` | StatusAlterOn | `Dictionary<RCG_EffectTriggerOn, eStatusDecreaseType>` | |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_UnitStates` | UnitStates | `List<UnitState>` | enum: Stun / Guard / Dodge / etc. |
| `m_StopDecayStatus` | StopDecayStatus | `List<RCG_CustomStatusGenData>` | |
| `m_ImmuneStatus` | ImmuneStatus | `List<RCG_CustomStatusGenData>` | |
| `m_Resistance` | Resistance | `List<RCG_CustomStatusGenData>` | |

### A.3 重要 Method 摘要

*   **`GetDecreaseType(triggerOn)`** — 查 m_StatusAlterOn 表。
*   **`CheckIsOffset(targetStatus)`** — 三层判断：Immune class / OffsetTags / StatusOffsets。
*   **`OnTriggerEffect(triggerOn, data)`** — 触发效果。
*   **`TriggerOnUnitState(triggerOn)`** — 是否会在此 trigger 变化或触发效果（quick check）。
*   **`Description` (property)** — 大型 StringBuilder 组合：Offsets + UnitStates + AtkBuff + DefBuff + StopDecay + Immune + Resistance + Effects。
*   **`IconTMPKey`** — UI 模式下回 TMPKey，否则回 LocalizedName。
*   **`CreateLayerIncreaseVFX / CreateStatusPersistantVFX`** — async VFX 建立。

### A.4 与其他系统的互动

*   **`RCG_StatusGenData`** — Asset Entry 包装；`Status` (property) 回 `new RCG_StatusGenData(ID)`。
*   **`RCG_CustomStatusGenData`** — 直接引用此资料的型别；含 `s_Default` / `s_ChargeUp` 预设实例。
*   **`RCG_AtkBuffData` / `RCG_DefBuffData`** — 攻防 buff 子资料。
*   **`UnitState` (enum)** — 13 种写死的特殊状态。
*   **`RCG_VFXManager`** — 特效建立。

### A.5 已知议题

*   旧版 `m_AtkBuffData` / `m_DefBuffData` 单一栏位已被 list 取代，反序列化迁移逻辑已注解。
*   `Init()` 是空壳，预留进入大地图时的初始化 hook。
