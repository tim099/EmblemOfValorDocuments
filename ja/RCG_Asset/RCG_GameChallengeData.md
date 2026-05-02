---
title: ゲームチャレンジ (RCG_GameChallengeData)
description: グローバルゲームチャレンジ目標：このクリア目標を達成して初めて「チャレンジ達成」とみなされる（Achievement と異なる）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ゲームチャレンジ

> クラス名：`RCG_GameChallengeData`

## 用途

**グローバルな「クリア目標」設定**。例：「最終ボス撃破」「アイテム不使用クリア」「3ターン以内に戦闘終了」。チャレンジは「**クリア目標**」レベル、純粋に取得する `RCG_AchievementAsset` と異なる — チャレンジは「このゲームがクリア済か」を決定する。

`RCG_Asset<RCG_GameChallengeData>` を継承。実装：`RCGI_Unloackable`。

## エディタ上の見た目

```
RCG_GameChallengeData: <ID>
    Name             ← チャレンジ名（多言語、デフォルト "GameChallenge"）
    Unlock           ← 解放条件
    ChallengeGoals   ← 達成条件（複数 RCG_QuestGoalData で構成）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | チャレンジ表示名（多言語） |
| **Unlock** | いいえ | 解放条件；空 = デフォルト解放 |
| **ChallengeGoals** | はい | 達成条件一覧（`RCG_QuestGoalData`）、全達成でクリアとみなす |

## 動作説明

### 解放判定 (`IsUnlocked`)
*   `m_Unlock.IsEmpty || m_Unlock.Unlocked`：条件設定なしまたは条件達成 → 解放済。

### 達成判定
本ファイル自体は「達成済か」の判定を担当しない；外部の quest manager や challenge manager が `m_ChallengeGoals` の進度をチェックする。

## 注意事項

*   **デフォルト ID `Challenge_FinalBoss`** は「最終ボス撃破」の標準チャレンジ、多くのルートクリアでこれを使用。
*   **`RCG_AchievementAsset` との違い**：成就は「附加栄誉」（解放条件達成 → 報酬 / 表示）；チャレンジは「**クリア判定**」（達成 → 当該局終了 / 集計発動）。
*   **`m_Name` のデフォルト値は `"GameChallenge"`**：UI で意味のあるタイトルを表示するには改名必須。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_GameChallengeData.cs`
*   **継承**：`RCG_Asset<RCG_GameChallengeData>`
*   **実装**：`RCGI_Unloackable`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | デフォルト `"GameChallenge"` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_ChallengeGoals` | ChallengeGoals | `List<RCG_QuestGoalData>` | |

### A.3 主要メソッド

*   **`IsUnlocked` (property)** — `m_Unlock.IsEmpty || m_Unlock.Unlocked`。
*   **`UnlockEntry`** — `m_Unlock`。
*   **`LocalizedName`** — `m_Name.Name`。

### A.4 他システムとの連携

*   **`RCG_QuestGoalData`** — クリア条件要素。
*   **`RCG_GameChallengeGenData`** — Asset Entry；デフォルト `Challenge_FinalBoss`。
*   **`RCG_UnlockEntry`** — 解放システム。

### A.5 既知の問題

*   `Preview` 実装なし；基底クラスデフォルト描画使用。
