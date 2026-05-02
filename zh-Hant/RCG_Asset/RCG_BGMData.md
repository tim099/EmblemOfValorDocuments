---
title: 背景音樂 (RCG_BGMData) 說明
description: BGM 設定：音檔、音量；支援 PlayBGM（取代）與 PushBGM（堆疊）兩種播放模式
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 背景音樂

> 程式類別名稱：`RCG_BGMData`

## 用途

**背景音樂的模板**。與一次性音效 (`RCG_SEData`) 不同，BGM 是持續播放的背景音樂；地圖、戰鬥、選單各有自己的 BGM。本資料定義 BGM 的音檔、音量、備註。

繼承自 `RCG_Asset<RCG_BGMData>`。

## 編輯器中的樣貌

```
RCG_BGMData: <ID>
    BGM        ← 音檔（RCG_AudioData，預設 GroupBGM）
    Volume     ← 音量
    Note       ← 備註
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **BGM** | 是 | 音檔，預設 `BGM_MapBGM` |
| **Volume** | 是 | 音量 0~1（slider），預設 1.0 |
| **Note** | 否 | 備註（編輯時自用） |

## 行為說明

### `PlayBGM()` vs `PushBGM()`
*   **PlayBGM**：取代當前 BGM（停掉舊的、播新的）。
*   **PushBGM**：堆疊到 BGM stack 上（常用於「戰鬥開始播戰鬥音樂、結束 pop 回原本的地圖音樂」）。
兩者都有對應的 async 版本（`PlayBGMAsync` / `PushBGMAsync`）。

### `m_BGM.IsEmpty` 時靜默 return
不會 LogError；只有 clip 載入失敗（IsEmpty=false 但 GetClip 回 null）才會 LogError。

### 預設 ID
`RCG_BGMData.DefaultBGM = "BGM_MapBGM"`：地圖預設 BGM。

## 注意事項

*   **PushBGM 沒有對應的 PopBGM 在這個檔**：pop 邏輯在 `UCL_GameAudioService` 自己處理（外層呼叫 `PopBGM`）。
*   **BGM 與 SE 走不同混音器**：分別有獨立音量控制；挑錯路徑會被當成 SE。
*   **無 `Preview` override**：用基底類預設繪製，編輯器內看不到音檔波形預覽。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BGMData.cs`
*   **繼承自**：`RCG_Asset<RCG_BGMData>`
*   **AssetGroup**：`EditGameSetting`
*   **常數**：`DefaultBGM = "BGM_MapBGM"`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_BGM` | BGM | `RCG_AudioData` | 預設 `ModResource + GroupBGM + DefaultBGM` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]` |
| `m_Note` | Note | `string` | |

### A.3 重要 Method

*   **`PlayBGM() / PlayBGMAsync()`** — 取代當前 BGM。
*   **`PushBGM() / PushBGMAsync()`** — 堆疊播放。

### A.4 與其他系統的互動

*   **`UCL.Core.Game.UCL_GameAudioService`** — 實際播放服務（`PlayBGM` / `PushBGM`）。
*   **`RCG_AudioData`** — 音檔資源封裝。
*   **`RCG_BGMGenData`** — Asset Entry。
