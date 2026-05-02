---
title: 设定战斗资讯 说明
description: 修改战斗阶段的全域变数值（战斗内共享资料）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 设定战斗资讯

> 程式类别名称：`RCG_SetBattleDataSetting`

## 用途
**修改战斗阶段的共享资料**（`RCG_BattleManager.BattleData`）— 在这场战斗中跨设定共享的数值容器，类似「战斗内全域变数」。常见用途：
*   「累积造成的总伤害」
*   「记录玩家本场使用了多少次某动作」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **RuntimeDataOperator** | 是 | 内嵌的 `RCG_RuntimeDataSetter`（指定要设定的栏位 + 值 + 计算公式）。预设绑定 `RCG_RuntimeStructGenData.BattleDataID`（战斗资料结构）。 |

## 行为说明
*   触发时呼叫 `RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)`，把计算结果写入指定栏位。
*   描述完全代理到 `m_RuntimeDataOperator.GetDescription`。
*   `GetShortName` 预设为 `"SetBattleData:{operator.GetShortName()}"`。

## 注意事项
*   **资料结构需事先定义**：`RuntimeDataOperator` 要设定哪个栏位，由 `RCG_RuntimeStructGenData.BattleDataID` 结构定义 — 想新增栏位请查该资料结构。
*   **不能融合**：本设定不参与卡片融合（`GetFusionBaseSetting` 回传 null）。
*   **战斗结束后资料消失**：BattleData 是战斗内的；跨战斗要记录请改用「**资源变化**」或全域 RuntimeData。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetBattleDataSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_SetBattleDataSetting` → 「设定战斗资讯」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_RuntimeDataOperator` | `RuntimeDataOperator` | `RCG_RuntimeDataSetter` | — | 预设绑 `RCG_RuntimeStructGenData.BattleDataID` |

### A.3 重要 Method 摘要
*   **`AddAction`**：`RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)`。
*   **`GetDescription / GetDescriptionFormat / GetShortName`**：代理到 `m_RuntimeDataOperator`。
*   **`GetFusionCandidateSettings`** → 空清单；**`GetFusionBaseSetting`** → null（不参与融合）。

### A.4 与其他系统的互动
*   **`RCG_BattleManager.BattleData`**：战斗内共享资料容器。
*   **`RCG_RuntimeDataSetter`**：栏位设定器，含公式计算。
*   **`RCG_RuntimeStructGenData.BattleDataID`**：对应的资料结构模板。
