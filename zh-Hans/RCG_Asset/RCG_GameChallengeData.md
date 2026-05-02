---
title: 游戏挑战 (RCG_GameChallengeData) 说明
description: 全局游戏挑战目标：完成此通关目标才视为「达成挑战」（与 Achievement 不同）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 游戏挑战

> 程式类别名称：`RCG_GameChallengeData`

## 用途

**全局的「通关挑战目标」设定**。例如「击败最终 Boss」「不使用任何道具通关」「3 回合内结束战斗」。挑战是「**通关目标**」级别，与单纯获得的 `RCG_AchievementAsset` 不同——挑战决定「这场游戏是否已通关」。

继承自 `RCG_Asset<RCG_GameChallengeData>`，实作介面：`RCGI_Unloackable`。

## 编辑器中的样貌

```
RCG_GameChallengeData: <ID>
    Name             ← 挑战名（多语系；预设 "GameChallenge"）
    Unlock           ← 解锁条件
    ChallengeGoals   ← 完成条件（多个 RCG_QuestGoalData 共同构成）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **Name** | 是 | 挑战显示名（多语系） |
| **Unlock** | 否 | 解锁条件；空 = 预设已解锁 |
| **ChallengeGoals** | 是 | 完成条件清单（`RCG_QuestGoalData`），达成全部视为通关 |

## 行为说明

### 解锁判断 (`IsUnlocked`)
*   `m_Unlock.IsEmpty || m_Unlock.Unlocked`：没设条件或条件已通过 → 已解锁。

### 完成判断
本档本身不负责「是否完成」的判断；交由外部的 quest manager 或 challenge manager 检查 `m_ChallengeGoals` 的进度。

## 注意事项

*   **预设 ID `Challenge_FinalBoss`** 是「击败最终 Boss」的标准挑战，多数路线通关都用这个。
*   **与 `RCG_AchievementAsset` 的差异**：成就是「附加荣誉」（解锁条件达成 → 获得奖励 / 显示）；挑战是「**通关判定**」（达成 → 该局结束 / 触发结算）。
*   **`m_Name` 预设值是 `"GameChallenge"`**：必须改名才会在 UI 显示有意义的标题。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_GameChallengeData.cs`
*   **继承自**：`RCG_Asset<RCG_GameChallengeData>`
*   **实作介面**：`RCGI_Unloackable`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | 预设 `"GameChallenge"` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_ChallengeGoals` | ChallengeGoals | `List<RCG_QuestGoalData>` | |

### A.3 重要 Method

*   **`IsUnlocked` (property)** — `m_Unlock.IsEmpty || m_Unlock.Unlocked`。
*   **`UnlockEntry`** — `m_Unlock`。
*   **`LocalizedName`** — `m_Name.Name`。

### A.4 与其他系统的互动

*   **`RCG_QuestGoalData`** — 通关条件元素。
*   **`RCG_GameChallengeGenData`** — Asset Entry 包装；预设 `Challenge_FinalBoss`。
*   **`RCG_UnlockEntry`** — 解锁系统。

### A.5 已知议题

*   无 `Preview` 实作；用基底类预设绘制。
