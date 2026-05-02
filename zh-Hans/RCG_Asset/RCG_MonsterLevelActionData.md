---
title: 等级化怪物动作 (RCG_MonsterLevelActionData) 说明
description: 把同一招式的不同等级版本绑在一起；难度提升时自动换成更强的版本
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 等级化怪物动作

> 程式类别名称：`RCG_MonsterLevelActionData`

## 用途

**把「同一招式的多个等级版本」绑在一起**。例如「攻击_近距_x1」是 `Lv0`、「攻击_近距_x2」是 `Lv1`、「攻击_近距_AOE」是 `Lv2`⋯ 怪物等级提升时系统自动依索引换成更强的版本。也可选用「**单一动作模式**」(`m_UseLevelAsIndex = false`)：只放一个 Action，由动作内部用 HiddenVariable 自行读等级调整。

继承自 `RCG_Asset<RCG_MonsterLevelActionData>`。

## 编辑器中的样貌

```
RCG_MonsterLevelActionData: <ID>
    UseLevelAsIndex (bool)
    MaxSkillLevel
    SkillLevelOffest
    ▼ Actions (UseLevelAsIndex = true)   ← 索引模式：列出每个等级对应的 Action
    ▼ SingleAction (UseLevelAsIndex = false) ← 单一模式：只放一个 Action
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **UseLevelAsIndex** | 是 | `true`：用 `Actions[level]` 取对应动作；`false`：永远用 `SingleAction`，由内部处理等级 |
| **MaxSkillLevel** | 是 | 等级上限；超过会 clamp（预设 100） |
| **SkillLevelOffest** | — | 等级基准偏移（>0 才生效）；用来**整体提升**这组动作的有效等级 |
| **SingleAction** | UseLevelAsIndex=false | 唯一的 Action（自行用 `m_SkillLevel` 变数调整威力） |
| **Actions** | UseLevelAsIndex=true | 依等级排序的 Action 清单；`Actions[i]` 对应 level i |

## 行为说明

### `GetAction(level)`
1. 加上**全局难度技能等级加成**：`level += DifficultyData.m_EnemySkillLevel`。
2. 若 `SkillLevelOffest > 0`，再加上 offset。
3. clamp 到 `[0, MaxSkillLevel]`。
4. **索引模式** (`UseLevelAsIndex = true`)：
    *   `Actions` 为空 → 回 `Idle`。
    *   否则 `result = Actions[clamp(level, 0, Actions.Count - 1)]`。
5. **单一模式**：直接 `result = SingleAction`。
6. 把 `level` 写到 `result.m_Action.m_SkillLevel`，再回传。

### 预览
*   索引模式：列出所有 Action（可逐个展开预览）。
*   单一模式：只显示一行 SingleAction。

## 注意事项

*   **`Actions[i]` 与 `level i` 对应**：列表索引就是等级，没列到的等级会被 clamp 到最后一个。
*   **`MaxSkillLevel = 100` 比 `Actions.Count` 大时**：超过 `Actions.Count - 1` 的等级全部用最后一个 Action（强度卡在顶）。
*   **`SkillLevelOffest` 只在 >0 时生效**：负值会被忽略；想降低强度要走全局难度设定。
*   **单一模式的 `m_SkillLevel`**：runtime 写入回传的是 clone，原 Asset 不受影响。
*   **`m_SingleAction` 的 HiddenVariable** 是设计上的扩充点：在 Action 内部用变数绑定 `SkillLevel` 可实现「同一招式按等级线性增强」而不必开十个 Action。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterLevelActionData.cs`
*   **继承自**：`RCG_Asset<RCG_MonsterLevelActionData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_UseLevelAsIndex` | UseLevelAsIndex | `bool` | 预设 `true` |
| `m_MaxSkillLevel` | MaxSkillLevel | `int` | 预设 100 |
| `m_SkillLevelOffest` | SkillLevelOffest | `int` | 预设 0；只在 > 0 时加 |
| `m_SingleAction` | SingleAction | `RCG_MonsterActionGenData` | `Conditional(UseLevelAsIndex == false)` |
| `m_Actions` | Actions | `List<RCG_MonsterActionGenData>` | `Conditional(UseLevelAsIndex == true)` |

### A.3 重要 Method 摘要

*   **`GetAction(int level)`** — 主入口；含难度加成 / offset / clamp / level 写回 Action。
*   **`Preview`** — 列出所有 Action（索引模式）或单一 Action（单一模式）。

### A.4 与其他系统的互动

*   **`RCG_MonsterActionData`** — 此处掉的 Action 模板。
*   **`RCG_MonsterActionGenData`** — Asset Entry 包装；预设 `IdleID = "Idle"`。
*   **`RCG_MonsterLevelActionGenData`**（档内）— 对外引用此资料的型别，自带 `m_Level` (`IntVariable`)；`GetAction()` 会用此 level 去查 `RCG_MonsterLevelActionData`。
*   **`RCG_DataService.Ins.m_DifficultyData.m_EnemySkillLevel`** — 全局技能等级加成。

### A.5 已知议题

*   `RCG_MonsterLevelActionGenData` 内部建构出错时会印 LogError 后 fallback 到 `Idle`；隐藏 bug 风险。
