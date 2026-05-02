---
title: コンボ効果
description: 複数のバトル設定（攻撃・回復・ドロー…）を1つの複合効果としてまとめる、最も使われるコンテナ型
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# コンボ効果

> クラス名：`RCG_CombineSetting`

## 用途
**複数の**バトル設定（攻撃・回復・ドロー・状態付与…）を**1つの**ユニットにまとめ、バトル中に**一緒に**実行します。複合効果カードを作るときの定番コンテナ：
*   「3 ダメージ + 1 ドロー」
*   「ブロック 2 + 敏捷 1 スタック付与」
*   「敵前列攻撃 + 自傷 1 HP」

## エディタ表示
```
▼ ✓ [コンボ効果(Combine)] [サムネイル]
    OverrideDescription   □
    説明                  [非表示 / 表示]
    組み合わせ設定        ▶ (子設定リスト)
```

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **OverrideDescription** | いいえ | 子設定の説明文を連結したものを上書きするか。チェック時は「**説明**」フィールドを直接表示し、文字列が冗長になるのを防ぎます。 |
| **説明** | OverrideDescription チェック時 | 上書き用の説明本体（`RCG_LocalizeData`）；チェックなしのときは自動で隠れます。 |
| **組み合わせ設定** | はい | 組み合わせ実行する子バトル設定リスト。各項目は完全なバトル設定（攻撃 / 回復 / ドロー…）です。 |

## 挙動

### 説明文の表示
*   **OverrideDescription オフ**：各子設定の説明を改行で連結（空文字スキップ）。
*   **OverrideDescription オン**：「説明」フィールドを直接表示；子設定は説明合成に関与せず。

> [!TIP]
> 子設定説明が長くなりすぎたら **OverrideDescription をオン**にして自前で簡潔な説明を書きましょう。ただし手動同期維持を忘れずに。

### プレイ判定
**全部の**子設定が「プレイ可能」のときのみコンボ全体がプレイ可能。1 つでも失敗するとカード全体が使えません（AND ロジック）。

### プレビューダメージ表示
全子設定の `GetPreviewDamage` の **最大値** を採用（例：3 子の予想 5 / 8 / 2 → UI 表示 8）。

### 攻撃タグ / カード情報
全子設定のタグ・Buff アイコンは**重複排除**して合成表示。同じ Buff アイコンが複数子設定間で連続表示されないように。

## 注意点

*   **子設定の粒度を単純に保つ**：各子設定は1つの仕事（ダメージ / ドロー / 状態付与）に専念し、コンボ効果が組立を担当。これで再利用しやすく、単一挙動の変更が全体に波及しません。
*   **過度なネストを避ける**：コンボ効果内にコンボ効果も可能ですが、説明とプレビューが直感的でなくなります。**最大 2 階層を推奨**。
*   **空リストも有効**：「組み合わせ設定」を空にできますが、何も発動しないカードになります — データエラーとみなされます。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CombineSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_CombineSetting` → 「コンボ効果」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_OverrideDescription` | `OverrideDescription` | `bool` | `OverrideDescription` | `m_Description` の表示制御 |
| `m_Description` | 説明 | `RCG_LocalizeData` | `Description` | `[Conditional(nameof(m_OverrideDescription), false, true)]` |
| `m_CombineSettings` | 組み合わせ設定 | `List<RCG_BattleSetting>` | `CombineSettings` | `[AlwaysExpendOnGUI]` 常時展開 |

### A.3 主なメソッド
*   **`CombineSettings` (property)** → `m_CombineSettings.GetEnableBattleSettings()` で `IsEnable=false` を除外。
*   **`CheckPlayable`** → 子全体に対する AND；1 つでも false なら短絡。
*   **`GetPreviewDamage`** → 子設定の `GetPreviewDamage` に対し `Mathf.Max`、初期値 -1。
*   **`Infos / GetBattleTags`** → `AppendIfNotRepeat` で重複排除して連結。
*   **`GetDescription`**：`m_OverrideDescription = true` なら `m_Description.Name`、それ以外は子説明を改行で連結（空はスキップ）。
*   **`GetBattleSettings<T> / (Type)`** → 自身がマッチすれば結果に追加 + `CombineSettings` を再帰。

### A.4 他システムとの連携
*   **`RCG_LoopSetting`** は `m_LoopContent` として `RCG_CombineSetting` を保持；Loop は Combine の上に成立。
*   **`RCG_BattleTagCombineSetting`** は `RCG_CombineSetting` を継承し BattleTag 専用ロジックを特化。

### A.5 既知の課題
*   `m_OverrideDescription` と `m_Description` は連結欄；旧 `m_OverridingDescriptionKey` はコメントアウト済み。
