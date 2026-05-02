---
title: 弱体化済みプレースホルダ (Diminished)
description: 「カード効果無効化」で弱体化された葉効果のプレースホルダ；赤字「無効化済み」を表示しさらなる弱体化に耐性
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 弱体化済みプレースホルダ (Diminished)

> クラス名：`RCG_DiminishedPlaceholder`

## 用途
**「カード効果無効化」（`RCG_DiminishSetting`）で弱体化された葉効果のプレースホルダ**。元々の葉効果（例：「攻撃」）はこのプレースホルダに置換され、説明には**赤字「無効化済み」**が表示されます。

> [!IMPORTANT]
> この設定は**通常手動で作成しない** — 弱体化システムが実行時に自動で置換します。データ上で見かけたら、そのカードは過去に弱体化された痕跡。

## エディタ表示
通常は新規作成のドロップダウンには現れません（`AllTypes` には登録されていますが）；表示されるとフィールドが何もありません — ただの視覚マーカーです。

## 主なフィールド
（なし）

## 挙動
*   発動時に何もしない（純粋な placeholder）。
*   説明は常に**赤字「無効化済み」**（`RCG_Extensions.TagColors.Diminished`）。
*   「葉ノード」として存在し、**さらなる弱体化を受けない**（フュージョン候補は空）。

## 注意点
*   **データ上で手動作成しない**：意図的に「すでに弱体化済み」マーカーを付けたい場合を除き。
*   **「Placeholder」との違い**：`RCG_PlaceholderSetting` は**フュージョンシステム**用の中性占位（青枠）；`RCG_DiminishedPlaceholder` は**弱体化システム**用（赤字「無効化済み」）。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishedPlaceholder.cs`
*   **継承元**：`RCG_PlaceholderSetting`
*   **i18n クラス名 key なし**：エディタ表示は stripped name `Diminished`

### A.2 フィールド対照
（自身のフィールドなし）

### A.3 主なメソッド
*   **`GetDescription / GetDescriptionShort / GetShortName`**：いずれも `UCL_LocalizeManager.Get("DiminishedPlaceholder").GetTagColor(TagColors.Diminished)` を返す。
*   **`GetFusionCandidateSettings`** → 空リスト（フュージョン候補にならない）。
*   **`GetFusionBaseSetting`** → `this`（自身を保持、**さらに置換されない**）。

### A.4 他システムとの連携
*   **`RCG_DiminishSetting.AddAction`**：このプレースホルダの置換フローをトリガーする。
*   **`RCG_CardBattleData.Diminish`**：置換実行のエントリ。
*   **`RCG_Extensions.TagColors.Diminished`**：赤字色票ソース。
*   **i18n key `DiminishedPlaceholder`**：表示文字（「無効化済み」風）。
