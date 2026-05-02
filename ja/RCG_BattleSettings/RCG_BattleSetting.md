---
title: バトル設定（基底）
description: 全バトル設定（攻撃・回復・条件・組み合わせなど）の共通基底クラス。共通の「有効化」スイッチ・説明文システム・ドロップダウン分類を提供
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# バトル設定（基底）

> クラス名：`RCG_BattleSetting`

## 用途
「バトル中に発生する1つの効果単位」の **抽象基底クラス** です。直接作成するものではなく、すべてのバトル設定（攻撃・回復・組み合わせ・条件…）の親になります。カードや敵スキル・ステータス効果上の「**バトル設定**」フィールドのドロップダウンに表示されるのが、この基底のサブクラスです。

## エディタ表示
あらゆる `RCG_BattleSetting` サブクラスは Inspector で次の形式で表示されます：
```
▼ ?  ✓  [タイプ(英語)] 概要説明
```
*   **左側のチェックボックス**：「有効にする」フィールド。チェックを外すと外側のコンテナ（組み合わせ効果など）から自動的にスキップされます。
*   **`?` アイコン**：対応するヘルプドキュメントを開く（`[HelpURL]` が付いている場合）。
*   **`[タイプ]` ラベル**：サブクラスの日本語名（「攻撃」「回復」「コンボ効果」など）。i18n key `RCG_XxxSetting` で定義；後ろの括弧は英語識別子。
*   **末尾テキスト**：折りたたみ時の識別用にサブクラス自身が生成する短い説明。

## 共通フィールド

| エディタ表示 | 対応コード | 説明 |
|---|---|---|
| **有効にする** | `m_IsEnable` | バトル中に発動するか。外側のコンテナは無効項目を自動スキップします。 |

> [!NOTE]
> サブクラス固有のフィールド（攻撃力・攻撃回数など）は、それぞれの設定の説明書を参照してください。

## 選択可能なサブクラス

ドロップダウンの選択肢は文脈に応じて2つのプールが切り替わります：

### 通常カード資料の場合
カード・ステータス・強化資料を編集するときに表示。50種以上、例：
*   **効果系**：攻撃・防御・回復・状態 …
*   **リソース系**：ドロー・捨て札・コスト変化・アイテム消費 …
*   **制御フロー**：組み合わせ効果・条件判断・繰り返し発動・ランダム発動・各ターゲット発動 …
*   **高度**：召喚・変身・協力・カウンター調整 …

### モンスター（敵方）資料の場合
`RCG_UnitData` または `RCG_MonsterActionData` を編集する際は、モンスター専用の追加 2 項目が表示されます：
*   **モンスター逃走** (`RCG_MonsterFleeSetting`)
*   **モンスター移動** (`RCG_MonsterMoveSetting`)

## 共通挙動（自分で設定するものではないが、目に入る）

*   **説明文の自動ローカライズ**：各サブクラスは自フィールド値から「詳細説明」（カード情報パネル）と「短縮説明」（敵意図バー）を生成します。
*   **リソース事前読込**：システムがバトル前にアイコン・特効を自動プリロード — **手動設定不要**。
*   **フュージョン**：各サブクラスが独自の合成ルールを定義します（例：2 つの「回復」をフュージョンすると治療量が加算される）。

## 注意点

*   **無効化 ≠ 削除**：データに残り Inspector 表示はされますが、実行時にスキップされるだけ。完全に消すには右端の `−` ボタンを使ってください。
*   **新サブクラスがドロップダウンに出ない**：プログラム側の `AllTypes` リストに登録されていない可能性。データ側では解決できません。
*   **`RCG_BattleSetting` 自身は選べない**：ドロップダウンは具象サブクラスのみ列挙；基底クラスはインターフェース契約専用。

---

## 付録：プログラマ向け (Programmer Reference)

> 以下の内容はプログラム内部用語を含み、対象はプログラマと AI agent。前半の利用者向け記述を優先してください。

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleSetting.cs`
*   **継承元**：`UnityJsonSerializable`
*   **実装インターフェース**：`UCLI_ShortName` / `UCLI_TypeList` / `UCLI_GetTypeName` / `UCLI_IsEnable` / `UCLI_NameOnGUI`

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_IsEnable` | 有効にする | `bool` | `IsEnable` | `[UCL_HideOnGUI]`；`NameOnGUI` で手動描画 |
| `IsShowOnUI` | （GUI なし） | `static bool` | — | グローバル UI スイッチ；一部サブクラスの説明分岐に影響 |
| `s_Types` / `s_MonsterDataTypes` | （GUI なし） | `static List<Type>` | — | サブクラス選択肢；`AllTypes` getter で文脈に応じ切替 |

### A.3 主なメソッド
*   **`virtual CheckPlayable(TriggerEffectData)`** → デフォルト `true`；サブクラスでリソース／条件チェックを実装。
*   **`virtual AddAction(...)`** → **核心エントリ**；対応 Action を `RCG_BattleManager` キューに push。基底は空実装。
*   **`virtual GetDescription / GetDescriptionFormat / GetDescriptionParams`** → 説明三点セット；`Format + Params` を override 推奨。
*   **`virtual GetDescriptionShort`** → 敵意図バー用の短縮説明、デフォルト空文字。
*   **`virtual GetShortName`** → Inspector 折りたたみ用に説明 25 文字を切り出し。
*   **`virtual Info / Infos`** → ツールチップ情報；`Infos` は単一 `Info` をラップ。
*   **`virtual GetBattleTags / HasTerm / GetCollaborators`** → タグ・用語・協力者問い合わせ；コンテナ系サブクラスは再帰必須。
*   **`virtual GetAtk / GetPreviewDamage`** → 攻撃力／プレビュー値；後者は `-1` 返却で「非攻撃カード」扱い。
*   **`virtual PreloadData(CancellationToken)`** → 戦闘前リソース読込；コンテナ系は子設定を `await` 必須。
*   **`virtual Fusion / GetFusionCandidateSettings / GetFusionBaseSetting`** → カードフュージョン三点セット；既定は非対応／自身を候補／プレースホルダ化。
*   **`implicit operator string`** → `GetDescription()` を呼ぶ；文字列連結に便利だが null 注意。

### A.4 他システムとの連携
*   **`AllTypes` getter**：`UCLI_Asset.s_CurOnGUIAsset` が `RCG_UnitData` / `RCG_MonsterActionData` ならモンスター用 `s_MonsterDataTypes`（`RCG_Monster*Setting` 2 種を含む）を返却。
*   **JSON シリアライズ**：`UnityJsonSerializable` 継承；既定で `JsonConvert.SaveDataToJson(this, JsonConvert.SaveMode.Unity)` 経由。
*   **GUI 描画**：`NameOnGUI` が Inspector 1 行目に `IsEnable` チェック + `[タイプ] 短縮説明` を描画；タイプ名は `GetTypeName(GetType().Name)` から（`RCG_` と `Setting` を除いた key で i18n 検索）。

### A.5 既知の課題
*   `CanEnhence` / `Enhence(RestSetting)` はコメントアウト（旧強化系の遺物）。
*   `DeserializeFromJson` / `SerializeToJson` も同様にコメントアウト（base 既定使用）。カスタムが必要な場合はコメントを解除。
