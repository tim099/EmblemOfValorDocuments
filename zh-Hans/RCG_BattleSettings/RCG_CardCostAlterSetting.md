---
title: 手牌费用变化 说明
description: 对指定范围的手牌进行费用增加 / 降低 / 归零
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 手牌费用变化

> 程式类别名称：`RCG_CardCostAlterSetting`

## 用途
**改变手牌费用**。常见用途：
*   「全部手牌费用 -1」
*   「一张随机手牌变 0 费」
*   「费用最高的手牌归零」

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **CardAlterRange** | 是 | 影响范围：<br>• **AllHandCards** — 全部手牌<br>• **SelectedCards** — 玩家事前选择的手牌<br>• **OneRandomHandCard** — 随机一张<br>• **ThisCard** — 这张卡本身（卡牌触发才有效）<br>• **MostCostCard** — 费用最高的卡<br>• **NewSelectedCards** — 在本效果中选择（会自动展开选取设定） |
| **CostAlterType** | 是 | 变化模式：<br>• **AddHandCardCost** — 增加费用<br>• **SubHandCardCost** — 降低费用<br>• **HandCardCostToZero** — 变为 0 费（不需填数值） |
| **SelectHandCardSetting** | NewSelectedCards 时必填 | 内嵌的选取设定；其他模式下会自动隐藏。 |
| **费用** (`Cost`) | AddHandCardCost / SubHandCardCost 时必填 | 变化的数值（变为 0 费时忽略）。 |

## 行为说明
*   **NewSelectedCards** 模式会先触发内嵌的选取设定，玩家选完才执行。
*   依范围筛出卡片后依 CostAlterType 套：
    *   增加 → `AlterCost(+Cost)`
    *   降低 → `AlterCost(-Cost)`
    *   归零 → `AlterCost(-当前 Cost)`（直接抵消）

### 卡片融合
两张同 `CostAlterType` 的设定融合会把 `Cost` 加总。**不同 CostAlterType 之间不能融合**（避免「+1 与归零」融合产生语意冲突）。

## 注意事项
*   **OneRandomHandCard** 会即时随机，而非事先抽好；多次触发会抽到不同张。
*   **ThisCard 在非卡牌触发场合无效**：例如被状态效果触发时 `iData.Card == null`。
*   **HandCardCostToZero 不是「费用 -∞」**：是把当前费用减去当前值，**叠加 +1 费后又归零会变 0 费**，非反向加成。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCostAlterSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_CardCostAlterSetting` → 「手牌费用变化」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_CardAlterRange` | `CardAlterRange` | `CardAlterRange` (enum) | — | 6 种来源 |
| `m_CostAlterType` | `CostAlterType` | `CostAlterType` (enum) | — | 3 种模式 |
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[Conditional(m_CardAlterRange, false, NewSelectedCards)]` 条件显示 |
| `m_Cost` | 费用 | `IntVariable` | `Cost` | |

### A.3 重要 Method 摘要
*   **`AddAction`**：`NewSelectedCards` 模式先呼叫 `m_SelectHandCardSetting.AddAction`，后续 ActionTrigger 中依范围取卡并 `AlterCost`。
*   **`Fusion(other)`**：要求两者 `m_CostAlterType` 相同；clone 自身并 `IntVariable.FuseAdd` 合并 `m_Cost`。
*   **`GetDescriptionFormat`**：依 CostAlterType 套不同 i18n key（`AddHandCardCost_Des` / `SubHandCardCost_Des` / `HandCardCostToZero_Des`）。

### A.4 与其他系统的互动
*   **`RCG_Card.AlterCost(int)`**：实际修改费用的入口。
*   **`UCL_Random.Instance.Next`**：OneRandomHandCard 的随机来源。
