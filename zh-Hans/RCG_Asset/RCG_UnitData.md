---
title: 单位资料 (RCG_UnitData) 说明
description: 战场上所有单位（怪物、玩家角色、召唤物）的本体资料：HP、外观、AI 行为、技能等全部定义在这
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 单位资料

> 程式类别名称：`RCG_UnitData`

## 用途

**战场上每一个单位的本体模板**——怪物、玩家角色、召唤物都是这个类别的实例。包含：HP / 战斗力、外观（图、Spine、龙骨、3D Prefab）、AI 行为（不同状态下使用哪些技能）、初始动作、必要的单位设定等。取代旧的 `RCG_MonsterData`。

继承自 `RCG_Asset<RCG_UnitData>`。

## 编辑器中的样貌

```
RCG_UnitData: <ID>
    UnitSetting (m_UnitSetting)        ← 主要设定：HP、标签、初始动作、外观、定位
    MonsterStates (m_MonsterStates)    ← AI 状态机：每个状态下使用哪些 Action
    Preview (右侧)                      ← 即时呈现外观、技能列表、HP
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **UnitSetting** | 是 | 单位的核心设定（HP / 标签 / 初始 Action / 外观 / 定位等） |
| **MonsterStates** | 是 | AI 状态机：dictionary，key 是状态 ID，value 是该状态下可用的 `MonsterAction` 清单。**至少要有 `Default` 状态**（系统会自动补上） |

`UnitSetting` 内部包括：

| 子栏位 | 说明 |
|---|---|
| **Name** | 单位显示名（多语系） |
| **MaxHP** | 最大生命值 |
| **CombatEffectiveness** | 战斗力指标；序列化时若为 0 会自动以 `MaxHP` 补上 |
| **MonsterTags** | 怪物标签（种族、属性等），影响某些卡牌/装备效果 |
| **InitActions** | 进场时触发的初始动作（buff / debuff / 召唤物） |
| **DetailSetting** | 细节设定（攻击次数、计数器⋯，视类型而定） |
| **SummonSetting** | 被召唤时的设定 |
| **UnitDisplayerType** | 显示方式：Sprite / DragonBone / Spine / 3D Model / Prefab |
| **SpriteDisplayData / DragonBoneDisplayData / ...** | 对应显示器的资料（Conditional 显示） |
| **Pos / PosAt** | 站位偏移、座标 |
| **BaseLevel / UnitGenData** | 基础等级与单位 ID 包装 |
| **HasAI / Classes / UnitSkills** | AI 开关、职业（卡牌专精用）、携带的单位技能 |

## 行为说明

### AI 状态机 (MonsterStates)
每个怪物有多个「状态」（例如 `Default`、`Berserk`、`Phase2`），每个状态下有一组可用的 `MonsterAction`。实际战斗中由 `MonsterStateTransition` 决定何时切换状态，由 Action 的选择规则决定当回合用哪一招。

### 显示方式
*   **Sprite**：静态图（最简单，适合 placeholder）。
*   **DragonBone / Spine**：2D 骨架动画。
*   **Model3DDisplayer / Prebab**：3D 模型 / 自订 Prefab。
切换 `UnitDisplayerType` 后对应栏位才会显示。

### 初始动作 (InitActions)
进场时自动套用一次的动作。常见用途：给予自身 buff、召唤从者、设定计数器初值。

### 预览（编辑器内）
右侧即时显示：头像、最大 HP、怪物标签、初始动作描述、各状态下的技能图示与名称。**Low RAM 模式**会跳过头像绘制以省记忆体。

## 注意事项

*   **`Default` 状态必须存在**：系统会在 `OnGUI` 自动补上，但保险起见先设好。
*   **`CombatEffectiveness` 的 fallback**：`SerializeToJson` 时若该值为 0 会用 `MaxHP` 补；想保留 0 须特别处理。
*   **`NullMonsterID = "Null"`、`DefaultUnitID = "Devil"`、`BattleSceneMonsterID = "BattleScene"`**：这些是系统保留 ID，不要拿来命名一般怪物。
*   **编辑介面的 Skill / Action / SummonSetting** 各自从 `RCG_MonsterActionData` / `RCG_UnitSkillData` 等 Asset 引用，本档只持有 ID 包装。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_Battles/RCG_Monsters/RCG_UnitData.cs`
*   **继承自**：`RCG_Asset<RCG_UnitData>`
*   **AssetGroup**：`EditBattleSetting`，sort = `RCG_UnitData`

### A.2 栏位对照（外层）

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_UnitSetting` | UnitSetting | `RCG_UnitSetting`（巢状） | 主要核心资料 |
| `m_MonsterStates` | MonsterStates | `Dictionary<string, MonsterState>` | AI 状态机；key 是状态 ID |

`UnitSetting` 内含 `m_Name` / `m_MaxHP` / `m_CombatEffectiveness` / `m_MonsterTags` / `m_InitActions` / `m_DetailSetting` / `m_SummonSetting` / `m_UnitDisplayerType` + 对应 DisplayData / `m_Pos` / `m_PosAt` / `m_BaseLevel` / `m_UnitGenData` / `m_HasAI` / `m_Classes` / `m_UnitSkills`。

### A.3 重要 Method 摘要

*   **`Preview` / `OnGUI`** — 编辑器渲染；OnGUI 会自动补 `Default` MonsterState。
*   **`SerializeToJson`** — 补 `m_CombatEffectiveness == 0` 的 fallback。
*   **`Avatar` / `GetAvatar(RCG_BattleUnit)`** — 显示器图像取得。
*   **`CreateSelectAssetPage`** — 开 `RCG_UnitDataEditorPage`。
*   **`LocalizeName`** — `GetData().LocalizedName`。

### A.4 与其他系统的互动

*   **`RCG_BattleUnit`** — runtime 战斗单位实例。
*   **`MonsterState` / `MonsterStateTransition`** — 状态机的元件（含 `DefaultStateID`）。
*   **`RCG_MonsterAction` / `RCG_MonsterActionData`** — 怪物可用招式定义。
*   **`RCG_UnitSkillData`** — 单位被动 / 领袖技。
*   **`RCG_UnitDataEditorPage`** — 编辑主画面。

### A.5 已知议题

*   `DeserializeFromJson` 有被注解掉的 `m_UnitSetting = m_MonsterData` 容错（旧版栏位迁移）。
*   程式注解 `// 暂时用HP当作战斗力指标` 指 `m_CombatEffectiveness` 的 fallback 规则待重新设计。
