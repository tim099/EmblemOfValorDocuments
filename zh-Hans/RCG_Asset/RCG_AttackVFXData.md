---
title: 攻击特效 (RCG_AttackVFXData) 说明
description: 卡牌攻击时的特效设定：发射 VFX、命中 VFX、是否冲向敌人；含 preload 机制
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻击特效

> 程式类别名称：`RCG_AttackVFXData`

## 用途

**卡牌攻击时的特效模板**。一张攻击卡可指定：
*   **发射 VFX**（`m_VFX`）：攻击瞬间在使用者位置播放
*   **命中 VFX**（`m_HitVFX`）：伤害结算时在敌人位置播放
*   **是否冲向敌人** + 动画时间配置（冲向时的延迟、移动时间、停留、退回）

继承自 `RCG_Asset<RCG_AttackVFXData>`。

## 编辑器中的样貌

```
RCG_AttackVFXData: <ID>
    AttackVFXSetting (m_AttackVFXSetting)
        VFX                       ← 发射特效
        HitVFX                    ← 命中特效
        MoveTowardEnemy           ← 是否冲向敌人
        MoveTowardEnemySetting    (true 时) ← 动画时间配置
    Note                          ← 备注
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **VFX** | 是 | 发射 VFX；预设 Addressable `AttackVFXs/VFX_AttackEffect` |
| **HitVFX** | 否 | 命中 VFX |
| **MoveTowardEnemy** | — | 攻击时是否移动到敌人身旁 |
| **MoveTowardEnemySetting** | MoveTowardEnemy=true | 动画时间：`AttackAnimDelay` / `MoveTime` (0.5) / `WaitTime` (0.8) / `MoveBackTime` (0.25) / `OffsetX` |
| **Note** | 否 | 备注（编辑时自用，runtime 不显示） |

## 行为说明

### Preload (`PreloadData`)
卡牌资料载入时呼叫 `AttackVFXSetting.PreloadData(token)`：把 `m_VFX` 与 `m_HitVFX` 提前载入记忆体，避免战斗中第一次播放时卡顿。

### 冲向敌人
`m_MoveTowardEnemy = true` 时：
1. 套 `m_AttackAnimDelay` 等待。
2. 移到敌人身旁（`m_MoveTime`）。
3. 在敌人位置停留（`m_WaitTime`）— 期间播 VFX、结算伤害。
4. 退回原位（`m_MoveBackTime`）。
5. `m_OffsetX` 控制停留位置的水平偏移（敌我相反——玩家攻击敌人时 +X，敌人反过来就是 -X）。

## 注意事项

*   **预设 ID `AttackEffectPlain`**（`RCG_AttackVFXGenData.DefaultID`）：是内建「平凡攻击」特效。
*   **`HitVFX` 路径常数**：`PathConst.HitVFXs`，预设为空（很多攻击不需要单独命中特效）。
*   **`Note` 栏位是纯备注**：无实际作用，给设计师自己看的。
*   **Preload 后仍可能在战斗首发卡顿**：如果用 Resource 路径而非 Addressable，预载效果有限。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AttackVFXData.cs`
*   **继承自**：`RCG_Asset<RCG_AttackVFXData>`
*   **AssetGroup**：`EditVFX`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_AttackVFXSetting` | AttackVFXSetting | `AttackVFXSetting`（巢状） | 主资料 |
| `m_Note` | Note | `string` | |

`AttackVFXSetting` 内含 `m_VFX` / `m_HitVFX` / `m_MoveTowardEnemy` / `m_MoveTowardEnemySetting`。
`MoveTowardEnemySetting` 内含 `m_AttackAnimDelay` / `m_MoveTime` / `m_WaitTime` / `m_MoveBackTime` / `m_OffsetX`。

### A.3 重要 Method

*   **`AttackVFXSetting.PreloadData(token)`** — 预载 m_VFX + m_HitVFX。

### A.4 与其他系统的互动

*   **`RCG_VFXResData`** — VFX 资源包装。
*   **`PathConst.AttackVFXs / HitVFXs`** — Addressable 路径常数。
*   **`RCG_AttackVFXGenData`** — Asset Entry；预设 `AttackEffectPlain`。
*   **战斗动画系统** — runtime 套用 `MoveTowardEnemySetting` 的时间参数。

### A.5 已知议题

*   旧 ID 迁移逻辑（`m_ID == "AttackEffect"` → `DefaultID`）已注解。
