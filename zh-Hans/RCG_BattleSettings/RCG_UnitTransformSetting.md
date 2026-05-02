---
title: 变身 说明
description: 把目标单位变身为另一种单位；支援预设变身、转生、重生三种模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 变身

> 程式类别名称：`RCG_UnitTransformSetting`

## 用途
**把单位变身为另一种单位**。常见用途：
*   Boss 切换型态（保留血量改变外观）
*   「转生」「重生」类复活机制
*   特殊变身卡牌

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **目标** (`Target`) | 是 | 变身目标选择器。 |
| **UnitData** | 是 | 要变身为哪种单位（`RCG_UnitGenData`）。 |
| **TransformSetting** | 是 | 变身细节设定（嵌套）。 |

### TransformSetting 内栏位
| 栏位 | 预设 | 说明 |
|---|---|---|
| **TransformType** | `Default` | 变身类型：<br>• **Default** — 一般变身。**保留 HP 与状态**，只改变行为与模型（**Boss 换型用**）。<br>• **Reincarnation** — 转生。**只保留状态**；HP / 上限 / 数值全变新单位的；触发初始行动。<br>• **Rebirth** — 重生。**不保留状态**；HP / 上限 / 数值全变新单位的；触发初始行动。 |
| **ClearAllStatus** | `false` | 是否清空所有状态（与 TransformType 的「保留状态」逻辑独立）。 |

## 行为说明
*   描述格式：`UnitTransformDes_{TransformType}`，套入目标与单位名称。
*   执行时依 TransformType 决定保留哪些属性，并进行视觉切换（换 Spine 模型 / AI / 行为）。

## 注意事项
*   **Default vs Reincarnation 区别**：两者都会换模型，但 Default 保留所有当前数值（适合 Boss 切型）；Reincarnation 重置 HP 但保留状态（适合「转生」）。
*   **触发初始行动**：Reincarnation / Rebirth 会跑新单位的 `OnBattleStart`（含进场状态 / 进场攻击等）。
*   **ClearAllStatus 与 TransformType 冲突**：Default + ClearAllStatus 等于「保留 HP 但清状态」 — 罕见组合，请慎用。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_UnitTransformSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_UnitTransformSetting` → 「变身」
*   **同档顶层类**：`TransformSetting`（含 `TransformType` enum 与 `m_ClearAllStatus`）

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_Target` | 目标 | `RCG_SelectTargetData` | `Target` | |
| `m_UnitData` | `UnitData` (=「战斗角色」) | `RCG_UnitGenData` | `UnitData` | |
| `m_TransformSetting` | `TransformSetting` | `TransformSetting` | — | 含 `TransformType` 与 `ClearAllStatus` |

### A.3 重要 Method 摘要
*   **`AddAction`**（档案 80 行外）：依 `TransformType` 切换实际变身逻辑。
*   **`GetDescriptionFormat`** → `UnitTransformDes_{TransformType}` i18n key，套 target / unit。
*   **`Info`** → `new CardInfoData(m_UnitData.GetData())`。

### A.4 与其他系统的互动
*   **`RCG_Unit` 变身入口**（具体 method 名见后续实作）。
*   **`RCG_UnitGenData`**：变身目标单位模板。
*   **`OnBattleStart`**：Reincarnation / Rebirth 的初始行动触发点。
