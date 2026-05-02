---
title: 協力
description: 専門条件に合うキャラ数が指定数に達すると効果発動；条件発動 / 数量変数の 2 モード対応
last_invalid_input: 2026-05-02
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 協力

> クラス名：`RCG_CollaborationSetting`

## 用途
**チーム合体メカニズム**：「**味方に特定の専門を持つキャラが何人いるか**」で効果発動を判定するか、合致人数を変数として使えます。よくある用途：
*   「魔法使いが 2 人以上いれば追加ダメージ」
*   「合致人数に応じて治療量が変動」

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CollaborationType** | はい | モード：<br>• **Default** — `CollaborationCount` に達すると発動<br>• **Variable** — 自動発動；合致人数を変数 X として使える |
| **CollaborationCount** | Default モード | 必要キャラ数（デフォルト 2）。 |
| **RequireSkills** | いいえ | 専門要求リスト；空リスト = 全員が合致扱い。 |
| **CollaborationVariable** | Variable モード | 変数名（デフォルト `X`）；合致人数を保存し、後続設定が参照可能。 |
| **CollaborationSetting** | はい | 発動後に実行する「**組み合わせ効果**」。 |

## 挙動
*   **Default モード**：
    *   味方キャラから `RequireSkills`（任一一致）を持つ数を数え `CollaborationCount` 以上か判定。
    *   達成 → `CollaborationSetting` 実行；未達成 → 発動しない。
*   **Variable モード**：
    *   閾値判定なし、**直接発動**。
    *   合致人数を `CollaborationVariable` に保存、`CollaborationSetting` 内部で参照可能（人数で効果スケール）。
*   説明には：条件説明（i18n key `CollaborationTypeInfo_*`）+ 子設定説明が含まれる。

## 注意点
*   **RequireSkills 空 + Default モード**：「全員が合致」と等しく、協力人数 = 味方総数 — 設計意図を確認。
*   **変数命名衝突**：複数の設定が `X` を使うと後者が前者を上書き；意味のある変数名（`CollabCount` など）を。
*   **トリガー子効果内に新動作を積み重ね可能**：`CollaborationSetting` は `RCG_CombineSetting`、複数効果を組み合わせ可能ですが、**ネスト深度**が説明の可読性に影響。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CollaborationSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CollaborationSetting` → 「協力」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CollaborationType` | `CollaborationType` | enum | — | `Default` / `Variable` |
| `m_CollaborationCount` | `CollaborationCount` | `int` | — | デフォルト 2 |
| `m_RequireSkills` | `RequireSkills` | `List<RCG_SkillTagGenData>` | — | |
| `m_CollaborationVariable` | `CollaborationVariable` | `string` | — | `[Conditional("m_CollaborationType", false, Variable)]`；デフォルト `"X"` |
| `m_CollaborationSetting` | `CollaborationSetting` | `RCG_CombineSetting` | — | `[AlwaysExpendOnGUI]` |

### A.3 主なメソッド
*   **`Infos`**：自身の description（`GetCollaborationDes()` + `CollaborationTypeInfo_*` i18n を含む）+ `m_CollaborationSetting.Infos`。
*   **`GetCollaborationRequireSkillDes()` (private)**：空リスト → `CollaborationRequireSkills_Any`；他は `CollaborationRequireSkills_Specify`。
*   **`AddAction`**（ファイル 100 行外）：合致人数を解析、CollaborationType で `m_CollaborationSetting` を発動するか判定。

### A.4 他システムとの連携
*   **`RCG_SkillTagGenData`**：専門タグテンプレート。
*   **`RCG_CombineSetting`**：子効果コンテナ。
