---
title: 解放データ (RCG_UnlockData)
description: 汎用解放条件設定（カード / 装備 / アイテム / スキル / システム機能）；祝福ショップ解放と Steam 成就連携を含む
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 解放データ

> クラス名：`RCG_UnlockData`

## 用途

**汎用の解放条件設定**。複数のデータ（カード / 装備 / アイテム / スキル / キャラ / 大マップ / デッキ / 主動能力）が同一の `RCG_UnlockData` を参照して解放条件を共有可能 — 例：「Lv5 到達時に初級カードと装備のセットを一括解放」は1つの UnlockData を構築し、関連 Asset の `m_Unlock.ID = "Lv5_Unlock"` のみで OK。

「祝福ショップ解放」（解放後にも祝福で購入必要）と「システム機能解放」（解放後に最終関 / 禁卡 / キャンプファイア強化等を有効化）もサポート。

`RCG_Asset<RCG_UnlockData>` を継承。

## エディタ上の見た目

```
RCG_UnlockData: <ID>
    UnlockSetting       ← 解放条件本体（UnlockType: Level / SkillTagLevel / None / Tutorial / Achievement / Never）
    IsBlessingShop      ← 祝福ショップ経由で解放するか
    IsHiddenInCodex     ← 図鑑非表示
    SystemUnlockItems   ← この Asset 解放時に有効化するシステム機能（FinalLevel / BanCard / Inheritance...）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **UnlockSetting** | はい | 解放条件主設定 |
| **IsBlessingShop** | — | true：解放条件達成後にも祝福ショップで購入必要 |
| **IsHiddenInCodex** | — | 図鑑非表示（未解放対象名を露出回避） |
| **SystemUnlockItems** | いいえ | 一同で有効化するシステム機能一覧（`SystemUnlockItem` enum） |

`UnlockSetting` の内訳：

| サブフィールド | 説明 |
|---|---|
| **UnlockType** | `Level`（プレイヤーレベル）/ `SkillTagLevel`（職業レベル）/ `None`（直接解放）/ `Tutorial`（チュートリアルクリア）/ `Achievement`（特定成就）/ `Never`（永不解放） |
| **SkillTag** | UnlockType=SkillTagLevel 時の職業 |
| **UnlockLevel** | UnlockType=Level/SkillTagLevel 時のレベル閾値 |
| **Achievement** | UnlockType=Achievement 時の成就 ID |

## 動作説明

### `CheckUnlock()` (static)
全 UnlockData をスキャン：
1. 既に `RCG_GameRecord.UnlockRecords` 内 → スキップ。
2. `UnlockType.None` → 直接解放記録 + `SystemUnlockItems` 適用、ただしポップアップ表示しない。
3. その他タイプ：`!IsLocked()` → 記録 + SystemUnlockItems 適用；非祝福ショップアイテムは返却 set に追加（上層で解放通知をポップ）。
返却は「今回新解放かつ非祝福ショップの ID 集合」、UI 用に「新解放アイテム」表示。

### `GetUnloackables<T>(id)` (generic static)
`RCGI_Unloackable` を実装する全 Asset 中で `UnlockEntry.ID == id` のものを問合せ；この UnlockData がどのカード / 装備 / アイテム / スキル / キャラ / 大マップ / デッキ / 主動能力に対応するかに使用。

### `IsLocked()` (UnlockSetting)
*   `Level` → プレイヤーレベル < `m_UnlockLevel`。
*   `SkillTagLevel` → 対応職業レベル < `m_UnlockLevel`。
*   `None` → false（解放済）。
*   `Tutorial` → true（この層は常に true 返却、実解放判定は GameRecord の Tutorial 集合）。
*   `Achievement` → 該成就が未解放。
*   `Never` → true。

### Editor プレビュー
4 カテゴリの対応 unlockable 表示：CardData / ItemData / EquipmentData / UnitSkillData、ページング切替可。BigMap / Character はスキップ。

## 注意事項

*   **`Tutorial` UnlockType は IsLocked で常に true 返却**：実解放パスは `RCG_GameRecord.Ins.UnlockedCharacters` 等の記録経由、`IsLocked` 依拠でない。
*   **`IsBlessingShop = true` の解放**は二段階：条件達成 → ショップ販売解放；プレイヤー購入 → 真に使用可（`RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments` に記録）。
*   **`SystemUnlockItems` は enum でハードコード**：FinalLevel / BanCard / Inheritance LV1/2 / RestPointEnhancement LV1/2 / Blessing_Shop LV1/2/3 / Secret_Base、新機能追加には enum + プログラム対応点改修必要。
*   **Disabled 解放**：`UnlockEditor` が解放済記録を「無効化」するスイッチを提供、`DisabledUnlockRecord` 集合に追加（debug / テスト用）。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnlockData.cs`
*   **継承**：`RCG_Asset<RCG_UnlockData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_UnlockSetting` | UnlockSetting | `UnlockSetting` | 条件主設定 |
| `m_IsBlessingShop` | IsBlessingShop | `bool` | |
| `m_IsHiddenInCodex` | IsHiddenInCodex | `bool` | |
| `m_SystemUnlockItems` | SystemUnlockItems | `List<SystemUnlockItem>` | enum：10 種システム機能 |

### A.3 主要メソッド

*   **`CheckUnlock()` (static)** — 全 UnlockData スキャンし解放記録更新。
*   **`GetUnloackables<T>(id)` (generic static)** — この UnlockData を参照する全 Asset 対応。
*   **`GetLockedCards / GetLockedEquipments / ...` (instance)** — 各タイプのショートカット問合せ。
*   **`UnlockSetting.IsLocked()`** — 条件判定。
*   **`UnlockSetting.GetShortName()`** — 解放説明文字列を自動生成。
*   **`RCG_UnlockEntry.Unlocked / IsShopItem / Name / UnlockType`** — このデータを参照する entry の helper。

### A.4 他システムとの連携

*   **`RCG_GameRecord.Ins.UnlockRecords`** — 解放済記録集。
*   **`RCG_GameRecord.UnlockedCards / UnlockedItems / UnlockedEquipments / UnlockedCharacters`** — ショップ購入記録。
*   **`RCG_GameAchievement.Ins.IsAchievementUnlocked`** — Achievement タイプの解放チェック。
*   **`RCG_GameRecord.DisabledUnlockRecord`** — debug 解放無効化。
*   **`RCG_DataService.Ins.m_GameData.m_UnlockedSystemUnlocks`** — システム機能解放記録。
*   **`UCLI_Asset.GetUtilByType`** — リフレクションで対応型の Asset Util 取得。

### A.5 既知の問題

*   `// TODO: Add Deck QWQ234!!` — プレビュー UI が Deck / Character の2カテゴリを欠落、列挙不完全。
*   `RCG_ShopUnlockData` / `CommonItemGenData` は同ファイルの補助構造、RCG_UnlockData と共に使用。
