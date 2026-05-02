---
title: カードをアイテムに変える
description: 選択した手札をアイテム化（「このカードをプレイ」効果のアイテム生成）；臨時 / 永久を選択可能
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カードをアイテムに変える

> クラス名：`RCG_CardToItemSetting`

## 用途
**カードをインベントリのアイテムに変換**。アイテムの使用効果は「**このカードを手札に生成しプレイ**」。よくある用途：
*   「攻撃カードをポーションとして保存」
*   「カード化アイテム」風の特殊メカニズム

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SelectHandCardSetting** | はい | アイテム化対象を選ぶネスト選択設定；条件・枚数・フィルタを設定。 |
| **IsTmpItem** | — | デフォルト ON。ON = 戦闘終了で自動削除；OFF = 永久保持。 |

## 挙動
*   先に `SelectHandCardSetting` でプレイヤーに選択させる。
*   選択された各カードに対し：
    1. 新 `RCG_ItemData` を clone（名前・アイコンはカードから流用）。
    2. アイテムの「使用効果」に `RCG_CardCreateSetting`（このカードを生成して手札に追加）を自動書込。
    3. チャントカードは `IsChanted` を引き継ぎ、二重詠唱を回避。
    4. `BattleRuntime` データとして登録し、インベントリに追加。
*   元カードは消滅。
*   説明形式：`IsTmpItem=true` → `CardToTmpItem_Des`；他 → `CardToItem_Des`。

## 注意点
*   **臨時 vs 永久アイテムのセーブ**：臨時アイテムは戦闘終了でクリア；永久アイテムはプレイヤーインベントリに保存され戦闘間で持続 — 大量の永久アイテム化はインベントリを爆発させる、上限を設計を。
*   **アイテム化後はフュージョン不可**：元カードは消滅；「カードフュージョン + アイテム化」はフュージョンしてからアイテム化を。
*   **「カードフュージョン」との違い**：フュージョンは複数を合成して新カードに；本設定は「**カード → アイテム**」一方向変換。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardToItemSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CardToItemSetting` → 「カードをアイテムに変える」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | |
| `m_IsTmpItem` | `IsTmpItem` | `bool` | `IsTmpItem` (=「臨時アイテム?」) | デフォルト `true` |

### A.3 主なメソッド
*   **`AddAction`**：選択後、各カードに対し：
    1. `RCG_ItemData` を構築（`Data.m_Name = m_LocalizeName`, `Data.m_Icon = m_CardIcon`, `ID = aCard.ID`）。
    2. `RCG_CommonEffect`（`OnPlay`）→ `RCG_CardCreateSetting`（`AddToHandCard`, `m_RuntimeCardData = new RCG_CardBattleDataPointer(aCard)`）をネスト。
    3. チャントカード → `aSetting.m_IsChanted = true`。
    4. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.BattleRuntime)`。
    5. `new RCG_Item(aItemData, RCG_Item.ItemType.Runtime)` + `m_TmpItem = m_IsTmpItem` + `AddItem()`。
*   **`Info`**：`m_IsTmpItem ? CardInfoData.TmpItemInfo : null`。

### A.4 他システムとの連携
*   **`RCG_CardCreateSetting` + `RCG_CardBattleDataPointer`**：アイテムの使用効果は「記憶したカードをプレイ」のパターン。
*   **`DataType.BattleRuntime`**：戦闘終了で自動クリアのデータコンテナ。
*   **`RCG_AquireItemPanel`**：UI フィードバック（ファイルの 80 行外の続き）。
