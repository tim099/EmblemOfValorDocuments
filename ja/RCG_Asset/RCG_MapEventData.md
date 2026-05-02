---
title: マップイベントデータ (RCG_MapEventData)
description: 小マップノードで発動可能なイベントパッケージ：会話、過場、アイテム付与、戦闘前奏等
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# マップイベントデータ

> クラス名：`RCG_MapEventData`

## 用途

**小マップノードで発動可能なイベントパッケージ**。例：「神秘ノード」を踏むと会話 → 選択 → 報酬付与；「キャンプファイア」は直接回復メニュー；「商人」はショップを開く。各 `RCG_MapEventData` は1セットの `RCG_MapEvent` のコンテナ；発動時に順次イベント管理者に追加。

`RCG_Asset<RCG_MapEventData>` を継承。

## エディタ上の見た目

```
RCG_MapEventData: <ID>
    Name                     ← イベント表示名（多言語）
    Icon                     ← カスタムアイコン（空白で最初の Event のデフォルトアイコン使用）
    CanTriggerRepeatedly     ← 繰返し発動可能か
    Events                   ← 実イベント一覧（RCG_MapEvent）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | いいえ | 表示名（多言語）；空白時は最初の Event の ShortName にフォールバック |
| **Icon** | いいえ | アイコン；空白時は最初の Event の `DefaultIcon` 使用 |
| **CanTriggerRepeatedly** | — | 繰返し発動可能か；false → 発動後にノード上のイベントクリア |
| **Events** | はい | 実イベント一覧（`List<RCG_MapEvent>`） |

## 動作説明

### 発動 (`StartEvent(node)`)
各 `RCG_MapEvent` を順次 clone し `RCG_MapEventManager` に追加。直接使用ではなく Clone なのは、同 Asset を複数回発動する時に各回が独立 instance であることを保証するため。

### アイコンと名前の fallback
*   `Icon`：`m_Icon` 空 → `m_Events[0].DefaultIcon` → null。
*   `LocalizedName`：`m_Name` あり → 使用；なければ → `m_Events[0].GetShortName()`。
*   `EventHoverTips`：`LocalizedName`（マウスがノード上にある時のチップ）。

## 注意事項

*   **`CanTriggerRepeatedly = false`** で発動後にノード上のイベントクリア：「一度きりのイベント」如劇情キーポイント、固有報酬向き。
*   **Events 空の時**：アイコンと名前の fallback 失敗（null / 空文字列返却）、UI 上はイベントなしのように見える。
*   **デフォルト ID `Start`**：通常はスタートノード専用イベントとして使用。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MapEventData.cs`
*   **継承**：`RCG_Asset<RCG_MapEventData>`
*   **AssetGroup**：`EditQuestSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Icon` | Icon | `RCG_SpriteData` | Addressable デフォルト |
| `m_CanTriggerRepeatedly` | CanTriggerRepeatedly | `bool` | デフォルト `false` |
| `m_Events` | Events | `List<RCG_MapEvent>` | |

### A.3 主要メソッド

*   **`StartEvent(RCG_MapNode)`** — 主発動入口；各 event を clone し manager に追加。
*   **`Icon` (property)** — fallback ロジック含む。
*   **`LocalizedName / EventHoverTips`** — 表示用プロパティ。

### A.4 他システムとの連携

*   **`RCG_MapEvent`** — 実イベント単位（会話 / 選択 / 報酬 / 戦闘前奏）。
*   **`RCG_MapEventManager`** — runtime イベントキュー管理。
*   **`RCG_MapNode`** — 発動ソースノード。
*   **`RCG_MapEventGenData`** — Asset Entry；デフォルト ID = `"Start"`。
