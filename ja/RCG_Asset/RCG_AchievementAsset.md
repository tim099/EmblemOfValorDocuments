---
title: 成就データ (RCG_AchievementAsset)
description: プレイヤーが解放可能な成就：条件達成時に自動解放；Steam 成就連携可、前提成就を持てる
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 成就データ

> クラス名：`RCG_AchievementAsset`

## 用途

**プレイヤーが解放可能な成就テンプレート**。条件達成時に自動解放（Steam 成就と同期可）。前提成就チェーン、職業 / キャラ関連マーク、カスタム説明上書きをサポート。

`RCG_Asset<RCG_AchievementAsset>` を継承。

## エディタ上の見た目

```
RCG_AchievementAsset: <ID>
    IsClassAchievement / SkillTag             ← 職業関連フラグ（true で対応職業表示）
    IsCharacterAchievement / Character        ← キャラ関連フラグ
    OverrideDescription                       ← 自動説明上書き
    OrderIndex                                ← 表示ソート
    HasSteamAchievement / SteamAchievement    ← Steam 成就連携
    Conditions                                ← 達成条件一覧（AND）
    HasPrerequisiteAchievement / PrerequisiteAchievement  ← 前提成就
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **IsClassAchievement** | — | 「職業成就」かどうか（SkillTag フィールド表示判定） |
| **SkillTag** | IsClassAchievement=true 時 | 対応職業（`RCG_SkillTagGenData`） |
| **IsCharacterAchievement** | — | 「キャラ成就」かどうか |
| **Character** | IsCharacterAchievement=true 時 | 対応キャラ（`RCG_CharacterGenData`） |
| **OverrideDescription** | いいえ | 自動生成説明を上書き（多言語、パラメータハイライト含む） |
| **OrderIndex** | — | ソートインデックス |
| **HasSteamAchievement** | — | Steam 成就連携するか |
| **SteamAchievement** | HasSteamAchievement=true 時 | Steam 成就 ID |
| **Conditions** | いいえ | 達成条件（`RCG_Condition` リスト；**AND** 関係） |
| **HasPrerequisiteAchievement** | — | 前提成就があるか |
| **PrerequisiteAchievement** | HasPrerequisiteAchievement=true 時 | 先に達成する必要のある成就 |

## 動作説明

### 条件チェック (`CheckAchievementCondition`)
*   `m_Conditions` が空 → false 返却（条件なしは「未達成」とみなす、GameChallenge の「条件なしは通過」と逆）。
*   `m_Conditions.CheckConditions_AND` → 全条件満たすと true 返却。
*   前提成就があり本成就条件達成済 → 再帰で前提条件チェック。

### 解放 (`CheckAchievement`)
1. Steam 成就があり**達成済**（`steamAchievement.GetStat() == true`）→ 直接 true 返却（重複発動回避）。
2. `CheckAchievementCondition` 実行。
3. 達成 + Steam 成就あり → `steamAchievement.SetStat(true)` で Steam に報告。

### 説明生成 (`RequirementDes`)
*   `OverrideDescription` 設定済 → 使用（`Term` 色タグでパラメータハイライト含む）。
*   それ以外 → `m_Conditions` の各 `GetShortName()` を連結。

## 注意事項

*   **`m_Conditions` 空時に false 返却**：直感に反する — 空条件は自動達成しない。**この設計は GameChallenge の `IsEmpty || Unlocked` と不整合**、注意必要。
*   **Steam 成就は一度達成するとリセットされない**：本ファイルは「成就取消」フローを提供しない。
*   **OverrideDescription にパラメータハイライト含む**：`RCG_Extensions.TagColors.Term` でパラメータに色付け、編集時にパラメータを `{0}` 等のプレースホルダで書く必要（具体ルールは `RCG_LocalizeData.GetName` 参照）。
*   **前提成就チェーンの条件チェック**：`CheckAchievementCondition` が前提条件を再帰実行 — 前提条件満たすと本成就解放可だが、**前提成就自体が解放済か（GetStat == true）はこの層は本成就ロジックに影響しない**。
*   **コメント `// TODO: use as condition QWQ?`** が示す：`m_SkillTag` / `m_Character` は現状「分類フラグ」のみ、条件として使われていない；将来自動条件化される可能性。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AchievementAsset.cs`
*   **継承**：`RCG_Asset<RCG_AchievementAsset>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_IsClassAchievement` | IsClassAchievement | `bool` | |
| `m_SkillTag` | SkillTag | `RCG_SkillTagGenData` | `Conditional(IsClassAchievement)` |
| `m_IsCharacterAchievement` | IsCharacterAchievement | `bool` | |
| `m_Character` | Character | `RCG_CharacterGenData` | `Conditional(IsCharacterAchievement)` |
| `m_OverrideDescription` | OverrideDescription | `RCG_LocalizeData` | |
| `m_OrderIndex` | OrderIndex | `int` | |
| `m_HasSteamAchievement` | HasSteamAchievement | `bool` | |
| `m_SteamAchievement` | SteamAchievement | `UCL_SteamAchievementEntry` | `Conditional(HasSteamAchievement)` |
| `m_Conditions` | Conditions | `List<RCG_Condition>` | AND |
| `m_HasPrerequisiteAchievement` | HasPrerequisiteAchievement | `bool` | |
| `m_PrerequisiteAchievement` | PrerequisiteAchievement | `RCG_AchievementEntry` | `Conditional(HasPrerequisiteAchievement)` |

### A.3 主要メソッド

*   **`CheckAchievement(triggerEffectData)`** — 入口：Steam 達成済 quick check → 条件 → 前提 → Steam 報告。
*   **`CheckAchievementCondition(triggerEffectData)`** — 純条件チェック（報告なし）；前提を再帰。
*   **`RequirementDes` (property)** — 自動 / 上書き説明。

### A.4 他システムとの連携

*   **`UCL_SteamAchievementEntry`** — Steam SDK 連携。
*   **`RCG_Condition`** — 条件要素。
*   **`RCG_AchievementEntry`** — Asset Entry；デフォルト `CreatorAchievement`。
*   **`RCG_SkillTagGenData / RCG_CharacterGenData`** — 分類用。

### A.5 既知の問題

*   `m_SkillTag` / `m_Character` に `// TODO: use as condition QWQ?` マーク、現状は分類のみ、自動条件化されていない。
*   空 `m_Conditions` で false 返却の動作は直感に反する（GameChallenge と不整合）。
