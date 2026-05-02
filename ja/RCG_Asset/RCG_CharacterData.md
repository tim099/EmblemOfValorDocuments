---
title: プレイヤーキャラデータ (RCG_CharacterData)
description: プレイヤーが選択 / パーティに加入するキャラのテンプレート。HP、初期デッキ、初期スキル、解放条件、加入特典
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# プレイヤーキャラデータ

> クラス名：`RCG_CharacterData`

## 用途

**プレイヤーが選択またはストーリー中にパーティに加入するキャラのテンプレート**。各英雄、仲間、勧誘可能 NPC は1つの `RCG_CharacterData`。含む内容：基本情報、初期 HP、初期デッキ、初期スキル/専門性、解放条件、パーティ加入時の「加入デッキ (JoinDeck)」など。

`RCG_Asset<RCG_CharacterData>` を継承。実装インターフェース：`UCLI_ShortName` / `RCGI_Unloackable`（解放システム）。

## エディタ上の見た目

```
RCG_CharacterData: <ID>
    Data (UnitData)            ← 編集時に実際に修正する設定（HP / Deck / Skills / Order / Unlock）
    RuntimeData                ← ゲーム実行時のみ変動するデータ（装備、現スキルなど）
    Preview (右側)              ← アバター / 名前 / 最大 HP / スキル / デッキ
```

## 主要フィールド（Data 内部）

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | キャラ表示名（多言語） |
| **MaxHp** | はい | 最大生命値 |
| **Avatar** | はい | アバター / 立ち絵 sprite |
| **Intro** | いいえ | キャラ紹介テキスト（キャラ選択画面用） |
| **Order** | はい | キャラ選択画面での並び順インデックス |
| **Deck** | はい | 初期デッキ（`RCG_DeckData`） |
| **JoinDeck** | いいえ | 中盤加入時に持参するデッキ（Deck と異なる進度） |
| **UnitSkillDatas** | いいえ | 初期で習得済みのユニットスキル |
| **SkillTags** | はい | キャラが持つ専門性（戦士 / 法師 / 牧師…）、カード専門性チェックに使用 |
| **Unlock** | いいえ | 解放条件（`RCG_UnlockEntry`）。`UnlockType.Tutorial` はチュートリアル解放；ショップ解放は GameRecord で購入記録も別途必要 |

## 動作説明

### キャラ選択分類
プログラム提供の4つの static エントリで異なるキャラリスト取得（すべて `m_Order` で並び順）：
*   `GetAllTutorialCharacters()` — チュートリアル解放のキャラ
*   `GetAllUnlockedCharacters()` — 解放済み
*   `GetAllLockedCharacters()` — 未解放
*   `GetAllJoinCharacters()` — チュートリアル + 解放済み（パーティ加入可能）

### 解放判定 (`Unlocked`)
*   `UnlockEntry.Unlocked == true`：解放条件達成 → ショップ系は更に `RCG_GameRecord.UnlockedCharacters` の ID 含有確認。
*   `UnlockType.None`：常に解放。
*   `UnlockType.Tutorial`：GameRecord 内にチュートリアル完了登録が必要。

### エディタ画面
*   **OnGUI** は左側に完全な Data 編集欄 + 右側にプレビューを表示。
*   **TestDropSkills** ボタン（内蔵テストツール）はドロップ / 抽選組合せをプレビュー可。

## 注意事項

*   **`Order` はキャラ選択画面の並び順に影響**：未設定だと先頭に集中。
*   **`JoinDeck` と `Deck` は別々**：開始時選択キャラは `Deck`；ストーリー途中の勧誘キャラは `JoinDeck`（通常はより簡素、デッキバランス維持）。
*   **解放条件の二段階判定**：`UnlockEntry` + ショップ購入記録；解放ロジック改修時は両方確認。
*   **`m_RuntimeData`** は実行時のみ変動するもの（装備、現スキル）、エディタで触らない。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CharacterData.cs`
*   **継承**：`RCG_Asset<RCG_CharacterData>`
*   **実装**：`UCLI_ShortName` / `RCGI_Unloackable`
*   **AssetGroup**：`EditCharacter`

### A.2 フィールドマッピング（外層）

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Data` | Data | `UnitData`（入れ子） | 編集時データ |
| `m_RuntimeData` | RuntimeData | `UnitRuntimData` | 実行時データ（装備など） |

`UnitData` に `m_LocalizeName` / `m_Avatar` / `m_Intro` / `m_MaxHp` / `m_Deck` / `m_JoinDeck` / `m_UnitSkillDatas` / `m_SkillTags` / `m_Unlock` / `m_Order` 等。

### A.3 主要メソッド

*   **`Unlocked` (property)** — 二段階解放判定（UnlockEntry → GameRecord）。
*   **`GetAllTutorialCharacters / GetAllUnlockedCharacters / GetAllLockedCharacters / GetAllJoinCharacters`** (static) — キャラ選択画面用。
*   **`Preview` / `OnGUI`** — エディタ描画。
*   **`Data.TestDropSkills(...)`** — 内蔵ドロップテストツール。
*   **`Init(string ID)` / `Init(CharacterID)`** — 多型構築入口。

### A.4 他システムとの連携

*   **`RCG_DeckData`** — `m_Deck` / `m_JoinDeck` 参照デッキ。
*   **`RCG_UnitSkillData`** — `m_UnitSkillDatas` 参照初期スキル。
*   **`RCG_SkillTagGenData`** — 専門性システム。
*   **`RCG_GameRecord.UnlockedCharacters`** — ショップ / チュートリアル解放記録。
*   **`RCG_UnlockEntry`** — 解放条件。
*   **`UnitRuntimData`** — 実行時キャラ状態（HP、装備）。

### A.5 既知の問題

*   `Unlocked` の `UnlockType.Tutorial` パスに `// QWQ23!!` コメントあり、旧版ロジック変更を示す。
*   `m_ActivePowers` は廃止コメント、当初「主動能力」フィールドだったが現在は UnitSkill システムに置換。
