---
title: アクティブパワー (RCG_ActivePowerData)
description: キャラの「アクティブ能力」：手動発動可能な能力（アイテム類似だがバックパックでなくキャラに紐付く）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# アクティブパワー

> クラス名：`RCG_ActivePowerData`

## 用途

**キャラの「アクティブ能力」テンプレート**。「装備」と「アイテム」の中間：プレイヤーが戦闘中に**手動発動**できる能力（パッシブと違い）、ただし**特定キャラに紐付く**（アイテムと違い共通バックパックではない）。例：「治療師：戦闘ごと1回、グループ治療」「剣士：ターンごと手動でクリティカル発動」。

`RCG_Asset<RCG_ActivePowerData>` を継承。実装：`RCGI_Item` / `RCGI_Unloackable`。

## エディタ上の見た目

```
RCG_ActivePowerData: <ID>
    Data (m_Data)               ← 主要設定（Name / Description / Icon / TargetType / Price / Unlock / ItemUseType）
    ItemEffects                 ← アクティブパワーの効果（OnPlay 発動）
    Preview
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name / Description / Icon** | はい | 名前、フレーバー説明、アイコン |
| **TargetType** | はい | 使用時のターゲット選択範囲 |
| **Price** | はい | ショップ価格（購入可能なら） |
| **ItemUseType** | はい | `Consume` / `OncePerBattle` / `OncePerTurn` / `Infinite` |
| **UseTimes** | ItemUseType 依存 | 使用可能回数 |
| **Unlock** | いいえ | 解放条件 |
| **ItemEffects** | いいえ | 発動効果（`RCG_CommonEffect` リスト） |

## 動作説明

### アイテムとの違い
*   **アイテム**：共通バックパック、どのキャラでも使用可能。
*   **アクティブパワー**：特定キャラに紐付く（`RCG_DataService.Ins.m_ActivePowersData`）、通常はキャラパーティ加入と共に持参。

### 使用 / 発動
`CheckUsable(data)` → 全 effect の `CheckPlayable` 全 true で使用可能。
`TriggerEffect(data)` → `OnPlay` effects 取得し順次発動（try-catch あり）。

### 説明生成
`RCG_ItemData` 類似：effects から自動連結 + UseType マーク + UseTimes 表示。

## 注意事項

*   **`ShowInfo()` は空殻**：本来 `RCG_ItemInfoPanel` を起動すべきだったが、コメントアウト済（`// QWQ`）、現状無効。実 UI は別箇所で表示。
*   **ItemType フィールドなし**（`RCG_ItemData` と異なる）：既に「アクティブパワー」分類のため、Normal / QuestItem の細分化は不要。
*   **`InitActivePowers` の旧コメント**：本来キャラ加入時に InitActivePowers を自動適用していたが、UnitSkill システムに置換済（プログラム内に大段のコメントあり）。
*   **Auto Price ツールなし**：ItemData / EquipmentData と異なり、エディタページに無い。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ActivePowerData.cs`
*   **継承**：`RCG_Asset<RCG_ActivePowerData>`
*   **実装**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`

### A.2 フィールドマッピング（外層）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Data` | Data | `ActivePowerData`（入れ子） | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ActivePowerData` 入れ子に `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock`。

### A.3 主要メソッド

*   **`AddItem()`** — `RCG_ActivePower.Create(ID)` → `m_ActivePowersData.AddActivePower`。
*   **`CheckUsable(TriggerEffectData)`** — 全 effect AND check。
*   **`TriggerEffect(TriggerEffectData)`** — `OnPlay` effects 発動。
*   **`Description` / `FullDescription` / `GetDescription`** — 説明生成（`RCG_ActivePower` 任意パラメータで runtime 残回数を持参）。
*   **`UseTime`** — `ItemUseType` に応じて回数返却（Infinite → 0）。

### A.4 他システムとの連携

*   **`RCG_ActivePower`** — runtime アクティブパワーインスタンス。
*   **`RCG_DataService.Ins.m_ActivePowersData`** — キャラ紐付け保存。
*   **`RCG_CommonEffect`** — 発動効果単位。

### A.5 既知の問題

*   `ShowInfo()` はコメントアウト（`// QWQ`）、UI 入口要確認。
*   旧 `InitActivePowers` 自動加成ロジックは廃止（UnitSkill で置換）。
