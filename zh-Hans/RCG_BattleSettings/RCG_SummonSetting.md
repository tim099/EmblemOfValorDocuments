---
title: 召唤 说明
description: 召唤单位到战场（敌人或我方），可指定召唤类型（一般 / 尸位）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 召唤

> 程式类别名称：`RCG_SummonSetting`

## 用途
**召唤单位到战场**。常见用途：
*   召唤我方友军（治疗师、辅助、肉盾）
*   敌人召唤杂兵
*   特殊「在死亡单位位置召唤」效果

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **UnitData** | 是 | 召唤单位的资料；嵌套 `RCG_UnitSummonData`，含召唤类型、单位、是否怪物等。 |

> [!NOTE]
> 嵌套类别 `RCG_UnitSummonData` 才是真正配置处：
> *   `m_Unit` — 要召唤的单位类型
> *   `m_IsMonster` — 预设立场（**敌方触发时会自动翻转**）
> *   `m_SummonType` — `Default`（找空位）/ `UnitDeadPosiiton`（在死亡单位的位置召唤）
> *   `m_Pos` — 召唤位置（前 / 后 / All）

## 行为说明
*   **立场翻转**：如果是怪物方触发此设定，`IsMonster` 会被反转（怪物召唤我方 = 玩家视角的敌人）。
*   **Default 模式**：优先用 `iData.TargetPositions[0]` 指定的空位；无指定则自动找空位。
*   **UnitDeadPosiiton 模式**：在已阵亡单位的位置召唤（复生 / 尸位机制）。
*   播放 `VFX_Summon` 特效，等召唤动画结束后触发单位的 `OnBattleStart`（进场效果）。
*   描述格式：`SummonDes_{Monster|Player|UnitDeadPosiiton}` + 单位名。

## 注意事项
*   **无空位时静默失败**：找不到位置会 `Debug.LogError` 但不报错给玩家 — 卡片可能看起来「没效果」。
*   **召唤数上限**：受战场最大单位数限制，已满则无法召唤。
*   **进场效果触发**：`OnBattleStart` 会被触发，配备该触发点的能力会立即生效。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SummonSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_SummonSetting` → 「召唤」

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_UnitData` | `UnitData` (=「战斗角色」) | `RCG_UnitSummonData` | `UnitData` | `[AlwaysExpendOnGUI]` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：
    1. 依 `iData.Faction == UnitFaction.Enemy` 翻转 `aIsMonster`。
    2. 依 `m_SummonType` 决定召唤位置（`Default` 找空位 / `UnitDeadPosiiton` 用阵亡单位位置）。
    3. `RCG_BattleField.SpawnUnit` 生成单位 + `VFX_Summon` 特效 + `SummonAnim` 等待 + `OnBattleStart`。
*   **`LocalizeKey` (private)**：依 SummonType 与 IsMonster 取对应 i18n key。
*   **`Info` / `GetDescriptionFormat`**：呈现召唤单位资讯与描述。

### A.4 与其他系统的互动
*   **`RCG_UnitSummonData`**：召唤资料容器（单位、类型、位置、立场）。
*   **`RCG_BattleField.SpawnUnit / TryGetEmptyPositions`**：战场生成入口。
*   **`CommonVFX.VFX_Summon`**：召唤特效。
*   **`RCG_Unit.SummonAnim / OnBattleStart`**：召唤动画与进场效果触发。

### A.5 已知议题
*   旧版 `TriggerAsync` 与部分召唤逻辑保留为注解。
