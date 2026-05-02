---
title: 特效设定 说明
description: 播放纯视觉特效（无游戏逻辑）；支援 VFXGenData 或 Timeline 两种模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 特效设定

> 程式类别名称：`RCG_VFXSetting`

## 用途
**播放纯视觉特效**。无游戏逻辑影响，纯粹做演出。常见用途：
*   攻击前的闪光 / 蓄力光环
*   特殊技能的场景特效
*   背景时序动画（Timeline 模式）

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **VFXType** | 是 | 特效类型：<br>• **VFXGenData** — 使用 `RCG_CommonVFXGenData` 生成单一特效（预设）<br>• **Timeline** — 依时序播放多个音效 / 特效 |
| **VFXGenData** | VFXType=VFXGenData 时 | 特效模板。 |
| **Settings** | VFXType=Timeline 时 | 时序设定清单（每项一个 `VFXSetting`，可指定时间与内容）。 |

## 行为说明
*   **VFXGenData 模式**：直接 `m_VFXGenData.GetData().PlayVFX(iData, token)`。
*   **Timeline 模式**：每个 `VFXSetting` 并行 `Play(token)`，全部 `await` 完成后结束。
*   **不参与融合**：`GetFusionBaseSetting()` 回传 null。

## 注意事项
*   **VFX 为空**：`VFXGenData = null` 时不会 crash 但不播放任何效果。
*   **Timeline 并行与顺序**：所有设定**并行播放**而非依序；想要严格序列请改用「组合效果」+ 多个 VFXSetting。
*   **战斗节奏**：本设定会 await 直到特效播完才继续 — 过长的特效会拖慢战斗节奏。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VFXSetting.cs`
*   **继承自**：`RCG_BattleSetting`
*   **i18n 类别名 key**：`RCG_VFXSetting` → 「特效设定」
*   **同档顶层 enum**：`VFXType` (`VFXGenData`, `Timeline`)

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | Localize Key | 备注 |
|---|---|---|---|---|
| `m_VFXType` | `VFXType` | `VFXType` (档内 enum) | — | 预设 `VFXGenData` |
| `m_VFXGenData` | `VFXGenData` | `RCG_CommonVFXGenData` | — | `[Conditional(... VFXGenData)]` |
| `m_Settings` | `Settings` | `List<VFXSetting>` | — | `[Conditional(... Timeline)]` |

### A.3 重要 Method 摘要
*   **`AddAction`** (async)：依 `m_VFXType` 分流：
    *   `VFXGenData` → `await m_VFXGenData.GetData().PlayVFX(iData, iToken)`。
    *   `Timeline` → 收集每个 `setting.Play(iToken)` 为 `UniTask`，`UniTask.WhenAll(tasks)` 并行等待。
*   **`GetFusionCandidateSettings`** → 空清单；**`GetFusionBaseSetting`** → null。

### A.4 与其他系统的互动
*   **`RCG_CommonVFXGenData / VFXSetting`**：特效模板与时序设定容器。
*   **`PlayVFX(iData, token)`**：实际播放入口。

### A.5 已知议题
*   旧版 `m_VFX` (`RCG_VFXResData`) 与 `m_VFXTime` 已被新架构（VFXGenData）取代，反序列化逻辑保留注解供参考。
