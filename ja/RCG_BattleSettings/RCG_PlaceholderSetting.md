---
title: プレースホルダ (フュージョン)
description: カードフュージョンシステム用プレースホルダ；ネスト構造を保持し、フュージョン確定後に実際の効果に置換される
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# プレースホルダ (フュージョン)

> クラス名：`RCG_PlaceholderSetting`

## 用途
**カードフュージョンシステムが内部で使うプレースホルダ**。フュージョンプレビュー段階でネスト構造を保持し（例：「条件判断内に placeholder」）、プレイヤーに**新しい効果がどこに収まるか**を見せる。フュージョン確定後、本物の子設定に置換されます。

> [!IMPORTANT]
> この設定は**通常手動で作成しない** — フュージョンシステムが実行時に自動生成。データ上で見かけたら、そのカードはフュージョン中の中間状態の証。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **PlaceholderKey** | はい | i18n key、ヒントテキスト表示用（デフォルト `CardFusionPlaceholder`、PlaceholderIndex 番号付き）。 |
| **PlaceholderIndex** | はい | 占位番号（複数 placeholder を区別）。 |
| **PreviewDescription** | いいえ | カスタムプレビュー説明；非空のとき i18n の結果を上書き。 |

## 挙動
*   発動時に何もしない（純粋な占位）。
*   説明は `PreviewDescription` を優先、なければ i18n key `PlaceholderKey` の結果を表示。
*   **葉ノード**として存在 — `GetFusionCandidateSettings()` は空（フュージョン候補にならない）、`GetFusionBaseSetting()` は自身を返す（さらに置換されない）。

## 注意点
*   **データ上で手動作成しない**：意図的に「将来フュージョン穴」とマークしたい場合を除き。
*   **「DiminishedPlaceholder」との違い**：本クラスは**フュージョンシステム**用の中性占位（青枠）；`RCG_DiminishedPlaceholder` は**弱体化システム**用（赤字「無効化済み」）。
*   **PreviewDescription を優先**：UI 表示テストでよくこのフィールドを手動 override してヒントテキストを差し替える。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_PlaceholderSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key なし**：エディタ表示は stripped name `Placeholder`

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_PlaceholderKey` | `PlaceholderKey` | `string` | — | デフォルト `"CardFusionPlaceholder"` |
| `m_PlaceholderIndex` | `PlaceholderIndex` | `int` | — | i18n テンプレートで `{0}` に置換 |
| `m_PreviewDescription` | `PreviewDescription` | `string` | — | デフォルト空文字；非空のとき説明を上書き |

### A.3 主なメソッド
*   **`GetDescription`**：`m_PreviewDescription` 非空 → 直接返す；他 → `UCL_LocalizeManager.Get(m_PlaceholderKey, m_PlaceholderIndex)`。
*   **`GetDescriptionShort / GetShortName / GetDescriptionFormat`**：すべて `GetDescription` に委譲。
*   **`GetFusionCandidateSettings`** → 空リスト。
*   **`GetFusionBaseSetting`** → `this`。
*   **`AddAction`** → 空実装。

### A.4 他システムとの連携
*   **フュージョンシステムの `GetFusionBaseSetting`**：親クラス `RCG_BattleSetting.GetFusionBaseSetting` のデフォルトは `new RCG_PlaceholderSetting()` を返すため、未 override の全サブクラスはフュージョンプレビュー時に placeholder を生成。
*   **`RCG_DiminishedPlaceholder`**：このクラスを継承；弱体化システムの特化。
