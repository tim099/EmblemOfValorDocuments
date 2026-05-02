---
title: Runtime 脚本 (RCG_RuntimeScriptData) 说明
description: 把一段逻辑脚本（RCG_RuntimeScript）打包成 Asset，可被多处引用
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime 脚本

> 程式类别名称：`RCG_RuntimeScriptData`

## 用途

**把一段 `RCG_RuntimeScript` 逻辑脚本打包成 Asset**。让多处可重用同一段逻辑而不必复制贴上（例如「死亡时触发的通用流程」「某条件下执行的判断脚本」）。

继承自 `RCG_Asset<RCG_RuntimeScriptData>`。

## 编辑器中的样貌

```
RCG_RuntimeScriptData: <ID>
    RuntimeScript    ← 实际的脚本内容（RCG_RuntimeScript）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **RuntimeScript** | 是 | 实际的脚本（`RCG_RuntimeScript`，含节点、变数、执行顺序等） |

## 行为说明

本档本身只是个容器；所有逻辑都在 `RCG_RuntimeScript` 里。

## 注意事项

*   **Preview 只显示 ID**：要看实际内容必须开编辑按钮进入脚本编辑器。
*   **`RCG_RuntimeScript` 的具体结构**请参考程式内定义（不在此资料的职责内）。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeScriptData.cs`
*   **继承自**：`RCG_Asset<RCG_RuntimeScriptData>`
*   **AssetGroup**：`Runtime`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_RuntimeScript` | RuntimeScript | `RCG_RuntimeScript` | |

### A.3 与其他系统的互动

*   **`RCG_RuntimeScript`** — 实际脚本内容。
