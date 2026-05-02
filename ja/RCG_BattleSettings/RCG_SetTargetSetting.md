---
title: ターゲットを設定する
description: ルールに従って自動的にターゲットを再選択（UI なし）；既存のターゲットリストを上書き
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ターゲットを設定する

> クラス名：`RCG_SetTargetSetting`

## 用途
**自動的にルールに従ってターゲットを再選択**（UI なし）。よくある用途：
*   「攻撃を血量最低の敵にリダイレクト」
*   「後列ターゲットへリダイレクト」
*   AI トリガー効果が自動的にターゲットを指定する

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **UseOldSelectUnitRule** | — | デフォルト ON（前方互換）。ON は旧版 `RCG_SelectTargetData` を使用；OFF は新版 `RCG_TargetSelectRule` を使用。 |
| **SelectTarget** | UseOldSelectUnitRule=true 時 | 旧版ターゲットルール（`RCG_SelectTargetData`）。 |
| **SelectUnitRule** | UseOldSelectUnitRule=false 時 | 新版ターゲットルール（`RCG_TargetSelectRule`）。 |

## 挙動
*   設定したルールに基づいて自動的にターゲットリストを解析、`iData.Targets` を上書き。
*   UI を出さずプレイヤー入力不要；AI と自動化効果に最適。
*   説明には対応ルールの説明（短縮名）を表示。

## 注意点
*   **「対象選択」との違い**：本設定は**UI を出さない**；「対象選択」はプレイヤークリックを要求。
*   **新ルール vs 旧ルール**：新版 `RCG_TargetSelectRule` は表現力が強い；旧版 `RCG_SelectTargetData` は互換性のため保留。新データは新版を（`UseOldSelectUnitRule` をオフに）。
*   **ファイル内コメントに TODO**：プログラム内に `// TODO: Test QWQ2`、新版ルールの実際使用にはまだ検証が必要。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetTargetSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_SetTargetSetting` → 「ターゲットを設定する」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_UseOldSelectUnitRule` | `UseOldSelectUnitRule` | `bool` | — | デフォルト `true`（前方互換） |
| `m_SelectUnitRule` | `SelectUnitRule` | `RCG_TargetSelectRule` | — | `[Conditional("m_UseOldSelectUnitRule", false)]` |
| `m_SelectTarget` | `SelectTarget` | `RCG_SelectTargetData` | — | `[Conditional("m_UseOldSelectUnitRule", true)]` |

### A.3 主なメソッド
*   **`AddAction`** (async)：`m_UseOldSelectUnitRule` で `m_SelectUnitRule.SelectTargets(iData)` または `m_SelectTarget.GetTargets(iData)` を呼び、`iData.Targets` に代入。
*   **`GetDescription`**：対応するルールの `Description / GetShortName` に委譲。

### A.4 他システムとの連携
*   **`RCG_TargetSelectRule`**：新版ターゲットルールコンテナ。
*   **`RCG_SelectTargetData`**：旧版セレクタ；他の設定に広く使われている。

### A.5 既知の課題
*   `// TODO: Test QWQ2` — 新版ルールはまだ検証が必要。
