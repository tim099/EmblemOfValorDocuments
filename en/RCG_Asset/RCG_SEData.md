---
title: 音效資料 (RCG_SEData) 說明
description: 單一音效的設定：音檔、音量、AudioType、可同步等待結束
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 音效資料

> 程式類別名稱：`RCG_SEData`

## 用途

**單一音效（SE）的設定**。例如「卡牌打出音」「命中音」「按鈕點擊音」「勝利音樂」等。每個 `RCG_SEData` 是一份音檔 + 音量 + 類型，可以同步播放或等待播完。

繼承自 `RCG_Asset<RCG_SEData>`。

## 編輯器中的樣貌

```
RCG_SEData: <ID>
    SE          ← 音檔（RCG_AudioData）
    Volume      ← 音量（0~1，slider）
    AudioType   ← 音效類型（SE / UI 等）
    Note        ← 備註
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **SE** | 是 | 音檔（`RCG_AudioData`），預設走 `ModResource` 載入 |
| **Volume** | 是 | 音量 0~1（slider），預設 1.0 |
| **AudioType** | 是 | 音效類型（`UCL.Core.Game.AudioType`），預設 `SE` |
| **Note** | 否 | 備註（編輯時自用） |

## 行為說明

### `PlaySE()`
非阻塞播放——`PlaySEAsync().Forget()` 立刻返回，不等待音檔結束。

### `PlaySEAsync()` (UniTask)
async 取 clip → `UCL_GameAudioService.Ins.Play(clip, audioType, volume)`，回傳 `AudioPlayer`。

### `PlaySEUntilEnd(token)`
取 player 後設定 `EndAct` 旗標 → `WaitUntil` 結束；可用在「過場 SE 結束才能繼續」的情境。

### 預設 ID
`RCG_SEGenData.DefaultID = "Null"`：表示「無音效」；`s_Victory = "SoundEffect_Victory"` 是內建的勝利音效。

## 注意事項

*   **`m_SE.IsEmpty` 時不會 LogError**：靜默 return null；要確認音效有播要主動檢查。
*   **`PlaySE()` 是非阻塞**：要等音效播完用 `PlaySEUntilEnd(token)`。
*   **AudioType 影響混音器分組**：UI / SE / BGM 各有獨立音量控制，挑錯類型會被誤調。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_SEData.cs`
*   **繼承自**：`RCG_Asset<RCG_SEData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_SE` | SE | `RCG_AudioData` | 預設 `ModResource + GroupSoundEffect` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]`，預設 1.0 |
| `m_AudioType` | AudioType | `UCL.Core.Game.AudioType` | 預設 `SE` |
| `m_Note` | Note | `string` | |

### A.3 重要 Method

*   **`PlaySE()`** — fire-and-forget。
*   **`PlaySEAsync()`** — 取 clip 並播放，回 `AudioPlayer`。
*   **`PlaySEUntilEnd(token)`** — 等播完再回。

### A.4 與其他系統的互動

*   **`UCL.Core.Game.UCL_GameAudioService`** — 實際播放服務。
*   **`RCG_AudioData`** — 音檔資源封裝。
*   **`RCG_SEGenData`** — Asset Entry；預設 ID `Null`，`s_Victory` 內建。