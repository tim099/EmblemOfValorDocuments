---
title: アイテムデータ (RCG_ItemData)
description: プレイヤー使用可能な消耗 / クエストアイテムテンプレート（ポーション、巻物、鍵等）。レアリティ、効果、使用回数、解放
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# アイテムデータ

> クラス名：`RCG_ItemData`

## 用途

**プレイヤーが所持 / 使用するアイテムテンプレート**。ポーション、巻物、食材、クエスト鍵、回復アイテム等、すべてこのクラスのインスタンス。各アイテムが独自のアイコン、効果、レアリティ、使用回数制限、解放条件を持つ。

`RCG_Asset<RCG_ItemData>` を継承。実装：`RCGI_Item`（バックパック追加可）/ `RCGI_Unloackable`（解放可）。

## エディタ上の見た目

```
RCG_ItemData: <ID>
    Data (m_Data)              ← 主要設定（入れ子クラス）
    Effects (m_ItemEffects)    ← アイテム効果（OnPlay 発動）
    Preview                    ← リアルタイムプレビュー
```

## 主要フィールド（Data 内部）

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | アイテム名（多言語） |
| **Description** | いいえ | フレーバー説明（多言語、自動説明の後ろに付く） |
| **TargetType** | はい | 使用時のターゲット選択範囲（None / Friend / Enemy 等） |
| **Price** | はい | 商人売価；デフォルト 20、`Auto Price` ボタンで再計算可（基礎 3 × レアリティ価値） |
| **Icon** | はい | アイテムアイコン |
| **Rarity** | はい | レアリティタグ |
| **ItemType** | はい | `Normal`（普通）または `QuestItem`（クエスト用、ストーリー用途） |
| **ItemUseType** | はい | `Consume`（使用後消失）/ `OncePerBattle`（戦闘ごと1回）/ `OncePerTurn`（ターンごと1回）/ `Infinite`（無限回） |
| **UseTimes** | ItemUseType 依存 | 使用可能回数（Consume / OncePer* で有効） |
| **Unlock** | いいえ | 解放条件 |
| **HideInCodex** | — | 図鑑非表示（テスト用アイテム） |

外層に **m_ItemEffects** = `List<RCG_CommonEffect>`、使用時の発動効果（OnPlay が主 trigger）を記述。

## 動作説明

### 使用フロー
1. プレイヤーがバックパックからアイテム選択 → `RCG_ItemInfoPanel` 突入。
2. `CheckUsable(data)`：各 effect で `CheckPlayable` 実行、全て true で使用可能。
3. `TriggerEffect(data)`：全 `OnPlay` effect を取得し順次発動（try-catch 防爆あり）。
4. `ItemUseType` に応じて消失 / 次ターンまでロック / 次戦闘までロック。

### 説明生成
*   `Effects` から自動連結（各 effect 1行）。
*   付加：非 Normal の ItemType マーク、非 Consume の UseType マーク。
*   `UseTimes > 1` 時に「残り N/M 回」説明追加（runtime のみ表示）。
*   `m_Description` 設定時 → 自動説明の下にフレーバー文追加。

### Auto Price（エディタツール）
エディタの `RCG_ItemDataEditorPage` 上方に `Auto Price` ボタン：全 ItemData に対し `Price = 3 × Rarity.m_Value` を適用。**手動設定価格を上書き**。

## 注意事項

*   **ItemUseType = Infinite 時 UseTime は 0 を返す**：runtime で消費 / ロックなし。
*   **`m_Description` は自動説明と重畳**：手書き部分は効果の後に表示。
*   **QuestItem は通常使用フローで消耗されない**：通常はストーリーイベント連動で発動。
*   **Auto Price は手動価格を消去** — 押下前に確認。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ItemData.cs`
*   **継承**：`RCG_Asset<RCG_ItemData>`
*   **実装**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 フィールドマッピング（外層）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Data` | Data | `ItemData`（入れ子） | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ItemData` 入れ子内に `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_Rarity` / `m_ItemType` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock` / `m_HideInCodex`。

### A.3 主要メソッド

*   **`AddItem()`** — プレイヤー取得：`RCG_Item.Create(ID)` → `m_ItemsData.AddItem`。
*   **`CheckUsable(TriggerEffectData)`** — 全 enabled effects の `CheckPlayable` を AND 取得。
*   **`TriggerEffect(TriggerEffectData)`** — `OnPlay` effects 取得し順次発動（try-catch）。
*   **`Description` / `FullDescription` / `GetDescription`** — 三層説明生成。
*   **`Effects` (property)** — `m_ItemEffects.GetEnableEffects()`。
*   **`Infos` (property)** — effect infos + ItemType 説明を集約。
*   **`CreateSelectAssetPage`** — `RCG_ItemDataEditorPage.Create()`。

### A.4 他システムとの連携

*   **`RCG_Item`** — runtime アイテムインスタンス。
*   **`RCG_DataService.Ins.m_ItemsData`** — プレイヤーバックパック保存。
*   **`RCG_ItemDataEditorPage`** — 編集メイン画面（Auto Price ツール含む）。
*   **`RCG_ItemInfoPanel`** — 詳細情報 UI。

### A.5 既知の問題

*   `m_UseTimes` の「暫時未完成 先隱藏」コメントは、可変使用回数機能がまだ完全に実装されていないことを示唆。
*   `SerializeToJson` / `DeserializeFromJson` override はコメントアウト済；自動価格移行ロジックは以前計画されていた。
