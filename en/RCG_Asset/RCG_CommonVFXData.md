---
title: 通用特效 (RCG_CommonVFXData) 說明
description: 通用 VFX 設定：特效資源、附帶音效、演出時間、附著位置（DisplayPos）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 通用特效

> 程式類別名稱：`RCG_CommonVFXData`

## 用途

**戰場上各種通用特效模板**。例如「狀態層數增加」「充能光環」「過載」「選取圈」「死亡」等不屬於攻擊特效的視覺效果。每個 `RCG_CommonVFXData` 含 VFX 資源、附帶音效、演出時間、貼到單位身上的位置。

繼承自 `RCG_Asset<RCG_CommonVFXData>`。

## 編輯器中的樣貌

```
RCG_CommonVFXData: <ID>
    VFX             ← VFX 資源（RCG_VFXResData）
    SE              ← 附帶音效（可空）
    VFXTime         ← 特效演出時間（秒）
    DisplayPos      ← 特效附著位置（EDisplayPos enum）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **VFX** | 是 | VFX 資源 |
| **SE** | 否 | 附帶音效（建立 VFX 時自動播放） |
| **VFXTime** | 是 | 演出時間（秒），預設 0.4f |
| **DisplayPos** | 是 | 附著位置（`EDisplayPos.DisplayPos` 為預設） |

## 行為說明

### `CreateVFX(token)`
1. `m_VFX.IsEmpty` → return null。
2. 透過 `m_VFX.CreateVFX()` 建立。
3. 設 `vfx.CommonVFXData = this`（給 runtime 反查設定）。
4. `m_SE.GetData().PlaySE()` 同步播音。

### `PlayVFX(data, token)`
建立 VFX → 設定位置（`SetPosition(iData.User)`）→ 等待 `m_VFXTime` 秒。

### 預設 ID 常數
`RCG_CommonVFXGenData` 提供多個內建 ID：
*   `VFX_StatusLayer` — 狀態層數增加
*   `VFX_ChargeEffect` / `VFX_OverloadEffect` — 充能 / 過載
*   `VFX_SelectionRing` — 選取圈
*   `CommonVFX_Dead` — 死亡

也提供 static 實例 `s_ChargeEffect / s_OverloadEffect / s_SelectionRing / s_Dead`，方便不必手寫 ID 字串。

## 注意事項

*   **`m_SE` 在 `CreateVFX` 時自動播放**：不要在外層再呼叫一次 `PlaySE()` 否則會雙重播放。
*   **`m_VFXTime > 0` 才會等待**：設 0 表示「fire and forget」（建立後立即返回）。
*   **`m_DisplayPos` enum**：決定 VFX 掛在單位的哪個層級（前景 / 背景 / 中央等）；具體 enum 值參考程式定義。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CommonVFXData.cs`
*   **繼承自**：`RCG_Asset<RCG_CommonVFXData>`
*   **AssetGroup**：`EditVFX`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_VFX` | VFX | `RCG_VFXResData` | |
| `m_SE` | SE | `RCG_SEGenData` | |
| `m_VFXTime` | VFXTime | `float` | 預設 0.4 |
| `m_DisplayPos` | DisplayPos | `EDisplayPos` enum | |

### A.3 重要 Method

*   **`CreateVFX(token)`** — 建立 + 設 CommonVFXData + 播音。
*   **`PlayVFX(data, token)`** — 建立 + 設定位置 + 等待 VFXTime。

### A.4 與其他系統的互動

*   **`RCG_VFXResData`** — VFX 資源。
*   **`RCG_SEGenData / RCG_SEData`** — 音效系統。
*   **`RCG_VFX`** — runtime 特效實例。
*   **`RCG_CommonVFXGenData`** — Asset Entry；含多個內建 ID 與 static 實例。
*   **`EDisplayPos` (enum)** — 顯示位置定義。

### A.5 已知議題

*   舊版 `DeserializeFromJson` 對 LoadType=Resource 的 LogError 已註解。