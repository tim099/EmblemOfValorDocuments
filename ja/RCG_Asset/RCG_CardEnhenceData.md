---
title: カード強化データ (RCG_CardEnhenceData)
description: カード強化の「分岐テンプレート」：強化条件、effect 追加、コスト変更、カードタイプ変更、使用タイプ禁止
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カード強化データ

> クラス名：`RCG_CardEnhenceData`

## 用途

**カード強化の「分岐テンプレート」**。各 Asset が「強化が何をするか」を記述：新 effect 追加、コスト変更、カードタイプ変更、タグ追加、改名（α、β、γ 等）。カード側の `RCG_CardData.m_EnhencePool` が `RCG_CardEnhenceDropPool` を参照、池がドロップするのがこれら `RCG_CardEnhenceData` 分岐。

`RCG_Asset<RCG_CardEnhenceData>` を継承。

## エディタ上の見た目

```
RCG_CardEnhenceData: <ID>
    LocalizeName              ← 強化名（α / β / γ；空白で +）
    Rarity                    ← 強化分岐レアリティ
    Conditions                ← AND 条件群（どのカードがこの強化を使えるか）
    Effects                   ← 強化が追加する新効果
    CardTags                  ← 強化が追加するカードタグ
    EnhenceSettings           ← 既存 effect を変更する追加設定（条件式パッケージ）
    BannedUsedType            ← 特定使用タイプのカードがこの強化を使うことを禁止
    CostAlter                 ← コスト変化（+/-）
    SetCardType / CardType    ← カードタイプを強制変更するか（例：Chant → Default）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **LocalizeName** | いいえ | 強化命名替代（α / β / γ 等）；空白で `+` プレフィックス |
| **Rarity** | はい | 強化分岐レアリティ（ドロップ重みに影響） |
| **Conditions** | いいえ | AND 条件式：この強化を使うカードは全条件を満たす必要 |
| **Effects** | いいえ | 強化がカードに追加する新効果 |
| **CardTags** | いいえ | 強化が追加するカードタグ（重複時は追加しない） |
| **EnhenceSettings** | いいえ | 既存 effect を更に変更する設定（例：「OnPlay 最初の effect でダメージ +5」） |
| **BannedUsedType** | いいえ | 特定使用タイプのカードを禁止（例：`Unplayable` カードを強化不可に） |
| **CostAlter** | — | コスト変化値（正負可） |
| **SetCardType** | — | カードタイプを強制変更するか |
| **CardType** | SetCardType=true 時 | どのタイプに変更するか（`Default` / `Chant`） |

## 動作説明

### 条件チェック (`CheckCondition`)
1. `m_BannedUsedType` がターゲットカードの `UsedType` を含む → 使用不可。
2. `m_Conditions.CheckCondition(data)` 不通過 → 使用不可。
3. 各 `EnhenceSettings` の `CheckCondition(data)` 全通過 → 使用可能。

### 適用 (`Enhence(card)`)
1. `m_LocalizeName` をカードの `m_EnhenceLocalize` に書込（表示用）。
2. enabled な全 `Effects` を追加（深いコピー）。
3. `Cost += m_CostAlter`。
4. `SetCardType` → 新タイプ適用。
5. `CardTags` → タグ追加（重複なし）。
6. 各 `EnhenceSettings` で `Enhence(enhenceData)` 実行（既存 effect 変更）。

## 注意事項

*   **強化は「永続適用」**：clone されたカードは改名 / effect 追加 / コスト変更されるが、原カードは変化なし。
*   **`BannedUsedType` は通常 `Unplayable` を含む**：「手動使用不可」のカード（システムカード、隠し効果カード）は強化されるべきでない；以前自動的に `Unplayable` を補完するデシリアライズロジックがあった（コメントアウト済）。
*   **`EnhenceSettings` と `Effects` の違い**：`Effects` は「新 effect を追加」；`EnhenceSettings` は「既存 effect を変更」（例：最初のダメージ effect を +5 ダメージに変更）。
*   **空白 `LocalizeName`** は `+1` / `+2` のデフォルト表示を意味；非空白は `+α` / `α` 等で置換される。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardEnhenceData.cs`
*   **継承**：`RCG_Asset<RCG_CardEnhenceData>`
*   **AssetGroup**：`EditItems`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_LocalizeName` | LocalizeName | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Conditions` | Conditions | `RCG_CE_AND_Condition` | AND 群 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | |
| `m_EnhenceSettings` | EnhenceSettings | `List<RCG_CardEnhenceSetting>` | |
| `m_BannedUsedType` | BannedUsedType | `List<RCG_UsedTypeTagGenData>` | |
| `m_CostAlter` | CostAlter | `int` | |
| `m_SetCardType` | SetCardType | `bool` | |
| `m_CardType` | CardType | `CardType` enum | `Conditional(SetCardType)` |

### A.3 主要メソッド

*   **`CheckCondition(EnhenceData)`** — BannedUsedType + Conditions + EnhenceSettings の三層チェック。
*   **`Enhence(RCG_CardData)`** — 主適用ロジック：名前書込 → effects 追加 → cost 変更 → cardType 変更 → cardTags 追加 → EnhenceSettings 適用。
*   **`EnhenceSettings` (property)** — `IsEnable = true` の settings をフィルタ。

### A.4 他システムとの連携

*   **`RCG_CardData.m_EnhencePool`** / **`GetEnhenceBranchs`** — このデータを参照する入口。
*   **`RCG_CardEnhenceDropPool`** — 強化分岐ランダム池。
*   **`RCG_CardEnhenceCondition.EnhenceData`** — 条件チェックと適用時の伝達コンテナ。
*   **`RCG_CardEnhenceSetting`** — 既存 effect を変更するサブ設定。
*   **`RCG_CardEnhenceGenData`** — Asset Entry ラッパー；デフォルト ID = `"Defense"`。

### A.5 既知の問題

*   `DeserializeFromJson` 内に「自動的に Unplayable を BannedUsedType に補完」するロジックがコメントアウト、旧版自動動作の停止を示す。
