---
title: RCG_CombineSetting 解説
description: 複合バトルセッティング (CombineSetting) の責務、フィールド意味、挙動の合成規則
last_updated: 2026-05-02
target_audience: [AI_Agent, Gameplay_Programmer, Designer]
---

# RCG_CombineSetting

## 1. 責務 (Responsibility)
複数の `RCG_BattleSetting` を **単一** のセッティングへ統合し、順次または同時に実行します。複合挙動（例：「ダメージ + ドロー + ステータス付与」）を組み立てる際の最も一般的なコンテナです。

継承元：`RCG_BattleSetting`

## 2. 主なフィールド

| フィールド | 型 | 意味 |
|---|---|---|
| `m_OverrideDescription` | `bool` | 子セッティングを連結した説明文を上書きするか。`true` のとき `m_Description` を使用し、説明が冗長になるのを避けます。 |
| `m_Description` | `RCG_LocalizeData` | 上書き用の説明本体。`m_OverrideDescription = true` のときのみ有効（`Conditional` 属性で表示制御）。 |
| `m_CombineSettings` | `List<RCG_BattleSetting>` | 組み合わせて実行する子セッティングのリスト。`AlwaysExpendOnGUI` により Inspector で常時展開されます。 |

## 3. 挙動の合成規則

### 3.1 プレイ可否判定 `CheckPlayable`
**全ての** 子が `CheckPlayable == true` の場合のみ全体としてプレイ可能（AND ロジック）。1 つでも false なら短絡的に false。

### 3.2 プレビューダメージ `GetPreviewDamage`
全子の `GetPreviewDamage` の **最大値** を代表値として返します（**MAX ロジック**）。UI に表示される攻撃力 = 子の中で最強の一撃。

### 3.3 タグ / カード情報
`Infos` / `GetBattleTags()` は **重複排除付き連結** を採用し、同種の Buff/Tag が複数子間で重複表示されるのを防ぎます。

### 3.4 説明生成 `GetDescription`
*   **未上書き**：各子の `GetDescription` を改行で連結（空文字はスキップ）。
*   **上書き**：`m_Description.Name` を直接返却。子は説明合成に関与しません。

## 4. 設計上の注意
*   **粒度**：各子は単一挙動（damage / draw / buff など）に専念させ、組み立ては CombineSetting に任せるのが再利用・単体テスト共に有利です。
*   **過度なネストを避ける**：CombineSetting の中に CombineSetting を入れることは可能ですが、`GetPreviewDamage` の MAX ロジックが直感に反して見える原因になります。最大 2 階層を推奨。
