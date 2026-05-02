---
title: 難度データ (RCG_DifficultyData)
description: 難度レベルのグローバル倍率設定：HP / 攻撃 / ショップ価格 / 暗霧 / フィールド効果 / 解放前提難度
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 難度データ

> クラス名：`RCG_DifficultyData`

## 用途

**難度レベルの設定テンプレート** — 簡単 / 普通 / 困難 / 悪夢 / チュートリアル等は全て異なる ID の `RCG_DifficultyData`。本データはその難度下の：
*   HP / 攻撃 / 商品価格の倍率
*   敵スキル基礎レベル加算
*   開始リソース
*   キャンプファイア減少率
*   アイテム使用回数制限
*   永続適用のフィールド効果
*   この難度突入の前提条件（どの難度をクリア必要か）
を決定。

`RCG_Asset<RCG_DifficultyData>` を継承。

## エディタ上の見た目

```
RCG_DifficultyData: <ID>
    Name / IconSprite / IsHidden / SortOrder
    BaseLevel              ← モンスター基礎レベル（更に UnitLevelData 曲線で換算）
    HealthMult / AtkMult / PriceMult / SoulPriceMult / DarkMistMult
    SellPriceMult          ← アイテム/装備の売却価格倍率
    InitResources          ← 開始追加リソース
    CampFireDecreaseRate / CampFireHealPercentageMult
    ItemUsageLimit         ← アイテム使用回数制限
    AdditionalLength / AdditionalQuestProgress
    EnemySkillLevel        ← 敵基礎スキルレベル
    FieldEffects           ← 永続フィールド効果
    RequiredCompletedDifficulty  ← 前提条件（OR：いずれかクリアでこの難度解放）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name / IconSprite** | はい | 表示名とアイコン |
| **IsHidden** | — | 難度メニュー非表示（テスト用） |
| **SortOrder** | はい | メニューソート |
| **BaseLevel** | はい | モンスター基礎レベル（`RCG_UnitLevelData` 曲線で最終レベルに換算） |
| **HealthMult / AtkMult** | はい | HP / 攻撃倍率（`UnitLevelData.GetMaxHP / GetAtkMult` の結果に適用） |
| **PriceMult / SoulPriceMult** | はい | 商人 / 魂ショップ価格倍率 |
| **DarkMistMult** | はい | 暗霧蓄積速度倍率 |
| **SellPriceMult** | — | プレイヤーの物品売却倍率（デフォルト 0.5） |
| **InitResources** | いいえ | 開始追加リソース |
| **CampFireDecreaseRate** | — | キャンプファイア使用回数の減衰率（毎使用後に減る比率） |
| **CampFireHealPercentageMult** | — | キャンプファイア治療パーセンテージ倍率 |
| **ItemUsageLimit** | — | 戦闘ごとのアイテム使用上限（0 = 無限） |
| **AdditionalLength** | — | 大マップ長さ加算 |
| **AdditionalQuestProgress** | — | Quest 進度加算 |
| **EnemySkillLevel** | — | 敵基礎スキルレベル加算（`MonsterLevelActionData.GetAction` の level に影響） |
| **FieldEffects** | いいえ | 永続適用のフィールド効果一覧 |
| **RequiredCompletedDifficulty** | いいえ | 前提難度（**OR**：いずれかクリアでこの難度解放） |

## 動作説明

### グローバル倍率の作用箇所
これらの mult 値は様々な箇所で乗算される：
*   `UnitLevelData.GetMaxHP / GetAtkMult` で `HealthMult / AtkMult` 適用。
*   ショップ UI の価格表示時に `PriceMult / SoulPriceMult` 適用。
*   物品売却価格計算時に `SellPriceMult` 適用。
*   暗霧蓄積ロジックで `DarkMistMult` 適用。
*   `MonsterLevelActionData.GetAction` で `EnemySkillLevel` 加算。

### `OnVictory()`
勝利して継続選択時に呼出、自動 `++m_EnemySkillLevel` — 「無限モード」で勝利毎に敵スキルがレベルアップ。

### 説明
`LocalizedDescription` がローカライズシステムから key `<ID>_Description` 取得、**説明は zh-Hant.txt / en.txt に書く**、Asset に直接編集は無効。

## 注意事項

*   **`RequiredCompletedDifficulty` は OR 関係**：いずれか満たすと解放。多階解放を設計する時に全許容前提を列挙する必要。
*   **`OnVictory` は永続的に Asset 修正**：`m_EnemySkillLevel` はシリアライズフィールド、勝利後に永続化。**つまり毎勝利後に同難度を選ぶと更に難しくなる** — これは「無限モード」の設計、バグではない。
*   **`Difficulty_Tutorial`** (`TutorialDifficultyID`) はチュートリアル専用 ID、ゲーム開始時に強制使用。
*   **`m_FieldEffects` は永続適用**：毎戦闘開始時に適用、削除不可。
*   **`Description` は i18n key 使用**：Asset 上の description フィールド直接編集は無効、言語ファイルに追加必須。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DifficultyData.cs`
*   **継承**：`RCG_Asset<RCG_DifficultyData>`
*   **AssetGroup**：`EditGameSetting`
*   **デフォルト ID**：`Difficulty_Normal`（コンストラクタデフォルトでもある）

### A.2 フィールドマッピング（抜粋）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_SpriteData` | |
| `m_IsHidden` | IsHidden | `bool` | |
| `m_SortOrder` | SortOrder | `int` | デフォルト 1 |
| `m_BaseLevel` | BaseLevel | `int` | デフォルト 1 |
| `m_FieldEffects` | FieldEffects | `List<RCG_FieldEffectGenData>` | |
| `m_HealthMult` / `m_AtkMult` / `m_PriceMult` / `m_SoulPriceMult` / `m_DarkMistMult` | 各 Mult | `float` | デフォルト 1f |
| `m_SellPriceMult` | SellPriceMult | `float` | デフォルト 0.5f |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_CampFireDecreaseRate` / `m_CampFireHealPercentageMult` | キャンプファイア関連 | `float` | |
| `m_ItemUsageLimit` | ItemUsageLimit | `int` | |
| `m_AdditionalLength` / `m_AdditionalQuestProgress` | 進度加算 | `float / int` | |
| `m_EnemySkillLevel` | EnemySkillLevel | `int` | |
| `m_RequiredCompletedDifficulty` | RequiredCompletedDifficulty | `List<RCG_DifficultyGenData>` | OR 関係 |

### A.3 主要メソッド

*   **`OnVictory()`** — `++m_EnemySkillLevel`（無限モード increment）。
*   **`LocalizedName / LocalizedDescription`** — i18n 対応；description は `<ID>_Description` key 経由。
*   コンストラクタデフォルト `ID = "Difficulty_Normal"`。

### A.4 他システムとの連携

*   **`RCG_UnitLevelData.GetMaxHP / GetAtkMult`** — HealthMult / AtkMult 適用。
*   **`RCG_MonsterLevelActionData.GetAction`** — EnemySkillLevel 加算。
*   **`RCG_DataService.Ins.m_DifficultyData`** — runtime 適用箇所。
*   **`RCG_DifficultyGenData`** — Asset Entry；`TutorialDifficulty` がチュートリアル専用。

### A.5 既知の問題

*   `OnVictory` 永続的に `EnemySkillLevel` を増加 → Asset に書戻し；プレイヤーが異なるセーブで同難度をプレイすると累積されたレベル加算が表示される（**有意の「無限挑戦」メカニズム設計**）。
