---
title: 执行时资料 (RCG_RuntimeData) 说明
description: 通用 runtime 变数容器：依 RuntimeStructData 定义动态结构（Int/Bool/Struct/List/Dic/Enum 等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 执行时资料

> 程式类别名称：`RCG_RuntimeData`

## 用途

**通用的 runtime 变数容器**。系统内各处需要「动态结构的储存格」就用这个——例如战斗阶段状态、卡牌触发时机计数、各种临时 flag。实际结构由 `RCG_RuntimeStructData`（型别表）定义；`RCG_RuntimeData` 只是**这个结构的一个实例**，含初始值。

继承自 `RCG_Asset<RCG_RuntimeData>`，实作介面：`UCLI_FieldOnGUI`。

## 编辑器中的样貌

```
RCG_RuntimeData: <ID>
    StructType   ← 引用的 RCG_RuntimeStructData ID（型别定义）
    DefaultValue ← 此实例的初始值（依 StructType 动态绘制栏位）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **StructType** | 是 | 引用的 `RCG_RuntimeStructData` ID（决定本实例的资料形状） |
| **DefaultValue** | 是 | 此实例的初始值；型别变动时会自动重建 |

## 行为说明

### Type 变更时自动重建
`DefaultValue` 的 getter 比对 `m_DefaultValue.ID` 与 `m_StructType.ID`，不一致 → null 并重建。**避免旧型别资料残留**。

### 序列化
`SerializeToJson` 会把 base + `DefaultValue` 一起输出（key = `DefaultValueKey = "DefaultValue"`）；反序列化时若有此 key 才覆写 DefaultValue。

### 预设取得 (`InGameData`)
`RCG_RuntimeData.InGameData` 取 `RCG_RuntimeGenData.InGameDataID` 对应的 Asset，作为「全局 In-Game 变数储存」。

### `RCG_RuntimeDataConst`（档内子类）
跳过 m_StructType 编辑，直接显示 DefaultValue。标 `[UCL_IgnoreAsset]` 不会作为独立 Asset 出现。

## 注意事项

*   **`StructType` 必须先存在**：要先建好对应的 `RCG_RuntimeStructData` 才能在这里引用。
*   **改 StructType 会清空现有 DefaultValue**：型别不一致直接重建，旧资料会失。
*   **`DefaultType` enum 列举常用 ID**：`BattleState` / `CardTriggerTiming` 是内建常用 struct type 的快捷方式。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeData.cs`
*   **继承自**：`RCG_Asset<RCG_RuntimeData>`
*   **实作介面**：`UCLI_FieldOnGUI`
*   **AssetGroup**：`Runtime`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_StructType` | StructType | `RCG_RuntimeStructGenData` | 型别表引用 |
| `m_DefaultValue` (private) | DefaultValue | `RCG_RuntimeObject` | 透过 property 动态建立 |

### A.3 重要 Method 摘要

*   **`InGameData` (static)** — `Util.GetData(InGameDataID)`，全局 in-game 变数储存。
*   **`DefaultValue` (property)** — 含 type-mismatch 自动重建逻辑。
*   **`SerializeToJson / DeserializeFromJson`** — 含 DefaultValue 子物件序列化。
*   **`GetEnumIDs()`** — 对 Enum 型别取所有 enum 字串值。
*   **`OnGUIDefaultValue(field, dataDic)`** — 绘制 DefaultValue 编辑栏。
*   **`OnGUI(field, dataDic, params)`** — `UCLI_FieldOnGUI` 介面实作。

### A.4 与其他系统的互动

*   **`RCG_RuntimeStructData`** — 型别定义来源。
*   **`RCG_RuntimeObject`** — 动态值的容器类别。
*   **`RCG_RuntimeGenData`** — Asset Entry。
*   **`RCG_RuntimeDataConst`**（同档）— 不可独立存在的常数变体。

### A.5 已知议题

*   `RCG_RuntimeDataConst` 标 `[UCL_IgnoreAsset]` 但仍能被引用，使用时要避免误建 Asset。
