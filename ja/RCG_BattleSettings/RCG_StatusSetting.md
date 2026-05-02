---
title: 状態
description: 全機能の状態設定 — 付与 / 減少 / 削除 / 変換 / 吸収 / ランダム抽出
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 状態

> クラス名：`RCG_StatusSetting`

## 用途
**ゲーム中の「状態」のコア設定**。層数付与、層数減少、状態全体の削除、タイプによる一括操作、A→B 変換など多モード対応。攻撃以外の**最も使われる設定の 1 つ**。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **対象** (`Target`) | はい | 状態適用ターゲットセレクタ。 |
| **StatusModifiedType** | はい | 変化タイプ、**10 モード**（下表）。 |
| **状態** (`Status`) | Add/Decrease/Remove/Convert/Set モード必須 | 操作する状態タイプ（`RCG_CustomStatusGenData`）。 |
| **StatusDropPool** | StatusDropPool モード必須 | どのプールからランダム抽出するか。 |
| **ConvertStatus** | Convert モード必須 | 何の状態に変換するか。 |
| **ConvertLayer** | Convert モード | 何層消費で変換 1 回（デフォルト 10）。 |
| **ConvertRatio** | Convert モード | 変換の比率（デフォルト 1）。例：ConvertLayer=10 + ConvertRatio=3 → 「ぼーっとを 10 層 → 気絶を 3 層」。 |
| **StatusTags** | RemoveStatusOfTargetType / AbsorbStatusOfTargetType モード | 操作する状態タイプリスト（例：「全 Debuff」）。 |
| **量** (`Amount`) | Add/Decrease/Set モード | 変化層数（変数可）。 |
| **DescriptionType** | — | 説明文型：`Default` / `Then`。 |
| **詳細設定** (`DetailSetting`) | — | `SkipAnim`（演出をスキップ、層数を直接設定）を含む。 |

### StatusModifiedType モード
| 値 | 動作 |
|---|---|
| **AddStatusLayer** | 層数追加（デフォルト） |
| **DecreaseStatusLayer** | 層数減少 |
| **RemoveStatus** | 指定状態を削除（層数問わず） |
| **RemoveAllStatus** | 全状態削除 |
| **RemoveAllDebuff** | 全負面状態削除 |
| **RemoveStatusOfTargetType** | StatusTags リストで一括削除 |
| **AbsorbStatusOfTargetType** | ターゲットの特定タイプを自身に吸収 |
| **ConvertStatus** | A 状態 → B 状態（Layer / Ratio で変換） |
| **SetStatusLayer** | 層数を指定値に設定 |
| **StatusDropPool** | 状態プールからランダムに 1 つ付与 |

## 挙動
*   モードに応じて `m_Target.GetTargets(iData)` に適用。
*   `Amount` を持つモードは変数バインド可（例：「手札数で敏捷を加算」）。
*   `Convert` モード：`ConvertLayer` 層消費ごとに `ConvertRatio` 層の ConvertStatus を生成。
*   説明はモードによって異なる — UI に i18n 名 + アイコンを表示。

### カードフュージョン
`AddStatusLayer / DecreaseStatusLayer / SetStatusLayer` の 3 モードのみフュージョン可能（Amount 加算）。他のモードはフュージョン不可。

## 注意点
*   **AbsorbStatusOfTargetType**：「**自身**」に吸収する（ターゲットセレクタへではない）— 固定挙動。
*   **状態プールのランダム**：`StatusDropPool` は戦闘シードの影響を受ける；同シードでリプレイすると同じ状態を引きやすい。
*   **SkipAnim**：演出スキップで結算を高速化；プレイヤーが状態変化を見逃す可能性。重要な状態にはチェックなしを推奨。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_StatusSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable]`** 標記
*   **i18n クラス名 key**：`RCG_StatusSetting` → 「状態」
*   **同ファイル内ネストクラス**：`DetailSetting`（`m_SkipAnim` を含む）/ `DescriptionType` enum (`Default`, `Then`)
*   **同ファイルトップレベル enum**：`StatusModifiedType`（10 個の値）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |
| `m_StatusModifiedType` | `StatusModifiedType` | `StatusModifiedType` | — | デフォルト `AddStatusLayer` |
| `m_Status` | 状態 | `RCG_CustomStatusGenData` | `Status` | `[Conditional(...5 モード)]` |
| `m_StatusDropPool` | `StatusDropPool` | `RCG_StatusDropPoolGenData` | — | `[Conditional(... StatusDropPool)]` |
| `m_ConvertStatus` | `ConvertStatus` | `RCG_CustomStatusGenData` | — | `[Conditional(... ConvertStatus)]` |
| `m_ConvertLayer` | `ConvertLayer` | `IntVariable` | — | `[Conditional(... ConvertStatus)]`、デフォルト 10 |
| `m_ConvertRatio` | `ConvertRatio` | `IntVariable` | — | `[Conditional(... ConvertStatus)]`、デフォルト 1 |
| `m_StatusTags` | `StatusTags` | `List<RCG_StatusTagGenData>` | — | `[Conditional(... 2 種 OfTargetType モード)]` |
| `m_Amount` | 量 | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]` + `[Conditional(... 3 種要 Amount のモード)]` |
| `m_DescriptionType` | `DescriptionType` | `DescriptionType` (内部 enum) | `DescriptionType` | |
| `m_DetailSetting` | 詳細設定 | `DetailSetting` (内部) | `DetailSetting` | `m_SkipAnim` を含む |

### A.3 主なメソッド
*   **`Fusion`**：`m_StatusModifiedType` が 3 種の Layer 操作のいずれかである必要；clone + `IntVariable.FuseAdd(m_Amount)`。
*   **`GetShortName`**：StatusModifiedType の分岐：直接 i18n / `GetDescription()` / `m_StatusDropPool.GetData().GetShortName()`。
*   **`AddAction` / `Infos`**（ファイル 200 行外）：モードに応じた実際のロジックと情報集約。

### A.4 他システムとの連携
*   **`RCG_UnitStatus`**：実際の状態コンテナ（`GetStatusLayer` / `m_Status` 字典等）。
*   **`RCG_StatusTagGenData.StatusDebuff / StatusDot`**：タイプタグ、一括削除に使用。
*   **`CreateAction.StatusAction`**：実際の Action ビルダー（`RCG_ConsumeStatusSetting` などと共有）。

### A.5 既知の課題
*   `CanEnhence` / `Enhence` はコメントアウト（強化系リファクタ中）。
