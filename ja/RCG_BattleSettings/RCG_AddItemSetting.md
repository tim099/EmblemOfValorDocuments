---
title: アイテム獲得
description: 指定アイテムをプレイヤーのインベントリに追加；臨時フラグで戦闘後自動削除も可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# アイテム獲得

> クラス名：`RCG_AddItemSetting`

## 用途
1 個以上の指定アイテムをプレイヤーのインベントリに追加。「バトル報酬カード」「ピックアップイベント」「臨時 Buff アイテム配布」などで使用。

## エディタ表示
```
▼ ✓ [アイテム獲得(AddItem)] (アイテムサムネ)
    アイテム      ▶ (アイテムリスト)
    IsTmpItem     [✓]
```

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **アイテム** | はい | 追加するアイテムリスト；各項目は `RCG_ItemGenData`（アイテムテンプレート）。 |
| **IsTmpItem** | — | デフォルト ON。ON = 戦闘終了で自動削除（永続インベントリに影響しない）；OFF = 永久保持。 |

## 挙動
*   発動時、アイテムを順次インベントリに追加し、「アイテム獲得」パネルでプレイヤーに表示。
*   IsTmpItem ON のアイテムは臨時フラグが付き、戦闘終了で自動クリーンアップ。
*   説明：「**{アイテム1, アイテム2, ...} 獲得**」、臨時アイテムには `[臨時アイテム]` 表記が下に追加。

## 注意点
*   **臨時アイテムのセーブリスク**：戦闘中セーブ→再ロード時の臨時アイテム削除はインベントリ系統が処理；持続効果を持つアイテムは特にロード時の挙動を確認。
*   **空リスト**：合法ですがアイテム配布なし — データエラー扱い。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_AddItemSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_AddItemSetting` → 「アイテム獲得」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Items` | アイテム | `List<RCG_ItemGenData>` | `Items` | アイテムテンプレートリスト |
| `m_IsTmpItem` | `IsTmpItem` | `bool` | `IsTmpItem` | デフォルト `true` |

### A.3 主なメソッド
*   **`Infos`**：各 `m_Items[i].GetData()` の情報を集約；`m_IsTmpItem` なら `CardInfoData.TmpItemInfo` を追加。
*   **`GetDescriptionFormat`**：アイテム名を `, ` で結合、臨時なら `[TmpItem]` を末尾追加、最後に i18n key `AcquireItem` でラップ。
*   **`AddAction`**：`AddAsyncActionTrigger` 内で `m_Items[i].GetData().AddItem()` を順次呼び、`m_TmpItem` フラグを設定、`RCG_AquireItemPanel` を表示。

### A.4 他システムとの連携
*   **`RCG_ItemGenData / RCG_Item`**：アイテムテンプレートとインスタンス。
*   **`UI.RCG_AquireItemPanel`**：アイテム獲得パネル UI。
*   **`CardInfoData.TmpItemInfo`**：静的「臨時アイテム」情報ブロック。
