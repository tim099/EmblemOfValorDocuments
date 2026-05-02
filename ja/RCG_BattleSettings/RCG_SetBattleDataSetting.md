---
title: 戦闘データを設定
description: 戦闘段階のグローバル変数値を変更（戦闘内共有データ）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘データを設定

> クラス名：`RCG_SetBattleDataSetting`

## 用途
**戦闘段階の共有データを変更**（`RCG_BattleManager.BattleData`）— このバトル中に設定間で共有される数値コンテナ、「戦闘内グローバル変数」のような存在。よくある用途：
*   「累計ダメージを記録」
*   「プレイヤーが本戦闘で何回特定アクションを行ったかを記録」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **RuntimeDataOperator** | はい | ネストの `RCG_RuntimeDataSetter`（設定するフィールド + 値 + 計算式を指定）。デフォルトは `RCG_RuntimeStructGenData.BattleDataID`（戦闘データ構造）にバインド。 |

## 挙動
*   発動時に `RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)` を呼び、計算結果を指定フィールドに書き込み。
*   説明は完全に `m_RuntimeDataOperator.GetDescription` に委譲。
*   `GetShortName` のデフォルトは `"SetBattleData:{operator.GetShortName()}"`。

## 注意点
*   **データ構造を事前に定義する必要**：`RuntimeDataOperator` がどのフィールドを設定するかは `RCG_RuntimeStructGenData.BattleDataID` 構造に依存 — 新フィールドの追加にはその構造を確認。
*   **フュージョンしない**：本設定はカードフュージョンに参加しない（`GetFusionBaseSetting` は null を返す）。
*   **戦闘終了でデータ消失**：BattleData は戦闘内のもの；バトル間の記録は「**リソース変化**」または globalな RuntimeData を使ってください。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetBattleDataSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_SetBattleDataSetting` → 「戦闘データを設定」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_RuntimeDataOperator` | `RuntimeDataOperator` | `RCG_RuntimeDataSetter` | — | デフォルトで `RCG_RuntimeStructGenData.BattleDataID` にバインド |

### A.3 主なメソッド
*   **`AddAction`**：`RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)`。
*   **`GetDescription / GetDescriptionFormat / GetShortName`**：`m_RuntimeDataOperator` に委譲。
*   **`GetFusionCandidateSettings`** → 空リスト；**`GetFusionBaseSetting`** → null（フュージョン不参加）。

### A.4 他システムとの連携
*   **`RCG_BattleManager.BattleData`**：戦闘内共有データコンテナ。
*   **`RCG_RuntimeDataSetter`**：フィールド設定器、計算式付き。
*   **`RCG_RuntimeStructGenData.BattleDataID`**：対応するデータ構造テンプレート。
