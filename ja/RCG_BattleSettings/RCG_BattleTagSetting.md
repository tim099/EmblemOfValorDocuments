---
title: 戦闘タグ
description: 効果に戦闘タグ（脆弱・敏捷など）を付与；自身は Action を発動せず、上位設定の説明とタグ集約を変える
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘タグ

> クラス名：`RCG_BattleTagSetting`

## 用途
効果に**戦闘タグを付与** — 自身は**バトル行動を一切生成せず**、純粋に「**タグキャリア**」として機能。よくある用途：
*   組み合わせ効果に「**脆弱**」「**敏捷**」「**消費**」などの属性タグを与える
*   「カード情報」パネル上の用語ソース

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **BattleTags** | はい | 戦闘タグリスト；各項目は `RCG_BattleTagGenData`（タグテンプレート）。複数指定可能。 |

## 挙動
*   **Action 生成なし**：この設定の `AddAction` は空実装 — タグ宣言のみ。
*   **カード情報**：各タグはカードホバーツールチップに名前と説明が表示；積み重ね可能タグ（`m_IsStackable = true`）の説明には層数が含まれる。
*   **短縮説明**：全タグの短縮名を直結（追加修飾なし）。
*   **`GetBattleTags()`**：上位の「組み合わせ効果」「ループ」などのコンテナがこのメソッドで**全子設定のタグを集約**して統一表示。

## 注意点
*   **実効果が必要なら「組み合わせ効果」または「状態」へ**：戦闘タグそれ自体は metadata のみ；「脆弱」由来の減傷効果には別途トリガーロジックが必要。
*   **空リスト**：合法だが無意味；データ容量を消費するだけで挙動なし。
*   **常に展開**：この設定は `[AlwaysExpendOnGUI]` 属性を持つため、Inspector 上で**折りたたまれない**。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable] + [UCL.Core.ATTR.AlwaysExpendOnGUI]`** 標記
*   **i18n クラス名 key**：`RCG_BattleTagSetting` → 「戦闘タグ」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_BattleTags` | `BattleTags` | `List<RCG_BattleTagGenData>` | — | |

### A.3 主なメソッド
*   **`AddAction`** → 空実装（`// base.AddAction(...)` がコメントアウトのみ）。
*   **`GetBattleTags()`** → `m_BattleTags.Clone()`；上位コンテナがこのメソッドで集約。
*   **`Infos`**：各 tag が `{LocalizedName}\n{Description}` を構成；積み重ね可能タイプには `StackedBattleTag` テンプレート（`eBattleVariable_BattleTagStackCount` 変数を含む）を追加。
*   **`GetShortName`** → `m_BattleTags[i].GetShortName()` を結合；空リストでは base にフォールバック。
*   **`GetDescription`** → 常に `string.Empty`（意図的に override）。

### A.4 他システムとの連携
*   **`RCG_BattleTagGenData / RCG_BattleTag`**：タグテンプレートとインスタンス。
*   **i18n keys**：`StackedBattleTag` / `eBattleVariable_BattleTagStackCount` / タグ色 `RCG_BattleTag`。
*   **`GetBattleTags` の呼び出し元**：すべてのコンテナ系設定（CombineSetting / LoopSetting / ConditionalSetting / ForeachTargetSetting など）。
