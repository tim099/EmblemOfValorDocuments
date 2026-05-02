---
title: ユニットスキルデータ (RCG_UnitSkillData)
description: キャラが習得する「スキル」 — 装備に類似だが装備枠を消費しない。パッシブ効果、リーダースキル、習得特典を提供
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ユニットスキルデータ

> クラス名：`RCG_UnitSkillData`

## 用途

**キャラが習得するスキルのテンプレート**。装備に類似だが、装備枠を消費しない。各スキルが可能なこと：
*   パッシブ効果の提供（戦闘中の各種 trigger で発動）
*   「**リーダースキル**」としてマーク（キャラが先頭にいる時のみ発動）
*   習得時に一度きりのイベント発動（例：永続 +1 HP、追加抽選枠解放）

`RCG_Asset<RCG_UnitSkillData>` を継承。実装インターフェース：`RCGI_Status`（戦闘状態システム）/ `RCGI_Unloackable`（解放）。

## エディタ上の見た目

```
RCG_UnitSkillData: <ID>
    Name(多言語)
    Icon                          ← スキルアイコン
    Effects                       ← 戦闘中に発動する効果（OnPlay / OnTurnStart / ...）
    AcquireSkillEvents            ← 習得時に発動する一度きりのイベント
    SkillTags                     ← 必要な専門性（キャラが満たす必要あり）
    Tags                          ← 一般タグ
    UnitSkillDescription / Template ← 説明方式（Auto / Manually）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | スキル表示名（多言語） |
| **Icon** | はい | スキル図示 |
| **Effects** | いいえ | 戦闘中の各種 trigger で実行される効果（OnPlay / OnTurnStart / OnAttack…） |
| **AcquireSkillEvents** | いいえ | スキル習得時に発動するイベント（例：手札数拡張、永続強化） |
| **SkillTags** | いいえ | 必要専門性 — キャラの所持専門性が**全て含む**必要あり（AND 関係） |
| **Tags** | いいえ | 一般物品/スキルタグ（分類、条件判定用） |
| **CanLearnRepeatedly** | — | 重複習得可能か（効果累積） |
| **HideThisSkill** | — | 習得後**スキル欄に非表示**（「習得時特典のみ発動」隠し強化向き） |
| **IsLeaderSkill** | — | **リーダースキル**か：キャラが先頭にいる時のみ Effects 発動 |
| **CanDrop** | — | ドロップ池から取得可能か（false = 特定経路のみ取得） |
| **InitCounters** | いいえ | 初期カウンター値（Effects 内のカウンター系効果に使用） |
| **Unlock** | いいえ | 解放条件 |
| **UnitSkillDescriptionType** | はい | `Auto`（Effects から自動合成）または `Manually`（手書き） |
| **UnitSkillDescription** | Manually 時 | 手書き説明（多言語） |
| **DescriptionTemplate** | Manually 時 | `{(OnPlay.0)}` 等のプレースホルダ含むテンプレート文字列 |

## 動作説明

### 発動判定
`OnUnitState(triggerOn, data)` が各 trigger で：
1. `Effects` から該 trigger の全効果を取り出す。
2. `IsLeaderSkill = true` かつ所有者が**パーティ先頭でない**（`RCG_BattleField.LeadUnit` と ID 比較）場合スキップ。
3. それ以外は順に発動。

### 習得時 (`OnAquireSkill`)
全 `AcquireSkillEvents` を `RCG_MapEventManager` に追加、該キャラに適用（即時または適切なタイミングで fire）。

### 説明生成
*   **Auto**：「リーダースキルタグ → AcquireEvents 説明 → Effects 説明」順に自動連結。
*   **Manually**：`UnitSkillDescription` を本体とし、`GetDescriptionParams()` で `(OnPlay.0)` 等のプレースホルダを置換；`Generate Description Template` ボタンで現 Effects から自動テンプレート生成可。

### Tooltip Infos
`Infos` プロパティが全 effect の `CardInfoData` を集約；リーダースキルは先頭に `BattleTag_LeadAbility` の説明を挿入。

## 注意事項

*   **`SkillTags` は AND 関係**：複数専門性をリストすると「すべて所持」必要；「いずれか」を実現するには複数スキルに分ける。
*   **`HideThisSkill = true`** は「習得時に永続効果を加える、ただしパッシブなし」のスキル向き；プレイヤーには buff アイコンが見えないが効果は発動済み。
*   **`IsLeaderSkill`** はパーティリーダー切替メカニクスと連動；リーダー切替時に Effects は自動再発動しない（`OnPlay` は再 fire しない）。
*   **`CanDrop = false`** のスキルは DropPool から抽選されない、ストーリー解放スキルによく使用。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitSkillData.cs`
*   **継承**：`RCG_Asset<RCG_UnitSkillData>`
*   **実装**：`RCGI_Status` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`
*   **デフォルト Icon パス**：`RCG_SpriteData.SpriteFolder + "/UnitSkills"`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | Localize Key |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` |
| `m_Icon` | Icon | `RCG_SpriteData` | `Icon` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | `Effects` |
| `m_AcquireSkillEvents` | AcquireSkillEvents | `List<RCG_MapEvent>` | `AcquireSkillEvents` |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | `Tags` |
| `m_CanLearnRepeatedly` | CanLearnRepeatedly | `bool` | `CanLearnRepeatedly` |
| `m_HideThisSkill` | HideThisSkill | `bool` | `HideThisSkill` (Header `HideThisSkillDes`) |
| `m_IsLeaderSkill` | IsLeaderSkill | `bool` | `IsLeaderSkill` |
| `m_CanDrop` | CanDrop | `bool` | `CanDrop` |
| `m_InitCounters` | InitCounters | `List<int>` | `InitCounters` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` |
| `m_UnitSkillDescriptionType` | DescriptionType | `CardDescriptionType` | — |
| `m_UnitSkillDescription` | Description | `RCG_LocalizeData` | `Conditional(Manually)` |
| `m_DescriptionTemplate` | Template | `string` | `Conditional(Manually)` |

### A.3 主要メソッド

*   **`OnUnitState(RCG_EffectTriggerOn, TriggerEffectData)`** — 主発動入口；リーダースキル判定含む。
*   **`TriggerOnUnitState(RCG_EffectTriggerOn)`** — 該 trigger に effect があるか（trigger システム用 quick check）。
*   **`OnAquireSkill(RCG_CharacterData)`** — 習得時に `AcquireSkillEvents` を `RCG_MapEventManager` に追加。
*   **`CheckRequireSkill(HashSet<RCG_SkillTagGenData>)`** — キャラの現 skills が全 `m_SkillTags` を含むか。
*   **`Description` (property)** — Auto / Manually の説明ロジック。
*   **`GetDescriptionTemplate / GetDescriptionParams`** — Manually モードのテンプレート生成 / パラメータ抽出。
*   **`Status` (property)** — `new RCG_StatusGenData(StatusType.UnitSkill, ID)` を返す、RCGI_Status 介面実装。

### A.4 他システムとの連携

*   **`RCG_CommonEffect`** — 発動効果単位。
*   **`RCG_MapEvent` / `RCG_MapEventManager`** — 習得特典イベントシステム。
*   **`RCG_BattleField.LeadUnit`** — リーダースキル判定。
*   **`RCG_BattleTag.Util.GetData("BattleTag_LeadAbility")`** — リーダースキルタグ。
