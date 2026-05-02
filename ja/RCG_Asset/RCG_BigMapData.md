---
title: 大マップデータ (RCG_BigMapData)
description: 完全な大マップ（章）の骨格設定：クエスト生成モード、ステートマシン、暗霧、解放、開始装備、進度ルール
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 大マップデータ

> クラス名：`RCG_BigMapData`

## 用途

**1つの完全な大マップ章節の骨格設定**。各 BigMap は完全なゲーム旅程：開始からボスまで、途中で各種クエストノードを踏み、クエストタグで進度推進。本データの定義：
*   クエスト生成モード（デフォルトルール / クエスト池 / 池+混合）
*   多段ステートマシン（`MapState`：各段階に独自のクエスト池、報酬池、次段階突入条件）
*   暗霧設定、解放条件、開始装備、利用可能チャレンジ目標
*   常駐 + 最終クエスト池

`RCG_Asset<RCG_BigMapData>` を継承。実装：`RCGI_Unloackable`。

## エディタ上の見た目

```
RCG_BigMapData: <ID>
    Name / Description / QuestIcon / BGM / BigMap (prefab)
    BigMapBackground
    TopMenuState
    QuestGenerateType         ← Default / QuestPool / QuestPoolMix
    MaxShowingQuestCount      ← 同時に最大表示クエスト数（Default モード）
    QuestProgressToComplete   ← 最終関解放に必要なクエスト進度（0 = 無視）
    QuestPoolSetting          ← (QuestPool/QuestPoolMix) 多段ステートマシン
    PermanentQuests           ← (QuestPoolMix) 常駐クエスト
    FinalQuests               ← (QuestPoolMix) 最終クエスト
    QuestProgressTags         ← Boss クエストの Tag（Forest/Mountain/Boss…）
    DetailSetting             ← 隠しフラグ / チュートリアル設定 / 解放 / 開始装備 等
```

## 主要フィールド（抜粋）

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **QuestGenerateType** | はい | `Default`（デフォルトルール）/ `QuestPool`（純クエスト池）/ `QuestPoolMix`（池+常駐+最終混合） |
| **MaxShowingQuestCount** | Default 時 | 同時選択可能クエスト数（デフォルト 3） |
| **QuestProgressToComplete** | — | 最終関解放に必要なクエスト進度数；> 0 で進度インジケータ表示 |
| **QuestPoolSetting** | QuestPool/Mix 時 | 多段ステートマシン（各 `MapState` に独自のクエスト池 / 報酬池 / 突入条件） |
| **PermanentQuests** | QuestPoolMix 時 | 常駐クエスト（休憩点等）：常時出現し**表示数制限にカウントしない** |
| **FinalQuests** | QuestPoolMix 時 | 最終クエスト（Final 条件達成後に出現） |
| **QuestProgressTags** | Default/Mix 時 | どの Tag を「進度クエスト」とみなすか（通常は Boss タグ） |
| **DetailSetting** | はい | 詳細設定：HideThisBigMap / RandomQuestOrder / Difficulty / EnterBigMapTutorial / ShowMode / IsTutorial / BigMapType (Default/Loop) / Unlock / OverridingCharacters / StartingEquipments / AvailableChallenges 等 |

`MapState` 各段階データ：
*   `MaxShowingQuestCount` / `QuestDropSettings`（条件式重み付きクエスト池一覧）
*   `EquipmentRewardPools / UnitSkillRewardPools / CardRewardPools / ItemRewardPools`：段階専用報酬池
*   `NextStateConditions`（OR）/ `FinalConditions`（OR）

## 動作説明

### `EnterBigMap(newLoop)`
新一局目時に進度と通過クエスト記録をリセット。

### `GenerateQuests()`
`QuestGenerateType` に応じて分岐：
*   **Default**：ここでは処理せず、他 manager に委譲。
*   **QuestPool**：純粋に現 MapState のクエスト池から抽選。
*   **QuestPoolMix**：複雑なロジック —
    1. 開始祝福イベント（初回突入時）
    2. 前クエストが常駐/特殊 → 常駐クエスト + スキップオプション生成
    3. 前クエストが Boss → 再ランダム生成（ProgressTag でフィルタ）
    4. 最終条件達成 → 最終クエストのみ表示
    5. 一般的：環境 Tag で重複除外、Boss + 一般クエストを比率で混合

### `QuestStart(questData)`
クエスト突入時に `CurState.CheckFinal()` が true なら → 最終マップとマーク（勝利後にクリア UI ポップ）。

### `PassedQuest(questId, progress)`
クエストクリア時：
1. `m_PassedQuests` に当該 ID 追加。
2. `MapStatePassQuestCount` / `PassQuestCount` Tag に書込（GameTag システム）。
3. `CurState.CheckTransition()` チェック → 次の MapState に突入（MapStatePassQuestCount もリセット）。

### `QuestCompleteAsync()`
最終マップなら → `RCG_VictoryUI` 表示、プレイヤーが継続選択（継続 = 次 Loop 突入）。

## 注意事項

*   **`QuestGenerateType.Default`** は旧版ルール、新マップは `QuestPoolMix` 推奨。
*   **`PermanentQuests` は数量制限にカウントしない**：休憩点 / 永久ショップ等の「常時表示」クエストノードはここに置く。
*   **`m_QuestProgressToComplete` と `FinalQuests`** は併用：進度未達時は FinalQuests 出現せず。
*   **`HideThisBigMap`** が true となる原因は `m_HideThisBigMap` だけでない：「`OverrideAvailableGameModes` が現 GameVersion を含まない」も非表示にする。Demo 版で部分章をロックするのにこのメカニズム使用。
*   **`ShowMode`** がチュートリアルモード下の表示を制御：`ShowIfEnableTutorial` / `ShowIfDisableTutorial` でチュートリアル状態切替に対応。
*   **`Tutorial` 固定 ID**：`BigMap_Challenge_Card`、チュートリアル専用大マップ。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_BigMapData.cs`
*   **継承**：`RCG_Asset<RCG_BigMapData>`
*   **実装**：`RCGI_Unloackable`
*   **AssetGroup**：`EditQuestSetting`
*   **定数**：`TutorialName = "BigMap_Challenge_Card"`

### A.2 フィールドマッピング（抜粋）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` / `m_Description` | Name / Description | `RCG_LocalizeData` | |
| `m_QuestIcon` | QuestIcon | `RCG_SpriteData` | |
| `m_BGM` | BGM | `RCG_BGMGenData` | |
| `m_BigMap` | BigMap | `RCG_PrefabResData` | 大マップ prefab |
| `m_BigMapBackground` | BigMapBackground | `RCG_SpriteData` | |
| `m_TopMenuState` | TopMenuState | `TopMenuState` enum | |
| `m_QuestGenerateType` | QuestGenerateType | `QuestGenerateType` enum | |
| `m_MaxShowingQuestCount` | MaxShowingQuestCount | `int` | `Conditional(Default)` |
| `m_QuestProgressToComplete` | QuestProgressToComplete | `int` | |
| `m_QuestPoolSetting` | QuestPoolSetting | `QuestPoolSetting`（入れ子） | `Conditional(QuestPool / QuestPoolMix)` |
| `m_PermanentQuests` / `m_FinalQuests` | 常駐 / 最終 | `MapState` | `Conditional(QuestPoolMix)` |
| `m_QuestProgressTags` | QuestProgressTags | `List<RCG_QuestData.QuestTag>` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting`（入れ子） | |

### A.3 主要メソッド

*   **`EnterBigMap(newLoop)`** — 大マップ突入；新局目で進度リセット。
*   **`GenerateQuests()`** — 主クエスト生成入口。
*   **`GenerateQuestsQuestPool()`** / **`GenerateQuestsQuestPoolMix()`** — 2セットの生成実装。
*   **`QuestStart(questData)`** — クエスト開始；最終マップフラグ設定。
*   **`PassedQuest(questId, progress)`** — クリア時に Tag 更新、次 MapState 突入。
*   **`QuestCompleteAsync(token, map)`** — クエスト完了；最終マップで勝利 UI ポップ。
*   **`AutoTriggerQuests()`** — 開始祝福イベント / 単一クエスト自動突入。
*   **`CurState`** — `m_QuestPoolSetting.m_MapStates[saveData.m_State]`。
*   **`HideThisBigMap`** / **`ShowThisBigMap`** — 多重判定可視性。
*   **`MapState.GetQuestDropPool`** — 条件式重みに従いランダム池抽選。
*   **`MapState.CheckTransition` / `CheckFinal`** — 状態切替 / 最終判定（OR）。

### A.4 他システムとの連携

*   **`RCG_BigMapManager`** — runtime 主入口。
*   **`RCG_QuestData`** / **`RCG_QuestDropPool`** — クエストシステム。
*   **`RCG_BigMapGenData`** — Asset Entry；デフォルト `BigMap_Challenge_Card`。
*   **`BigMapSaveData`** — runtime 進度保存。
*   **`RCG_DifficultyData.m_AdditionalQuestProgress`** — 難度付加進度。
*   **`RCG_GameSettingData.m_GameVersion`** — Demo 版ロック判定。
*   **`RCG_TutorialService`** — チュートリアルモード可視性判定。

### A.5 既知の問題

*   `// QWQ23` マークが各所に散在、旧版データフォーマット移行と未確認ロジックを示す。
*   `GenerateQuestsQuestPoolMix` のロジックは極長（300+ 行の単一メソッド）、コメントアウト済 LogError 多数、メンテナンス困難。
