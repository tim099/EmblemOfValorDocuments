---
title: クエストデータ (RCG_QuestData)
description: 1つのクエスト（小マップ）の完全な定義：ノード配置、戦闘池、報酬池、目標、暗霧、ランダム生成
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# クエストデータ

> クラス名：`RCG_QuestData`

## 用途

**1つの「クエスト」（小マップ）の完全な定義**。大マップからクエストノード突入後、ゲームがこのクエストに対応する小マップを生成：ノード配置、ノード間パス、敵配置、報酬池、暗霧ルール、クエスト目標。クエストは複数の生成方式をサポート：手動編集、純ランダム、混合ランダム、殺戮尖塔風、洞窟、遺跡等。

`RCG_Asset<RCG_QuestData>` を継承。実装：`UCL.Core.UI.UCLI_FieldOnGUI`。

## エディタ上の見た目

```
RCG_QuestData: <ID>
    Name / QuestObjective / QuestIcon / BGM / Map prefab
    QuestGoals                 ← クエスト目標（主要 / 副）
    FieldEffects               ← フィールド効果（オン/オフ可）
    StoryPool                  ← このクエストのイベント池
    QuestBattles               ← デフォルト戦闘池（敵タイプごと）
    Card / Item / EquipmentDropPools  ← 各種報酬池（敵タイプごと）
    BattleScene                ← デフォルト戦闘シーン
    QuestType                  ← Default / RandomGen / MixRandom / SlayTheSpire / CaveGen / RuinGen / TestEvents / TestBattles
    [QuestType 対応のサブ設定]      ← 各種 RandomGenData
    DetailSetting              ← 条件 / Tag / 難度 / 暗霧 / マップサイズ / 自動突入等
    MapEditData                ← ノード編集データ（隠し、QuestEditor で編集）
    [ボタン] Open QuestEditor    ← Editor playing 時にノードエディタ開可
    [ボタン] Export Quest Info   ← このクエストのモンスター / 戦闘情報を Markdown でエクスポート
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name / QuestObjective** | はい | クエスト名 / 目標説明（多言語） |
| **QuestIcon** | はい | クエスト図（大マップノード表示） |
| **BGM** | はい | BGM |
| **Map** | はい | 対応する小マップ Prefab |
| **QuestGoals** | いいえ | 主要 / 副クエスト目標（達成で報酬発動） |
| **FieldEffects** | いいえ | このクエストのフィールド効果（オン/オフ可） |
| **StoryPool** | いいえ | このクエストのストーリーイベント池 |
| **QuestBattles** | いいえ | 敵タイプ対応の戦闘池（普通 / 精鋭 / Boss…） |
| **CardDropPools / ItemDropPools / EquipmentDropPools** | いいえ | 敵タイプ対応の報酬池；不在時は EnemyType のデフォルト池にフォールバック |
| **BattleScene** | いいえ | デフォルト戦闘シーン（指定なしで使用） |
| **QuestType** | はい | マップ生成方式（下のどのサブ設定を使うか決定） |
| **DetailSetting** | はい | 詳細（条件 / Tag / 難度 / 暗霧設定 / マップサイズ / `AutoEnterIfOnly` 等） |

`DetailSetting` の内訳：

| サブフィールド | 説明 |
|---|---|
| **CompleteConditions** | 表示前提条件（AND；不適合で非表示） |
| **IncompatibleConditions** | 互斥条件（不適合で非表示） |
| **TopMenuState / QuestTags / QuestLv** | 上部メニュー状態 / Tag リスト / 難度レベル |
| **QuestLength / QuestDarkMistChange** | クエスト長さ / 暗霧変化（`DifficultyData.m_DarkMistMult` で乗算される） |
| **DarkMistRelatedSettings** | 暗霧詳細：移動時上昇、暗霧外観、ノード抵抗自動分配 |
| **Difficulty / AddDifficultyAfterPass** | クエスト基礎難度 / クリア後難度増加 |
| **QuestProgress / QuestEventIcons** | 進度値 / イベントアイコン |
| **CustomizeMapSize / MapWidth / MapHeight** | カスタムマップ長幅 |
| **AutoEnterIfOnly** | 大マップにこのクエストのみの時に自動突入 |

## 動作説明

### ランダム生成 (`GetMapEditData`)
`QuestType` に応じて対応 GenData に分岐：
*   `MixRandom` → `m_QuestMixRandomData.RandomGen` + `BalanceDistanceIteration`
*   `RandomGen` / `SlayTheSpire` / `SlsHorizontal` / `CaveGen` / `TestEvents` → 各々の RandomGen
*   `Default` → 直接予存の `m_MapEditData` を読込
*   `RuinGen` / `TestBattles` → **未実装**（log error）

### クエスト目標自動生成 (`GenerateQuestGoals` static)
クエストに目標が設定されていない時、static メソッドが現大マップ段階の報酬池に基づいて生成：
*   **主目標**（hash seed 依拠）：3 種ローテーション — 装備報酬 / ユニットスキル報酬 / カード報酬；要求は「Boss 撃破」（Boss tag クエスト）または「精鋭 N 体撃破」。
*   **副目標**：3 種ローテーション — 魂値閾値 / 暗霧レベル閾値 / 一般敵撃破数；報酬は金銭 / 魂値 / アイテム。

### チャレンジ目標付加 (`AppendGameChallengeGoals` static)
`RCG_DataService.Ins.m_ChallengeGoalDatas` の未完了チャレンジ目標を `m_IsChallengeGoal` でマークしクエスト目標に付加。

### クリア (`QuestComplete`)
`Difficulty += m_DetailSetting.m_AddDifficultyAfterPass`：このクエストクリア後に難度が永続的に上昇。

### 進度 (`QuestProgress`)
*   `BigMapType.Loop`（旧版 Loop 大マップ）→ 直接 `QuestProgressToComplete` 返却。
*   新版（Tag カウント）→ `BigMapData.QuestProgressTags` のいずれかの Tag を含む → 1 返却、それ以外 0 返却。

### 条件チェック
*   **`IsPrerequirementMet(selected)`** — 既クリア → false；`CompleteConditions` 全 AND 通過。
*   **`IsQuestAvailable(selected)`** — 既クリア → false；`IncompatibleConditions` 全 AND 通過。

## 注意事項

*   **`QuestType` を切り替える時は `MapEditData` クリアを忘れずに**：旧ランダムデータが残ると干渉。`RemoveRandomNodes` でクリア可。
*   **`AutoEnterIfOnly` は常駐クエスト / 開始祝福シーンのみ有効**：大マップにこのクエストしか残っていない時に自動突入チェック。
*   **目標自動生成は deterministic**：quest IDs の hash で seed 計算、**同組クエストに繰返し突入で同じ目標生成**。
*   **`RuinGen` / `TestBattles` 未実装**：選択しても LogError で実生成しない。
*   **`m_FieldEffects` の要素は `ToggleableFieldEffectGenData`**：閉じる可、runtime に `FieldEffects` property で enabled をフィルタ。
*   **`Export Quest Info` ボタン**：モンスター配置 Markdown を `Assets/Export/Quest/<ID>.md` にエクスポート、外部で戦闘配置レビューに便利。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_QuestData.cs`
*   **継承**：`RCG_Asset<RCG_QuestData>`
*   **実装**：`UCL.Core.UI.UCLI_FieldOnGUI`
*   **AssetGroup**：`EditQuestSetting`
*   **CurQuest** (static) — 最近構築された RCG_QuestData、外部アクセスに便利

### A.2 フィールドマッピング（抜粋）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` / `m_QuestObjective` | Name / Objective | `RCG_LocalizeData` | |
| `m_QuestIcon` / `m_BGM` / `m_Map` | アイコン / BGM / マップ | `RCG_SpriteData` / `RCG_BGMGenData` / `RCG_PrefabResData` | |
| `m_QuestGoals` | QuestGoals | `List<RCG_QuestGoalData>` | |
| `m_FieldEffects` | FieldEffects | `List<ToggleableFieldEffectGenData>` | |
| `m_Tags` | Tags | `List<RCG_EventTagGenData>` | |
| `m_StoryPool` | StoryPool | `RCG_StoryDropPoolGenData` | |
| `m_MapEditData` | MapEditData | `MapEditData` | `[UCL_HideOnGUI]` |
| `m_QuestBattles` | QuestBattles | `Dictionary<RCG_EnemyTypeTagGenData, RCG_BattleSetDropPoolGenData>` | |
| `m_CardDropPools / m_ItemDropPools / m_EquipmentDropPools` | 各報酬池 | `Dictionary<EnemyType, *DropPoolGenData>` | |
| `m_BattleScene` | BattleScene | `RCG_BattleSceneGenData` | |
| `m_QuestType` | QuestType | `QuestType` enum | 8 モード |
| `m_QuestRandomGenData / m_QuestMixRandomData / m_QuestCaveGenData / m_SlayTheSpireGenData / ...` | 各 GenData | 各々の class | `Conditional(対応 QuestType)` |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | |

### A.3 主要メソッド

*   **`GetMapEditData(isLoadMap)`** — 主入口；QuestType でランダム生成または予存読込。
*   **`QuestComplete()`** — クリア後 Difficulty +=。
*   **`GenerateQuestGoals` (static)** — 主 + 副目標自動生成。
*   **`AppendGameChallengeGoals` (static)** — チャレンジ目標付加。
*   **`GetCardDropPool / GetItemDropPool / GetEquipmentDropPool / GetBattleSet`** — 敵タイプ対応の池取得；不在時 fallback。
*   **`IsPrerequirementMet / IsQuestAvailable`** — 表示条件チェック。
*   **`QuestProgress` (property)** — 新旧 2 種類の計算方式。
*   **`SaveGame / LoadGame`** — ランダム生成の MapEditData シリアライズ。
*   **`ExportQuestInfoToJson`** — モンスター情報 Markdown を `Assets/Export/Quest/` にエクスポート。
*   **`OnGUI(field, dataDic, params)`** — カスタム描画（PreviewTexture + Export ボタン）。

### A.4 他システムとの連携

*   **`MapEditData`** — ノード / パスコンテナ。
*   **`RCG_QuestDropPool`** — Quest のランダム池。
*   **`RCG_BigMapData.QuestProgressTags`** — 進度判断依拠。
*   **`RCG_QuestRandomGenData / RCG_QuestMixRandomData / RCG_QuestCaveGenData / RCG_SlayTheSpireGenData / ...`** — 各 QuestType のジェネレータ。
*   **`RCG_QuestGoalData / QuestGoalRequirementData / GoalReward*`** — 目標 / 報酬システム。
*   **`RCG_DataService.Ins.m_ChallengeGoalDatas`** — チャレンジ目標ソース。
*   **`UI.RCG_QuestEditorUI`** — ノードエディタ。

### A.5 既知の問題

*   `RuinGen` / `TestBattles` 未実装、選択で LogError のみ。
*   `GenerateQuestGoals` は hash seed 使用、ランダム seed リセットメカニズムなし — 同組 IDs で常に同じ目標生成。
*   `OnGUI` 内の旧版グラフィック描画コード（GL.Begin / GL.LINES）は大段でコメントアウト、現在は PreviewTexture 使用。
*   多くの `// QWQ` / `// QWQ23` マークが未リファクタリングと旧版移行ポイントを示す。
