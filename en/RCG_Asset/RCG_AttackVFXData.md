---
title: 攻擊特效 (RCG_AttackVFXData) 說明
description: 卡牌攻擊時的特效設定：發射 VFX、命中 VFX、是否衝向敵人；含 preload 機制
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 攻擊特效

> 程式類別名稱：`RCG_AttackVFXData`

## 用途

**卡牌攻擊時的特效模板**。一張攻擊卡可指定：
*   **發射 VFX**（`m_VFX`）：攻擊瞬間在使用者位置播放
*   **命中 VFX**（`m_HitVFX`）：傷害結算時在敵人位置播放
*   **是否衝向敵人** + 動畫時間配置（衝向時的延遲、移動時間、停留、退回）

繼承自 `RCG_Asset<RCG_AttackVFXData>`。

## 編輯器中的樣貌

```
RCG_AttackVFXData: <ID>
    AttackVFXSetting (m_AttackVFXSetting)
        VFX                       ← 發射特效
        HitVFX                    ← 命中特效
        MoveTowardEnemy           ← 是否衝向敵人
        MoveTowardEnemySetting    (true 時) ← 動畫時間配置
    Note                          ← 備註
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **VFX** | 是 | 發射 VFX；預設 Addressable `AttackVFXs/VFX_AttackEffect` |
| **HitVFX** | 否 | 命中 VFX |
| **MoveTowardEnemy** | — | 攻擊時是否移動到敵人身旁 |
| **MoveTowardEnemySetting** | MoveTowardEnemy=true | 動畫時間：`AttackAnimDelay` / `MoveTime` (0.5) / `WaitTime` (0.8) / `MoveBackTime` (0.25) / `OffsetX` |
| **Note** | 否 | 備註（編輯時自用，runtime 不顯示） |

## 行為說明

### Preload (`PreloadData`)
卡牌資料載入時呼叫 `AttackVFXSetting.PreloadData(token)`：把 `m_VFX` 與 `m_HitVFX` 提前載入記憶體，避免戰鬥中第一次播放時卡頓。

### 衝向敵人
`m_MoveTowardEnemy = true` 時：
1. 套 `m_AttackAnimDelay` 等待。
2. 移到敵人身旁（`m_MoveTime`）。
3. 在敵人位置停留（`m_WaitTime`）— 期間播 VFX、結算傷害。
4. 退回原位（`m_MoveBackTime`）。
5. `m_OffsetX` 控制停留位置的水平偏移（敵我相反——玩家攻擊敵人時 +X，敵人反過來就是 -X）。

## 注意事項

*   **預設 ID `AttackEffectPlain`**（`RCG_AttackVFXGenData.DefaultID`）：是內建「平凡攻擊」特效。
*   **`HitVFX` 路徑常數**：`PathConst.HitVFXs`，預設為空（很多攻擊不需要單獨命中特效）。
*   **`Note` 欄位是純備註**：無實際作用，給設計師自己看的。
*   **Preload 後仍可能在戰鬥首發卡頓**：如果用 Resource 路徑而非 Addressable，預載效果有限。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AttackVFXData.cs`
*   **繼承自**：`RCG_Asset<RCG_AttackVFXData>`
*   **AssetGroup**：`EditVFX`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_AttackVFXSetting` | AttackVFXSetting | `AttackVFXSetting`（巢狀） | 主資料 |
| `m_Note` | Note | `string` | |

`AttackVFXSetting` 內含 `m_VFX` / `m_HitVFX` / `m_MoveTowardEnemy` / `m_MoveTowardEnemySetting`。
`MoveTowardEnemySetting` 內含 `m_AttackAnimDelay` / `m_MoveTime` / `m_WaitTime` / `m_MoveBackTime` / `m_OffsetX`。

### A.3 重要 Method

*   **`AttackVFXSetting.PreloadData(token)`** — 預載 m_VFX + m_HitVFX。

### A.4 與其他系統的互動

*   **`RCG_VFXResData`** — VFX 資源包裝。
*   **`PathConst.AttackVFXs / HitVFXs`** — Addressable 路徑常數。
*   **`RCG_AttackVFXGenData`** — Asset Entry；預設 `AttackEffectPlain`。
*   **戰鬥動畫系統** — runtime 套用 `MoveTowardEnemySetting` 的時間參數。

### A.5 已知議題

*   舊 ID 遷移邏輯（`m_ID == "AttackEffect"` → `DefaultID`）已註解。