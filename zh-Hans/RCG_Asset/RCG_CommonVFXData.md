---
title: 通用特效 (RCG_CommonVFXData) 说明
description: 通用 VFX 设定：特效资源、附带音效、演出时间、附着位置（DisplayPos）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 通用特效

> 程式类别名称：`RCG_CommonVFXData`

## 用途

**战场上各种通用特效模板**。例如「状态层数增加」「充能光环」「过载」「选取圈」「死亡」等不属于攻击特效的视觉效果。每个 `RCG_CommonVFXData` 含 VFX 资源、附带音效、演出时间、贴到单位身上的位置。

继承自 `RCG_Asset<RCG_CommonVFXData>`。

## 编辑器中的样貌

```
RCG_CommonVFXData: <ID>
    VFX             ← VFX 资源（RCG_VFXResData）
    SE              ← 附带音效（可空）
    VFXTime         ← 特效演出时间（秒）
    DisplayPos      ← 特效附着位置（EDisplayPos enum）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **VFX** | 是 | VFX 资源 |
| **SE** | 否 | 附带音效（建立 VFX 时自动播放） |
| **VFXTime** | 是 | 演出时间（秒），预设 0.4f |
| **DisplayPos** | 是 | 附着位置（`EDisplayPos.DisplayPos` 为预设） |

## 行为说明

### `CreateVFX(token)`
1. `m_VFX.IsEmpty` → return null。
2. 透过 `m_VFX.CreateVFX()` 建立。
3. 设 `vfx.CommonVFXData = this`（给 runtime 反查设定）。
4. `m_SE.GetData().PlaySE()` 同步播音。

### `PlayVFX(data, token)`
建立 VFX → 设定位置（`SetPosition(iData.User)`）→ 等待 `m_VFXTime` 秒。

### 预设 ID 常数
`RCG_CommonVFXGenData` 提供多个内建 ID：
*   `VFX_StatusLayer` — 状态层数增加
*   `VFX_ChargeEffect` / `VFX_OverloadEffect` — 充能 / 过载
*   `VFX_SelectionRing` — 选取圈
*   `CommonVFX_Dead` — 死亡

也提供 static 实例 `s_ChargeEffect / s_OverloadEffect / s_SelectionRing / s_Dead`，方便不必手写 ID 字串。

## 注意事项

*   **`m_SE` 在 `CreateVFX` 时自动播放**：不要在外层再呼叫一次 `PlaySE()` 否则会双重播放。
*   **`m_VFXTime > 0` 才会等待**：设 0 表示「fire and forget」（建立后立即返回）。
*   **`m_DisplayPos` enum**：决定 VFX 挂在单位的哪个层级（前景 / 背景 / 中央等）；具体 enum 值参考程式定义。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CommonVFXData.cs`
*   **继承自**：`RCG_Asset<RCG_CommonVFXData>`
*   **AssetGroup**：`EditVFX`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_VFX` | VFX | `RCG_VFXResData` | |
| `m_SE` | SE | `RCG_SEGenData` | |
| `m_VFXTime` | VFXTime | `float` | 预设 0.4 |
| `m_DisplayPos` | DisplayPos | `EDisplayPos` enum | |

### A.3 重要 Method

*   **`CreateVFX(token)`** — 建立 + 设 CommonVFXData + 播音。
*   **`PlayVFX(data, token)`** — 建立 + 设定位置 + 等待 VFXTime。

### A.4 与其他系统的互动

*   **`RCG_VFXResData`** — VFX 资源。
*   **`RCG_SEGenData / RCG_SEData`** — 音效系统。
*   **`RCG_VFX`** — runtime 特效实例。
*   **`RCG_CommonVFXGenData`** — Asset Entry；含多个内建 ID 与 static 实例。
*   **`EDisplayPos` (enum)** — 显示位置定义。

### A.5 已知议题

*   旧版 `DeserializeFromJson` 对 LoadType=Resource 的 LogError 已注解。
