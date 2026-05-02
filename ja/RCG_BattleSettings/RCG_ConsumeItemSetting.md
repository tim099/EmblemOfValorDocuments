---
title: アイテムを消費する
description: プレイヤーのインベントリから指定アイテムを消費；「アイテム消費でプレイ可能」のカード条件としても使える
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# アイテムを消費する

> クラス名：`RCG_ConsumeItemSetting`

## 用途
**プレイヤーのインベントリにあるアイテムを消費**。「**プレイ条件**」としても機能 — 対応アイテムを持っていなければカードがプレイ不可。よくある用途：
*   「火薬包を 1 個消費して AOE 火炎ダメージ」
*   「薬草を 2 個消費してパーティ全体回復」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **アイテム** (`Items`) | はい | 消費するアイテムリスト；各項目は `RCG_ItemGenData`（アイテムテンプレート）。 |

## 挙動
*   **プレイ判定**：インベントリに全リストアイテムが揃っている必要（ID 一致；**同 ID が複数なら複数個必要**）。
*   **発動**：リストに従って各アイテムを順次消費。
*   説明：消費アイテム名を全て列挙。

## 注意点
*   **TODO 注意**：現状**リソース変化時にカードのプレイ可否が即時更新されない** — `Todo:資源變化時 要刷新卡牌才能正確判斷目前是否能打出` のコメントあり。
*   **空リスト**：合法だが消費なし — このカードは常にプレイ可能で何も消費しない；データエラー扱い。
*   **同 ID アイテムの繰り返し消費**：リスト内に同 ID が 2 回出現すると 2 個消費；インベントリに 2 個未満ならプレイ不可。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeItemSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_ConsumeItemSetting` → 「アイテムを消費する」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Items` | アイテム | `List<RCG_ItemGenData>` | `Items` | コメント「獲得的資源(金幣等..)」は古いコピペ残存、実際は消費用 |

### A.3 主なメソッド
*   **`CheckPlayable`**：`RCG_DataService.Ins.m_ItemsData.GetItems()` を clone し、`aItem.ID` で順次マッチして RemoveAt；見つからなければ即 false。
*   **`Infos`**：base.Infos + 各 `m_Items[i].GetData()` の情報。
*   **`AddAction`**（ファイル 80 行外）：実際にインベントリからアイテムを差し引き。

### A.4 他システムとの連携
*   **`RCG_DataService.Ins.m_ItemsData`**：プレイヤーインベントリデータソース。
*   **`RCG_ItemGenData / RCG_Item`**：アイテムテンプレートとインスタンス。

### A.5 既知の課題
*   コード内 `Todo:資源變化時 要刷新卡牌才能正確判斷目前是否能打出` — カードのプレイ状態が即時更新されない問題。
*   フィールドコメント「獲得的資源」は copy-paste の名残、実際は「消費するアイテム」用。
