---
title: 戦闘プリセット (RCG_BattlePresetData)
description: テスト戦闘起動用の「フル設定」：モンスター + プレイヤーキャラ + デッキ + 装備 + 難易度、ワンクリック StartBattle
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 戦闘プリセット

> クラス名：`RCG_BattlePresetData`

## 用途

**テスト戦闘用のクイック設定**。「モンスター + プレイヤーキャラ + デッキ + 装備 + スキル + 難易度」の一式を1つの Asset にパッケージ、Editor 内の `StartBattle` ボタン押下で即座に戦闘突入（大マップ、キャラ選択等のフロースキップ）。**正規ゲームフロー使用ではない**、純粋に QA / デザイナーが数値調整するためのデバッグツール。

`RCG_Asset<RCG_BattlePresetData>` を継承。

## エディタ上の見た目

```
RCG_BattlePresetData: <ID>
    BattleSetGenData   ← 参照戦闘構成
    Players            ← 出戦キャラ一覧
    ExtraCards         ← 開始追加カード（Deck 以外）
    Items              ← 開始アイテム
    Deck               ← 開始デッキ（デフォルト置換）
    Equipments         ← 各キャラ装備（dictionary）
    UnitSkills         ← 各キャラ習得スキル（dictionary）
    AllEquipments      ← 装備していない装備（バックパック）
    EnemyType          ← 敵タイプ（通常 / 精鋭 / ボス）
    DifficultyData     ← グローバル難易度データ
    Difficulty         ← 動的難易度値
    [ボタン] StartBattle ← Editor playing 時にワンクリック開戦
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **BattleSetGenData** | はい | 参照する `RCG_BattleSet`（モンスター配置） |
| **Players** | はい | 出戦キャラ一覧 |
| **ExtraCards** | いいえ | デッキに追加するカード（特定カード組合せテスト用） |
| **Items** | いいえ | 開始アイテム（ポーション、巻物） |
| **Deck** | いいえ | 開始デッキ；非 `Default` 時にプレイヤーデッキ上書き |
| **Equipments** | いいえ | キャラ → 装備一覧の dictionary |
| **UnitSkills** | いいえ | キャラ → スキル一覧の dictionary |
| **AllEquipments** | いいえ | 装備せずバックパックに置く装備 |
| **EnemyType** | いいえ | 敵タイプ |
| **DifficultyData** | いいえ | グローバル難易度設定（HP / Atk 倍率、敵スキルレベル） |
| **Difficulty** | いいえ | 動的難易度値（大マップ難易度の上に重ね） |

## 動作説明

### `StartBattleAsync`
`StartBattle` ボタン押下（Editor playing 時のみ表示）で：
1. BigMapManager 構築し大マップ突入。
2. Quest 突入（EnterQuestSetting でラップ）。
3. `m_DifficultyData` と `m_Difficulty` を `RCG_DataService` に適用。
4. プレイヤーデッキ置換（`m_Deck.ID != Default` の場合）。
5. ExtraCards / Items / Equipments / UnitSkills 等を追加。
6. 戦闘シーン突入。

### プレビュー
基底 BattleSet の Preview を表示し、StartBattle ボタン提供。

## 注意事項

*   **テスト専用**：正規ゲームはここから戦闘突入しない；正規関卡設計に依存しないこと。
*   **`m_Deck.ID == DefaultID` 時はデッキ置換しない**、プレイヤー現デッキ使用 — この Asset の `m_Deck` 指定確認後に有効。
*   **InitActivePowers ロジックはコメントアウト済**：元は当初キャラの初期主動能力を適用、現在は UnitSkill システムで置換、コメントアウトされた。
*   **Editor not playing 時は StartBattle ボタンが見えない** — Play Mode で Developer ページを開く必要あり。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattlePresetData.cs`
*   **継承**：`RCG_Asset<RCG_BattlePresetData>`
*   **AssetGroup**：`EditBattleSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_BattleSetGenData` | BattleSet | `RCG_BattleSetGenData` | |
| `m_Players` | Players | `List<RCG_CharacterGenData>` | |
| `m_ExtraCards` | ExtraCards | `List<RCG_CardGenData>` | |
| `m_Items` | Items | `List<RCG_ItemGenData>` | |
| `m_Deck` | Deck | `RCG_DeckGenData` | |
| `m_Equipments` | Equipments | `Dictionary<RCG_CharacterGenData, List<RCG_EquipmentGenData>>` | |
| `m_UnitSkills` | UnitSkills | `Dictionary<RCG_CharacterGenData, List<RCG_UnitSkillGenData>>` | |
| `m_AllEquipments` | AllEquipments | `List<RCG_EquipmentGenData>` | バックパック |
| `m_EnemyType` | EnemyType | `RCG_EnemyTypeTagGenData` | |
| `m_DifficultyData` | DifficultyData | `RCG_DifficultyData` | |
| `m_Difficulty` | Difficulty | `int` | |

### A.3 主要メソッド

*   **`StartBattleAsync(BigMap, Quest, CancellationToken)`** — 主入口；大マップ構築 → Quest 突入 → 難易度適用 → Deck 置換 → Cards/Items/Equipments/UnitSkills 追加。
*   **`Preview`** — エディタ内に BattleSet preview + StartBattle ボタン表示。

### A.4 他システムとの連携

*   **`RCG_BigMapManager`** / **`RCG_MapManager`** — 突入フローのコア。
*   **`RCG_DataService.Ins.m_DifficultyData / Difficulty`** — 難易度適用。
*   **`RCG_DataService.Ins.m_DeckData`** — デッキ置換ターゲット。
*   **`RCG_MapEventManager.Reset`** — 新戦闘突入前にイベントキューをクリア。

### A.5 既知の問題

*   `// 初始角色能力` セクションはコメントアウト、旧版 ActivePower システム経由の初期能力適用ロジック廃止。
