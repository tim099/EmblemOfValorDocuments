---
title: BGM (RCG_BGMData)
description: BGM 設定：音声ファイル、音量；PlayBGM（置換）と PushBGM（スタック）の2種類の再生モード対応
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# BGM

> クラス名：`RCG_BGMData`

## 用途

**BGM のテンプレート**。1度限りの SE (`RCG_SEData`) と異なり、BGM は持続再生される背景音楽；マップ、戦闘、メニューに各々の BGM がある。本データは BGM の音声ファイル、音量、メモを定義。

`RCG_Asset<RCG_BGMData>` を継承。

## エディタ上の見た目

```
RCG_BGMData: <ID>
    BGM        ← 音声ファイル（RCG_AudioData、デフォルト GroupBGM）
    Volume     ← 音量
    Note       ← メモ
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **BGM** | はい | 音声ファイル、デフォルト `BGM_MapBGM` |
| **Volume** | はい | 音量 0~1（slider）、デフォルト 1.0 |
| **Note** | いいえ | メモ（編集時自用） |

## 動作説明

### `PlayBGM()` vs `PushBGM()`
*   **PlayBGM**：現 BGM を置換（旧停止、新再生）。
*   **PushBGM**：BGM stack にプッシュ（「戦闘開始時に戦闘音楽再生、終了で pop して元のマップ音楽に戻る」によく使用）。
両方に async 版あり（`PlayBGMAsync` / `PushBGMAsync`）。

### `m_BGM.IsEmpty` 時静黙 return
LogError なし；clip 読込失敗（IsEmpty=false だが GetClip が null 返却）時のみ LogError。

### デフォルト ID
`RCG_BGMData.DefaultBGM = "BGM_MapBGM"`：マップデフォルト BGM。

## 注意事項

*   **このファイルに対応する PopBGM はない**：pop ロジックは `UCL_GameAudioService` 自身で処理（外層が `PopBGM` 呼出）。
*   **BGM と SE は異なるミキサー**：各々独立音量制御；誤パスは SE 扱いになる。
*   **`Preview` override なし**：基底クラスデフォルト描画使用、エディタ内で音声波形プレビュー見えない。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BGMData.cs`
*   **継承**：`RCG_Asset<RCG_BGMData>`
*   **AssetGroup**：`EditGameSetting`
*   **定数**：`DefaultBGM = "BGM_MapBGM"`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_BGM` | BGM | `RCG_AudioData` | デフォルト `ModResource + GroupBGM + DefaultBGM` |
| `m_Volume` | Volume | `float` | `[UCL_Slider(0, 1)]` |
| `m_Note` | Note | `string` | |

### A.3 主要メソッド

*   **`PlayBGM() / PlayBGMAsync()`** — 現 BGM を置換。
*   **`PushBGM() / PushBGMAsync()`** — スタック再生。

### A.4 他システムとの連携

*   **`UCL.Core.Game.UCL_GameAudioService`** — 実再生サービス（`PlayBGM` / `PushBGM`）。
*   **`RCG_AudioData`** — 音声ファイルリソースラッパー。
*   **`RCG_BGMGenData`** — Asset Entry。
