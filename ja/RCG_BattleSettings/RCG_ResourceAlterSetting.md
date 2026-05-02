---
title: リソース変化
description: 戦闘中にリソース（ゴールド、魂力等）を獲得 / 消費
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# リソース変化

> クラス名：`RCG_ResourceAlterSetting`

## 用途
**戦闘中にプレイヤー保有のリソースを変化**（ゴールド、魂力など）。よくある用途：
*   「戦闘中に 50 ゴールド獲得」
*   「魂力 1 を消費して強力なカード発動」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **ResourceAlterMode** | はい | 変化モード：<br>• **Add** — 獲得（デフォルト）<br>• **Consume** — 消費（**プレイ判定が掛かる**） |
| **AcquiredResources** | はい | リソースリスト；各項目は `RCG_ResourceGenData`（リソースタイプ + 量）。 |

## 挙動
*   **プレイ判定**：`Consume` モードでは、各リソースの当前値が消費量 ≥ である必要。1 つでも不足するとカードがプレイ不可。
*   **発動**：モードに応じて `RCG_DataService.Ins` の対応リソースに加 / 減算。
*   説明：各リソースを列挙；`Consume` は `ConsumeItem` i18n、`Add` は `AcquireItem` i18n。

## 注意点
*   **TODO 注意**：リソース変化時にカードのプレイ可否が即時更新されない — 同ファイルに `Todo:資源變化時 要刷新卡牌才能正確判斷目前是否能打出` のコメントあり。
*   **常に展開**：`[AlwaysExpendOnGUI]`、Inspector 上で**折りたたまれない**。
*   **空リスト**：合法だが変化なしと等価。
*   **「エネルギー変化」との違い**：エネルギーは戦闘内で毎ターンリフレッシュされるリソース；本設定は**バトル間で持続するグローバルリソース**（ゴールド、魂力）— 混同しないように。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ResourceAlterSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n クラス名 key**：`RCG_ResourceAlterSetting` → 「リソース変化」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_ResourceAlterMode` | `ResourceAlterMode` | enum (内部) | — | `Add` / `Consume` |
| `m_AcquiredResources` | `AcquiredResources` | `List<RCG_ResourceGenData>` | — | |

### A.3 主なメソッド
*   **`CheckPlayable`**：`Consume` モードは各リソースを `RCG_DataService.Ins.GetResource(type)` で `m_Amount.GetValue` と比較、1 つでも不足で false；`Add` モードは恒 true。
*   **`AddAction`**（ファイル 100 行外）：モードに応じて各リソースに加減 API を呼ぶ。
*   **`Infos`**：base + 各リソースタイプの情報を `AddIfNotRepeat` で追加。
*   **`GetDescriptionFormat / Params`**（ファイル内）：リソース名を `, ` で結合、`Consume`/`Add` モードに応じて異なる i18n key を使う。

### A.4 他システムとの連携
*   **`RCG_DataService.Ins.GetResource(type)`**：当前リソース値の取得。
*   **`RCG_ResourceGenData / RCG_ResourceTypeGenData`**：リソーステンプレート。
