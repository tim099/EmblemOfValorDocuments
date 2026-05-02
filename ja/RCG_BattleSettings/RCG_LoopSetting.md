---
title: 繰り返し発動(ループ)
description: 内包する組み合わせ効果を N 回繰り返す制御フロー型バトル設定
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 繰り返し発動(ループ)

> クラス名：`RCG_LoopSetting`

## 用途
**制御フロー型コンテナ**：`m_LoopContent`（1 つの `RCG_CombineSetting`）を `m_LoopTimes` 回繰り返します。よくある用途：
*   「3 ダメージを 5 回繰り返し」（連撃系）
*   「ドローした分発動」（変数連動の連撃）
*   「敏捷 1 を 3 回付与」（スタック累積）

## エディタ表示
```
▼ ✓ [繰り返し発動(ループ)(Loop)] (LoopContent) × N
    繰り返す回数        [数値] 1
    繰り返す効果        ▶ (組み合わせ効果コンテナ)
```

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **繰り返す回数** | はい | 繰り返し数；変数バインド可能（例：「残り手札数」「ある状態の層数」）。 |
| **繰り返す効果** | はい | 「組み合わせ効果」コンテナ；実際の繰り返し内容を含む。 |

> [!NOTE]
> なぜ内容が `RCG_CombineSetting` で `List<RCG_BattleSetting>` ではないのか？「組み合わせ効果」が複数 Setting の説明合成・タグ重複排除・プレビュー集約をすでに処理しているので、ラップしておくと効率が良いから。**単一効果のループでも 1 子の「組み合わせ効果」を中に置いてください**。

## 挙動

### バトル中の挙動
1. 「繰り返す回数」を解析して N を取得。
2. 「繰り返す効果」内のコンテンツを連続 N 回実行。
3. 各反復のアクションは順次バトルキューに挿入され、外側設定と連結実行。

### 説明文の表示
形式：「**N 回繰り返し: {内容説明}**」、内容の先頭文字を自動大文字化。

短縮説明（敵意図バー）：直接 `{内容} × {N}`。

### プレビューダメージは N を掛けない
カードのプレビューダメージは「**1 反復**」のダメージで、繰り返し回数を自動で掛けません。例：「3 回繰り返し 4 ダメージ」のプレビューは 4、**12 ではない**。

> [!IMPORTANT]
> これは設計選択 — プレイヤーは「毎回ダメージ」のほうが直感的。総ダメージを強調したいなら、カード説明に「**3 × 4 = 12**」と明記してください。

## 注意点

*   **繰り返し回数 = 0 / 負値**：crash しないが「カード無効」と同じ。プレイヤーは「バグ？」と思います。`Mathf.Max(1, ...)` で下限を与えるか、設計上避けてください。
*   **ネストループ**：「繰り返し発動」内に「繰り返し発動」も可能ですが、説明が「M 回繰り返し: N 回繰り返し: …」となり**プレイヤーに極めて不親切**。`M*N` でフラット化を。
*   **多ターゲット + ループ**：「全敵攻撃 + 3 回繰り返し」≠「敵各 3 回」。前者は「3 回 AOE、毎回ターゲット再解決」；後者は「**各ターゲット個別発動 (Foreach)**」を使ってください。
*   **AOE 攻撃をループに入れる**：毎回ターゲット再解決なので、途中で死んだ敵は後続反復から外れます（直感に合う挙動）。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_LoopSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_LoopSetting` → 「繰り返し発動(ループ)」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_LoopTimes` | 繰り返す回数 | `IntVariable` | `LoopTimes` | 変数型対応 |
| `m_LoopContent` | 繰り返す効果 | `RCG_CombineSetting` | `LoopContent` | 集約ロジック流用のため Combine をラップ |

### A.3 主なメソッド
*   **`AddAction`**：
    ```csharp
    int aLoopTimes = m_LoopTimes.GetValue(iData);
    for (int i = 0; i < aLoopTimes; i++)
        m_LoopContent.AddAction(iData, AddActionMode.InsertInOrder);
    ```
    **要点**：各反復で `InsertInOrder` を使い、外側設定との連結順を維持。
*   **`GetDescriptionFormat`**：
    1. `iData.m_FullSentence` を一時保存し `false` に設定（クリーンな内容説明取得用）。
    2. `LocalizedStringUtils.CapitalizeString` で先頭大文字化（暫定 hack）。
    3. `m_FullSentence = true` に戻し、i18n key `LoopSettingDes` でラップ。
*   **`GetDescriptionShort`** → i18n key `LoopSettingDesShort`、形式は `{Content} × {Times}`。
*   **`m_LoopContent` への透過**：`Infos` / `HasTerm` / `GetCollaborators` / `GetAtk` / `GetPreviewDamage` / `PreloadData`。
*   **`GetBattleSettings<T> / (Type)`** → 自身 + `m_LoopContent` 再帰。
*   **`GetFusionCandidateSettings`** → `m_LoopContent.GetFusionCandidateSettings()` に直接委譲（**Loop 自身は候補にならない**、構造で葉ではないため）。
*   **`GetFusionBaseSetting`**：自身を clone し、内容を placeholder 化された Combine に置換。

### A.4 他システムとの連携
*   **`RCG_CombineSetting`** → `m_LoopContent` のコンテナ型；Loop は Combine の上に成立し説明合成ロジックを再利用。
*   **`AddActionMode.InsertInOrder`** → 多反復アクションの順序維持；`PushBack` とは挙動が異なる。
*   **`IntVariable.GetValue`** → 繰り返し回数の解析；clamp なしのため 0 / 負数は 0 反復になる。

### A.5 既知の課題
*   `LocalizedStringUtils.CapitalizeString` は「暫定特殊処理」（コード内 `//暫時特殊處理` コメント）；i18n リファクタ時に正規対応すべき。
*   `GetPreviewDamage` が N を掛けないのは設計選択（バグではない）；要件変更時はここを直接修正。
*   旧 `CanEnhence` / `Enhence` はコメントアウト（強化系の旧ロジック）。
