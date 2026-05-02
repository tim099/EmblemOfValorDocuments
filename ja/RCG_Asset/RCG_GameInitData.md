---
title: ゲーム初期データ (RCG_GameInitData)
description: ゲームグローバル singleton 設定：開始アイテム/装備/スキル、カードシステム設定、Pooling、エディタ設定、暗霧
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ゲーム初期データ

> クラス名：`RCG_GameInitData`

## 用途

**ゲームのグローバル唯一設定データ**（singleton Asset、ID 固定 `Default`）。含む内容：
*   開始アイテム / 装備 / 主動能力 / リソース
*   カードシステム設定（`RCG_CardGameSetting`）
*   Pooling 設定（オブジェクト池）
*   Prefab リソース設定
*   エディタ設定
*   暗霧設定一覧
*   装備タイプ対応のアイコン

`RCG_Asset<RCG_GameInitData>` を継承。

## エディタ上の見た目

```
RCG_GameInitData: Default
    GameVersion / DefaultLanguage
    CardGameSetting       ← カードシステムのグローバル設定
    PrefabResSetting      ← 共通 prefab パス
    PoolingGenDataSetting ← オブジェクト池設定
    DarkMistSettings      ← 暗霧設定一覧
    EditorSetting
    InitItems / InitActivePowers / InitEquipments / InitResources
    EquipmentTypeIcon     ← 装備タイプ → アイコン対応
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **GameVersion** | はい | ゲームバージョン（`RCG_GameVersionGenData`） |
| **DefaultLanguage** | はい | デフォルト言語 |
| **CardGameSetting** | はい | カードシステム設定（手札数、ドローロール等） |
| **PrefabResSetting** | はい | 共通 prefab パス設定 |
| **PoolingGenDataSetting** | はい | オブジェクト池設定（どの prefab を pool するか / 初期数） |
| **DarkMistSettings** | いいえ | 暗霧設定一覧（複数 `RCG_DarkMistSettingsData` 参照） |
| **EditorSetting** | いいえ | エディタ関連設定 |
| **InitItems / InitActivePowers / InitEquipments / InitResources** | いいえ | 新ゲーム時の初期物品 / 主動能力 / 装備 / リソース |
| **EquipmentTypeIcon** | いいえ | `EquipmentType → RCG_SpriteData` 対応、UI 表示用 |

## 動作説明

### `GameInit()`
新ゲーム時に呼出、`m_InitItems` / `m_InitEquipments` / `m_InitActivePowers` を全部プレイヤーに追加（個別 Asset が `AddItem` ロジックを自帯）。

### `GetEquipmentIcon(EquipmentType)`
`m_EquipmentTypeIcon` 辞書を引く；対応 key 不在時に LogError し、最初の entry にフォールバック。

### Static エントリ
*   `RCG_GameInitData.Ins` — ID `Default` の Asset 取得。
*   `CreateInstance()` — Asset から再度初期データ生成（`useDefaultIfMissing = false`）。

## 注意事項

*   **ID は `Default` 固定**：このクラスは Asset 1つのみ想定。他 ID 追加してもシステム自動読込されない（`Util.GetData` 手動呼出経由のみ）。
*   **`m_GameSetting` はコメントアウト済**：元は GameInitData が GameSettingData を参照する設計、現在は `Application.version` 自動関連に変更。
*   **`m_UnlockSetting` はコメントアウト済**：解放設定は `RCG_UnlockData` が処理する設計に変更。
*   **`m_InitEquipments` 標 TODO**：「キャラ設定データに移行予定、異なるキャラが異なる装備を持参」 — 将来 `RCG_CharacterData` に移される可能性。
*   **装備タイプアイコンが見つからない時の fallback**：最初の entry を使用、意図したものでない可能性；漏れ設定で誤ったアイコンが表示される。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameInitDatas/RCG_GameInitData.cs`
*   **継承**：`RCG_Asset<RCG_GameInitData>`
*   **AssetGroup**：`EditGameSetting`
*   **定数**：`DefaultID = "Default"`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `RCG_GameVersionGenData` | |
| `m_DefaultLanguage` | DefaultLanguage | `UCL_LanguageCodeEntry` | |
| `m_CardGameSetting` | CardGameSetting | `RCG_CardGameSetting` | |
| `m_PrefabResSetting` | PrefabResSetting | `RCG_PrefabResSetting` | |
| `m_PoolingGenDataSetting` | PoolingGenDataSetting | `RCG_PoolingGenDataSetting` | |
| `m_DarkMistSettings` | DarkMistSettings | `List<RCG_DarkMistSettingsGenData>` | |
| `m_EditorSetting` | EditorSetting | `RCG_EditorSetting` | |
| `m_InitItems` | InitItems | `List<RCG_ItemGenData>` | |
| `m_InitActivePowers` | InitActivePowers | `List<RCG_ActivePowerGenData>` | |
| `m_InitEquipments` | InitEquipments | `List<RCG_EquipmentGenData>` | |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_EquipmentTypeIcon` | EquipmentTypeIcon | `Dictionary<EquipmentType, RCG_SpriteData>` | |

### A.3 主要メソッド

*   **`Ins` / `CreateInstance` (static)** — 2種の取得入口（前者は cache 使用、後者は強制再読込）。
*   **`GameInit()`** — 新ゲーム時に初期物品 / 装備 / 能力を追加。
*   **`GetEquipmentIcon(type)`** — アイコン検索；不在時に最初の entry にフォールバック。
*   **`CardGameSetting / PoolingGenDataSetting / PrefabResSetting` (static properties)** — ショートカットアクセス。

### A.4 他システムとの連携

*   **`RCG_DataService`** — runtime プレイヤーデータ；InitItems 等はここに追加される。
*   **`RCG_DarkMistSettingsData`** — 暗霧設定一覧の要素。
*   **`RCG_CardGameSetting`** — カードシステム設定。
*   **`UCL_LanguageCodeEntry`** — 言語設定。

### A.5 既知の問題

*   `m_GameSetting` / `m_UnlockSetting` / `m_CardClassBackgroundImages` 等の複数箇所でコメントアウト、旧版設計変更を示す。
*   `m_InitEquipments` のキャラデータへ移行予定の TODO。
