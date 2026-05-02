---
title: カードデータ (RCG_CardData) 解説
description: ゲーム内すべてのカードのデータテンプレート：基本情報、効果、強化ブランチ、パッシブ効果、解放条件などの完全な定義
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カードデータ

> クラス名：`RCG_CardData`

## 用途
**ゲーム内のすべてのカードのデータテンプレート**。基本情報（名前、コスト、レアリティ、カード絵）から実際の効果、強化ブランチ、パッシブタグ、解放条件まですべてこの Asset で定義します。Developer Page の `EditItems` グループ配下、すべての使用可能 / ショップ販売 / ドロップ可能なカードがこのクラスのインスタンスです。

継承元：`RCG_Asset<RCG_CardData>`、実装インターフェース：`RCGI_Item`（インベントリに追加可能）/ `RCGI_CardData`（カードデータ契約）/ `RCGI_Unloackable`（解放可能）。

## エディタ表示
カードを編集すると 3 つのセクションに分かれます：
```
カード設定 (CardData)        ← 基本フィールド：名前 / コスト / タイプ / レアリティ / タグ / 解放等
効果 (N)                     ← m_Effects：実際のカード効果（OnPlay 等のトリガー）
パッシブ効果 (N)              ← m_PassiveEffects：条件付き BattleTag（Cost 計算等に影響）
プレビュー                    ← カード外観と説明をリアルタイム表示
```

最右側の「プレビュー」はカード外観、Tooltip タグ、カード絵、解放条件をリアルタイム表示し、「**強化ブランチプレビュー**」と「**説明をコピー**」ボタンを提供。

## 主なフィールド（カード設定 / CardData）

### 基本情報
| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **名前(多言語)** (`LocalizeName`) | はい | カード名（`RCG_LocalizeData`、多言語対応）。 |
| **コスト** (`Cost`) | はい | このカードを使うのに消費するエネルギー；整数。 |
| **カードタイプ** (`CardType`) | はい | `Default`（通常カード）または `Chant`（**詠唱カード**：使用前に複数ターンの詠唱が必要）。 |
| **詠唱回数** (`ChantTurn`) | CardType=Chant 時 | 詠唱に必要なターン数（変数可）；条件表示。 |
| **レアリティ** (`Rarity`) | はい | `Bronze` / `Silver` / `Gold` / `Legend` / `Cursed` 等（`RCG_RarityTagGenData`）。 |
| **対象範囲** (`TargetType`) | はい | 17 種：`None`、`Friend`、`Allied(Front/Back)`、`Enemy(Front/Back)`、`All`、`AnyPos`、`AlliedNonLeader`、`AnySummoned` 等。 |
| **使用タイプ** (`UsedTypeTag`) | はい | 使用後の処理方式（例：「消費」「再利用可」「乙太」）。 |
| **価格** (`Price`) | はい | 商人販売価格；整数。デフォルト 100。 |
| **カードアイコン** (`CardIcon`) | はい | カード絵（`RCG_SpriteData`）。 |

### タグと専門
| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **専門** (`SkillTags`) | いいえ | キャラクターが持つべき専門（戦士 / 魔法使い / 牧師…）；空 = 任意のキャラクターが使用可。複数 = 「**いずれか一致でよい**」。 |
| **カードタグ** (`CardTags`) | いいえ | 攻撃 / スキル / 詠唱 / 消費… 等の分類タグ。**重要**：「詠唱カードか」の判定は `HasTag()` を使用してください。`m_CardTags` には詠唱タグが自動含まれません。 |

### カード効果
| ブロック | 説明 |
|---|---|
| **効果** (`m_Effects`) | 主なカード効果リスト；各項目は `RCG_CommonEffect`、内部に「トリガータイミング（OnPlay / OnDraw / OnDiscard / ...）+ 組み合わせ効果」を含む。カードプレイまたはトリガー条件成立時に発動。 |
| **パッシブ効果** (`m_PassiveEffects`) | 条件付き `RCG_ConditionalPassiveBattleTag` リスト；**ステータス問い合わせ**用（例：「手札 ≥ 5 のとき -1 コスト」）— **OnTriggerEffect には参加しない**、純粋に Cost などの属性計算に影響。 |

### 強化システム
| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **禁止強化ブランチ** (`BannedEnhence`) | いいえ | このカードに落ちて欲しくない強化ブランチ ID。 |
| **強化プール** (`EnhencePool`) | はい | 強化時にランダム抽選される `RCG_CardEnhenceDropPoolGenData`。 |
| **ApplyEnhenceOnAllTiming** | — | 強化効果をこのカードのすべてのトリガータイミングに適用するか（OnPlay だけでなく）。 |
| **強化レベル** (`UpgradeLv`) | — | 表示用：≥1 でカード名後ろに `+1` / `+2` を追加；通常システムが自動設定、手動変更は非推奨。 |
| **EnhenceLocalize** | いいえ | 強化版の代替命名（例：α、β、γ）；空のときは `+` プレフィックスを使用。 |

> [!IMPORTANT]
> 現在**強化回数の上限は 1**（`m_UpgradeLv < 1` のときのみ強化可）、**詛咒カード (Cursed) は強化不可**。`CanEnhence` 判定で阻止されます。

### 説明
| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **カード説明方式** (`CardDescriptionType`) | はい | `Auto`（効果から自動合成）または `Manually`（手動記述）。 |
| **カード説明** (`CardDescription`) | Manually 時 | 手動記述の説明本体（多言語）；自動モードでは隠れる。 |
| **説明テンプレート** (`DescriptionTemplate`) | Manually 時 | `{(OnPlay.0)}` などのプレースホルダを含むテンプレート文字列；`Generate Description Template` ボタンで現在の効果から自動生成可。 |

> [!TIP]
> 自動モードはほとんどのケースで対応可能 — 効果説明、対象範囲、battle tags をクリーンに合成。**精密な文型制御** / **一部効果の隠蔽** / **フレーバーテキスト追加**が必要なときだけ Manually に切替。

### 解放 / 雑項
| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **解放** (`Unlock`) | いいえ | 解放条件（`RCG_UnlockEntry`）；解放前は報酬 / ショップに登場しない。 |
| **HideInCodex** | — | 図鑑で隠す（テスト用 / 廃カード）。 |
| **メモ** (`Note`) | — | プレーンテキストのメモ（実効果なし、デザイナーの参考用）。 |
| **作者** (`Author`) | — | カード作者の署名。 |

## 挙動

### カード表示名
`LocalizedName` から計算：
*   通常カード：直接 `m_LocalizeName.Name`
*   強化版（`UpgradeLv ≥ 1`）：
    *   `EnhenceLocalize` 非空 → `{Name}{α/β/γ}`
    *   `UpgradeLv == 1` → `{Name}+`
    *   `UpgradeLv ≥ 2` → `{Name}+{N}`
*   詠唱カード：ゲーム中は「**詠唱**」プレフィックス + 詠唱色で表示；Editor 内では `詠唱[Name]` と表示

### 詠唱カード（CardType = Chant）
*   `m_ChantTurn` ターン詠唱しないと使用できない。
*   説明文の最上部に「**詠唱[N]**」（緑の詠唱色）を追加。
*   `HasTag(TagChant)` は true を返す（`CardTags` に明示されていなくても）。

### 説明生成
*   **自動モード (Auto)**：「対象範囲 → 各 OnTriggerOn グループ → トリガー効果 → battle tags → passive 効果」の順で自動合成。
*   **手動モード (Manually)**：`m_CardDescription` を本体とし、`GetDescriptionParams()` で取得した `(OnPlay.0)` 等のプレースホルダを置換。

### カード情報パネル（Tooltip 右側）
表示順序：
1. 必要専門（あれば）
2. 各効果の `Infos`（重複排除）
3. 戦闘タグの解説
4. パッシブ効果の条件 + タグ情報
5. 使用タイプ（`m_ShowTag = true` の場合）
6. 詠唱カードは詠唱説明を最上部に追加

### 強化ブランチ
*   `GetEnhenceBranchs(N)` で `EnhencePool` から N 個のブランチを抽選。
*   フィルター条件：`BannedEnhence` リストにあるものは除外、`RCG_CardEnhenceCondition` を通過しないものは除外。
*   各ブランチはカードを clone し `UpgradeLv` を上昇させ、対応する強化ロジックを適用。

### カードフュージョン
本クラスは**フュージョンルールを直接定義しない**；フュージョン時の「葉効果のマージ」は各 `RCG_BattleSetting` サブクラスが自前で処理します（`RCG_BattleSetting.GetFusionCandidateSettings / GetFusionBaseSetting` を参照）。

## 注意点

*   **詠唱カードは CardTags に詠唱タグを手動追加不要**：システムが `HasTag` チェック時に自動的に補ってくれます。ただし `CardType = Chant` の設定は必須。
*   **価格自動化**：エディタページに「Auto Price」ボタンがあり、すべてのカードの Price を「レアリティ価値 × 5」で上書きします。**手動で変更した価格は上書きされる**ため、保持したい場合はボタンを押さないこと。
*   **解放条件の入れ忘れ**：`Unlock` を空 = デフォルトで解放；隠し実績カードを作るときは忘れずに。
*   **m_Note / m_Author はプレイヤーには見えない**：純粋に開発者用のメモ / 署名。
*   **Description の多言語整合性**：手動説明モードではすべての言語に記入してください；欠落はデフォルト言語にフォールバック。
*   **強化カードの Enhance Pool は同源**：強化ブランチはカード自身の `EnhencePool` から抽選；同名カードの異なるインスタンスで違う強化ルートを持たせたい場合はそれぞれ設定。

---

## 付録：プログラマ向け (Programmer Reference)

> 以下の内容はプログラム内部用語を含み、対象はプログラマと AI agent。前半の利用者向け記述を優先してください。

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardData.cs`
*   **継承元**：`RCG_Asset<RCG_CardData>`
*   **実装インターフェース**：`RCGI_Item` / `RCGI_CardData` / `RCGI_Unloackable`
*   **AssetGroup**：`AssetGroup.EditItems`、sort = `RCG_CardData`
*   **i18n クラス名 key**：直接定義なし；`AssetGroup` 名が EditItems カテゴリ表示に使われる。

### A.2 フィールド対照（外側 `RCG_CardData`）

| コード | エディタ表示 | 型 | 説明 |
|---|---|---|---|
| `m_Data` | カード設定 | `CardData`（内部ネストクラス） | 主データ；A.3 で詳細フィールドを列挙 |
| `m_Effects` | 効果 | `List<RCG_CommonEffect>` | トリガー効果リスト |
| `m_PassiveEffects` | パッシブ効果 | `List<RCG_ConditionalPassiveBattleTag>` | 条件付きパッシブタグ；ステータス問い合わせ用、OnTriggerEffect 不参加 |

### A.3 フィールド対照（ネスト `CardData`）

| コード | エディタ表示 | 型 | Localize Key | 備考 |
|---|---|---|---|---|
| `m_LocalizeName` | 名前(多言語) | `RCG_LocalizeData` | `LocalizeName` | |
| `m_Cost` | コスト | `int` | `Cost` | デフォルト 0 |
| `m_CardType` | カードタイプ | `CardType` (内部 enum) | `CardType` | `Default` / `Chant` |
| `m_ChantTurn` | 詠唱回数 | `IntVariable` | `ChantTurn` | `[Conditional(nameof(m_CardType), false, CardType.Chant)]` |
| `m_Rarity` | レアリティ | `RCG_RarityTagGenData` | `Rarity` | |
| `m_TargetType` | 対象範囲 | `TargetType` (内部 enum) | `TargetType` | 17 種値 |
| `m_UsedTypeTag` | 使用タイプ | `RCG_UsedTypeTagGenData` | `UsedTypeTag` | |
| `m_Price` | 価格 | `int` | `Price` | デフォルト 100 |
| `m_SkillTags` | 専門 | `List<RCG_SkillTagGenData>` | `SkillTags` | |
| `m_CardTags` | カードタグ | `List<RCG_CardTagGenData>` | `CardTags` | |
| `m_BannedEnhence` | 禁止強化ブランチ | `List<RCG_CardEnhenceGenData>` | `BannedEnhence` | |
| `m_EnhencePool` | 強化プール | `RCG_CardEnhenceDropPoolGenData` | — | |
| `m_ApplyEnhenceOnAllTiming` | `ApplyEnhenceOnAllTiming` | `bool` | — | |
| `m_UpgradeLv` | 強化レベル | `int` | `UpgradeLv` | デフォルト 0 |
| `m_EnhenceLocalize` | `EnhenceLocalize` | `RCG_LocalizeData` | — | `+1`/`+2` 表示の代替 |
| `m_CardIcon` | カードアイコン | `RCG_SpriteData` | `CardIcon` | デフォルトは `Card_AngelsWings` |
| `m_CardDescriptionType` | カード説明方式 | `CardDescriptionType` (内部 enum) | `CardDescriptionType` | `Auto` / `Manually` |
| `m_CardDescription` | カード説明 | `RCG_LocalizeData` | `CardDescription` | `[Conditional(... Manually)]` |
| `m_DescriptionTemplate` | 説明テンプレート | `string` | `DescriptionTemplate` | `[Conditional(... Manually)]` |
| `m_Unlock` | 解放 | `RCG_UnlockEntry` | `Unlock` | |
| `m_HideInCodex` | 図鑑で隠す | `bool` | `HideInCodex` | |
| `m_Note` | メモ | `string` | `Note` | |
| `m_Author` | 作者 | `string` | — | |
| `m_DataType` | （非表示） | `DataType` | — | `[UCL_HideOnGUI]`、`BuiltIn` / `InGameRuntime` 等 |

### A.4 主なメソッド

#### インターフェース実装
*   **`AddItem()`** → `RCG_DataService.Ins.m_DeckData.AddCard(ID)`、self を返す；プレイヤーがカードを取得するエントリ。
*   **`ToCardData`** → `this`。
*   **`ToCardBattleData`** → `null`；**廃止**、`RCG_CardBattleData.CreateCard()` を使用。
*   **`Icon` / `IconSpriteData` / `IconTexture`** → `m_Data.m_CardIcon` の対応プロパティ。
*   **`ItemName` / `LocalizedName`** → `m_Data.LocalizeName`（強化レベル接尾語のロジック含む）。
*   **`ShowInfo()`** → `RCG_CardInfoPanel` を開く。
*   **`GetPreviewDamage(target)`** → デフォルト -1（CardData レイヤーは攻撃カードでない）。
*   **`UnlockEntry`** → `m_Data.m_Unlock`。

#### 説明システム
*   **`Description` (property)** → `DisplayDescription()`。
*   **`DisplayDescription(iData)`** → 詠唱カードは詠唱タイトルを前置、`GetDescription` の後に UsedType を後置（`m_ShowTag` の場合）。
*   **`GetDescription(iData, iShowBattleTags)`**：`Manually` モード + `m_CardDescription` 非空 → params 適用後直接返却。`Auto` モード → 「target desc + 各 effect desc + battle tags」を結合。
*   **`GetDescriptionTemplate(iData)`** → `(OnPlay.0)` 形式のテンプレート文字列を生成（Manually モードの `Generate` ボタン用）。
*   **`GetDescriptionParams(iData)`** → 全 effect のパラメータリストを取得（Manually テンプレート置換用）。
*   **`CardDisplayName`** → カード名（詠唱カードは詠唱プレフィックス / 色付き）。
*   **`CostStr`** → `Cost.ToString()`。
*   **`ChantTitle(iData)`** → 「詠唱[X]」文字列（緑）。

#### タグ / 用語 / 協力
*   **`CardTags`** → `m_Data.m_CardTags`（詠唱タグ除く）。
*   **`HasTag(tag)` / `HasTag(tags)`** → 「詠唱カードのとき自動的に `TagChant` を含む」特例ロジックを持つ。
*   **`HasTerm(term)`** → すべての `CardEffects` を再帰クエリ。
*   **`GetBattleTags()`** → すべての `CardEffects` を集約。
*   **`GetPassiveTags(iData)`** → `m_PassiveEffects` を返す；Cost 計算等の属性問い合わせ用。
*   **`GetCollaborators()`** → すべての `CardEffects` の協力者を集約、重複排除。
*   **`GetTagsDescription(showHiddenTags)`** → タグ説明文字列（隠しタグは `[]` でラップ）。

#### 戦闘・フュージョン関連
*   **`CheckPlayable(iData)`** → 使用タイプに `Unplayable` を含む場合は false；他は全 `CardEffects.CheckPlayable` の AND。
*   **`CheckRequireSkill(skill)` / `CheckRequireSkill(skills)`** → 全条件 AND。
*   **`CanPlayThisCard(skills)`** → 「**いずれか一つの専門が一致**」（`CheckRequireSkill` と異なる）。
*   **`GetBattleSettings<T>() / (Type)`** → すべての `m_Effects[i].m_CombineSetting` を再帰。
*   **`OnTriggerEffect(triggerOn, iData)`** → `m_Effects.GetEffects(triggerOn)` を取得；iData が null なら child を作成；`aEffects.TriggerEffects(iData)` を呼び統計を log。

#### 強化ブランチ
*   **`CanEnhence`** → `m_Data.m_UpgradeLv < 1 && !CardRarity.Equals(Cursed)`。
*   **`CreateEnhenceCard(ID)` (private)** → 自身を clone、新 ID 設定、`m_UpgradeLv += 1`、`InGameRuntime` フラグ。
*   **`GetEnhenceBranchs(branchCount)` (yield)** → `EnhencePool` から抽選、`BannedEnhence` と `CheckCondition` でフィルター、各々 clone + 強化データ適用。

#### シリアライズ / データフロー
*   **`SaveToJson(iData)` (static)** → `ID` + `IsRuntimeCardData` を書き込み。
*   **`LoadFromJson(iData)` (static)** → `IsRuntimeCardData = true` → `m_GameRuntimeAsset.GetRuntimeData<RCG_CardData>(aID)`；他は `Util.GetData(aID)`。
*   **`Save()`** → base + `RCG_CardFilter` cache をクリア。
*   **`CloneCard()`** → `SerializeToJson + DeserializeFromJson` で深コピー。
*   **`CreateSelectAssetPage()`** → `RCG_CardDataEditorPage.Create()`。
*   **`OnLoadModule()` (static)** → `RCG_CardFilter.Clear()` + `Util.PrewarmAllAssets()`。
*   **`PreloadData(token)`** → `Preloaded` をマーク後、カード絵、レアリティアイコン、各 effect の `PreloadData` を事前読込。

#### 編集 / プレビュー
*   **`OnGUI(iDataDic)`** → 三段描画：CardData → Effects → PassiveEffects → Preview。Manually モードでは「Generate Description Template」ボタン + 説明パラメータエディタを追加表示。
*   **`Preview(iDic, iIsShowEditButton)`** → アイコン + 名前 + タグ + 説明 + 強化ブランチボタン + 解放条件 + DisplayCardInfos。
*   **`DisplayCardInfos(iShowOnUI)`** → `RCG_BattleSetting.IsShowOnUI` を切替後、完全な InfoPanel 内容（詠唱情報含む）を組立。

### A.5 他システムとの連携

*   **`RCG_CardBattleData`** — 戦場で実際に動作するカードインスタンス；`RCG_CardData` がテンプレート、`RCG_CardBattleData.CreateCard(cardData, isChanted)` がインスタンスを生成。
*   **`RCG_CommonEffect`** — カード効果ユニット；`m_Effects` はその `List`、`m_EffectTriggerOn` + `m_CombineSetting` を含む。
*   **`RCG_ConditionalPassiveBattleTag`** — パッシブ条件付きタグ；`m_Conditions` + `m_Tags` を含み、属性問い合わせに影響するが OnTrigger には参加しない。
*   **`RCG_CardEnhenceData / RCG_CardEnhenceDropPoolGenData`** — 強化ブランチシステム；`GetEnhenceBranchs` のコア。
*   **`RCG_CardFilter`** — カードフィルター cache；`Save()` と `OnLoadModule()` がクリア。
*   **`RCG_DataService.Ins.m_DeckData`** — プレイヤー牌組ストレージ；`AddItem` がここに書き込む。
*   **`RCG_DataService.Ins.m_GameRuntimeAsset`** — runtime カード（CardToItem などが生成したもの）のストレージ；`LoadFromJson` が `IsRuntimeCardData = true` のとき読込。
*   **`UI.RCG_CardInfoPanel` / `RCG_CardDataEditorPage` / `RCG_PreviewEnhenceCardPage`** — 主な UI エントリ。
*   **`RCG_CardDataComparer`**（同ファイル） — ソート helper：専門クラス → レアリティ → コスト降順。

### A.6 既知の課題

*   `ToCardBattleData` は廃止されているが保持（`return null`）；コメントに「之後禁止用這個轉換為 CardBattleData」と明示。
*   `CanEnhence` が「強化回数 ≤ 1」をハードコード；解除には本 property の編集が必要。
*   `m_PassiveEffects` の `GetPassiveTags` 内に `// QWQ23`、`// Append does not work QWQ?` のコメントがあり実装上の懸念をマーク（`Add` で代用）。
*   `InitData()` は virtual だが現在は空実装；サブクラス `RCG_CardBattleData` が実際に使用。
*   旧版 `m_UpgradeCards` フィールドはコメントアウト（強化対象をリスト方式で列挙する設計だった）。
