---
title: 装備データ (RCG_EquipmentData)
description: キャラ装着装備テンプレート（武器、防具、アクセサリー、レリック）。タイプ、レアリティ、効果、強化分岐、非重複ドロップ
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 装備データ

> クラス名：`RCG_EquipmentData`

## 用途

**キャラが装着する装備のテンプレート**。武器、防具、アクセサリー、レリック（特殊加成物品）すべて。各装備が独自のレアリティ、タイプ、効果（戦闘中の各 trigger で発動）、強化分岐、重複ドロップ可否を持つ。

`RCG_Asset<RCG_EquipmentData>` を継承。実装：`RCGI_Item` / `RCGI_Unloackable`。

## エディタ上の見た目

```
RCG_EquipmentData: <ID>
    Name / Description / Rarity / Tags / EquipmentType / Icon / Price
    Effects                    ← 戦闘中に発動する効果
    InitCounters               ← 初期カウンター
    SkillTags                  ← 職業専用制限
    Unlock                     ← 解放条件
    UpgradeBranch              ← どの装備に強化可能か
    EnhenceLevel / HideInCodex / CanDropRepeatedly
    Preview
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | 装備名（多言語） |
| **Description** | いいえ | フレーバー説明 |
| **Rarity** | はい | レアリティ |
| **Tags** | いいえ | 一般タグ（DropPool 用） |
| **EquipmentType** | はい | `Weapon` / `Armor` / `Accessory` / `Relic`（**レリック：装備枠を消費せず、メインキャラに強制装着**） |
| **Icon** | はい | 装備アイコン |
| **Price** | はい | ショップ売価；デフォルト 500、`Auto Price` ボタンで再計算可（基礎 10 × レアリティ価値） |
| **Effects** | いいえ | 戦闘中の各 trigger で発動する効果 |
| **InitCounters** | いいえ | effect 内のカウンター系効果用初期値 |
| **SkillTags** | いいえ | 職業専用制限（パーティに対応職業がいる時のみドロップ；OR 関係：いずれか適合） |
| **Unlock** | いいえ | 解放条件 |
| **UpgradeBranch** | いいえ | 強化可能な装備一覧（ゲーム内強化メカニクス経由）；空 = 強化不可 |
| **EnhenceLevel** | — | 強化レベル > 0 は「強化版」を意味、**図鑑非表示** |
| **HideInCodex** | — | 図鑑非表示（テスト / 強化版は自動非表示） |
| **CanDropRepeatedly** | — | 重複ドロップ可能か；レリック (`Relic`) は通常 false |

## 動作説明

### 発動
`OnTriggerEffect(data, triggerOn)` → `m_Effects` から該 trigger の effects を取得し順次発動。`TriggerOnUnitState(triggerOn)` は quick check。

### 図鑑非表示判定
`HideInCodex` (property) = `m_HideInCodex || m_EnhenceLevel > 0`：強化版は自動非表示、図鑑に「+1 / +2」等の同名重複出現を回避。

### 説明
`GetFullDescription` = `Effects 説明 + m_Description（フレーバー文）`。

### Auto Price
エディタページに `Auto Price` ボタン：全装備に `Price = 10 × Rarity.m_Value` を適用。

### ソート
`RCG_EquimpmentComparer` は「装備タイプ（Weapon=10, Armor=20, Accessory=30）」→「価格降順」でソート。

## 注意事項

*   **レリック (`Relic`) ルール**：装備枠不要、メインキャラ強制装着、`CanDropRepeatedly` 通常 false（DropPool が身上の所持確認）。
*   **`SkillTags` とカード専門性の違い**：装備は OR 関係（いずれかの隊員が持てばドロップ可）、カードの `RequireSkills` は AND。
*   **強化版 (`EnhenceLevel > 0`) は図鑑自動非表示**：図鑑には原版のみ表示。
*   **`NullEquipmentID = "NullEquipment"`** はシステム予約 ID、「装備なし」を意味；一般装備の命名に使わない。
*   **Auto Price は手動価格を消去**。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_EquipmentData.cs`
*   **継承**：`RCG_Asset<RCG_EquipmentData>`
*   **実装**：`RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**：`EditItems`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | |
| `m_EquipmentType` | EquipmentType | `EquipmentType` enum | デフォルト `Weapon` |
| `m_Icon` | Icon | `RCG_SpriteData` | |
| `m_Price` | Price | `int` | デフォルト 500 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_InitCounters` | InitCounters | `List<int>` | |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_UpgradeBranch` | UpgradeBranch | `List<RCG_EquipmentGenData>` | |
| `m_EnhenceLevel` | EnhenceLevel | `int` | デフォルト 0 |
| `m_HideInCodex` | HideInCodex | `bool` | |
| `m_CanDropRepeatedly` | CanDropRepeatedly | `bool` | デフォルト `true` |

### A.3 主要メソッド

*   **`AddItem()`** — プレイヤー取得：`m_EquipmentsData.AddEquipment(new RCG_Equipment(ID))`。
*   **`OnTriggerEffect(data, triggerOn)`** — 戦闘発動。
*   **`HideInCodex` (property)** — `m_HideInCodex || m_EnhenceLevel > 0`。
*   **`GetDescription` / `GetFullDescription`** — 説明生成。
*   **`CreateSelectAssetPage`** — `RCG_EquipmentDataEditorPage.Create()`。

### A.4 他システムとの連携

*   **`RCG_Equipment`** — runtime 装備インスタンス。
*   **`RCG_DataService.Ins.m_EquipmentsData`** — プレイヤー装備保存。
*   **`RCG_EquipmentDropPool`** — ドロップ池（`m_CanDropRepeatedly` チェック含む）。
*   **`RCG_EquipmentInfoPanel`** — 詳細情報 UI。
*   **`RCG_EquimpmentComparer`** — ソート helper。

### A.5 既知の問題

*   `m_CanDropRepeatedly` の TODO コメント（"需要記錄掉落&排除"）は「非重複ドロップ」記録メカニズムの履歴改修を示唆；現在は DropPool runtime でプレイヤー装備一覧と比較して実現。
*   `DeserializeFromJson` の自動価格 `m_Price = m_Rarity.GetData().m_Value.GetValue(null) * 15` はコメントアウト、Auto Price ツールで置換。
