---
title: SE データ (RCG_SEData)
description: 単一 SE の設定：音声ファイル、音量、AudioType、終了まで同期待機可
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# SE データ

> クラス名：`RCG_SEData`

## 用途

**単一 SE（サウンドエフェクト）の設定**。例：「カード使用音」「命中音」「ボタン押下音」「勝利 BGM」等。各 `RCG_SEData` は1つの音声ファイル + 音量 + タイプ、同期再生または再生終了まで待機可。

`RCG_Asset<RCG_SEData>` を継承。

## エディタ上の見た目

```
RCG_SEData: <ID>
    SE          ← 音声ファイル（RCG_AudioData）
    Volume      ← 音量（0~1、slider）
    AudioType   ← SE タイプ（SE / UI 等）
    Note        ← メモ
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SE** | はい | 音声ファイル（`RCG_AudioData`）、デフォルト `ModResource` 経由読込 |
| **Volume** | はい | 音量 0~1（slider）、デフォルト 1.0 |
| **AudioType** | はい | SE タイプ（`UCL.Core.Game.AudioType`）、デフォルト `SE` |
| **Note** | いいえ | メモ（編集時自用） |

## 動作説明

### `PlaySE()`
非ブロッキング再生 — `PlaySEAsync().Forget()` で即返却、音声ファイル終了を待たない。

### `PlaySEAsync()` (UniTask)
async で clip 取得 → `UCL_GameAudioService.Ins.Play(clip, audioType, volume)`、`AudioPlayer` 返却。

### `PlaySEUntilEnd(token)`
player 取得後に `EndAct` フラグ設定 → `WaitUntil` で終了；「過場 SE 終了後に継続必要」の状況に使用可能。

### デフォルト ID
`RCG_SEGenData.DefaultID = "Null"`：「SE なし」を意味；`s_Victory = "SoundEffect_Victory"` は内蔵勝利 SE。

## 注意事項

*   **`m_SE.IsEmpty` 時に LogError なし**：静黙 return null；SE が再生されたか確認したいなら主動チェック必要。
*   **`PlaySE()` は非ブロッキング**：SE 再生終了まで待つには `PlaySEUntilEnd(token)` 使用。
*   **AudioType がミキサーグルーピングに影響**：UI / SE / BGM に独立音量制御あり、誤タイプで誤調整される。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_SEData.cs`
*   **継承**：`RCG_Asset<RCG_SEData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_SE` | SE | `RCG_AudioData` | デフォルト `ModResource + GroupSoundEffect` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]`、デフォルト 1.0 |
| `m_AudioType` | AudioType | `UCL.Core.Game.AudioType` | デフォルト `SE` |
| `m_Note` | Note | `string` | |

### A.3 主要メソッド

*   **`PlaySE()`** — fire-and-forget。
*   **`PlaySEAsync()`** — clip 取得し再生、`AudioPlayer` 返却。
*   **`PlaySEUntilEnd(token)`** — 再生終了まで待機。

### A.4 他システムとの連携

*   **`UCL.Core.Game.UCL_GameAudioService`** — 実再生サービス。
*   **`RCG_AudioData`** — 音声ファイルリソースラッパー。
*   **`RCG_SEGenData`** — Asset Entry；デフォルト ID `Null`、`s_Victory` 内蔵。
