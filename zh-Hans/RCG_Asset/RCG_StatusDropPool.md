---
title: 状态效果掉落池 (RCG_StatusDropPool) 说明
description: 定义「会掉哪些状态效果」的资料；随机 Buff/Debuff、神秘祝福等情境用的池
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 状态效果掉落池

> 程式类别名称：`RCG_StatusDropPool`

## 用途

定义「**这个情境下可能抽到哪些状态效果**」。例如「神坛给予的随机祝福」「诅咒宝箱的随机 debuff」「特殊事件的群体 buff」都从这里抽 `RCG_CustomStatusData`。

继承自 `RCG_Asset<RCG_StatusDropPool>`，实作介面：`UCL.Core.UCLI_ShortName`。

## 编辑器中的样貌

```
RCG_StatusDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← 没有 FilterDrop
    ▼ DropPool / MixDropPools
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **DropType** | 是 | `DropPool` / `MixPool`（**只有两种模式**，没有 FilterDrop） |
| **DropPool** | DropType=DropPool | 状态清单 + 权重 |
| **MixDropPools** | DropType=MixPool | 混合其他池子 |

## 行为说明

与其他 Drop Pool 骨架类似，但**只有两种模式**：直接列、混合别的池。没有「依条件动态筛」这个选项。

## 注意事项

*   **没有 FilterDrop 模式**：要做动态筛选需手动列举。
*   **enum 名称叫 `DropType`**（不是 `EDropType`）。
*   无 unlock 等 runtime 筛选。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StatusDropPool.cs`
*   **继承自**：`RCG_Asset<RCG_StatusDropPool>`
*   **AssetGroup**：`EditDropSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_DropPool` | 掉落池 | `RCG_CommonDropSetting<RCG_CustomStatusGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | 混合池 | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | 只有 `DropPool` / `MixPool` |

### A.3 与其他系统的互动

*   **`RCG_CustomStatusData`** — 掉落目标。
*   **`RCG_CustomStatusGenData`** / **`RCG_StatusDropPoolGenData`** — Asset Entry 包装。
