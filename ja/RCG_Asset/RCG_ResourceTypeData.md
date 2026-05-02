---
title: リソースタイプ (RCG_ResourceTypeData)
description: ゲーム内各種リソース（金貨 / 補給 / 魂 / 祝福）の定義：名前、説明、アイコン
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# リソースタイプ

> クラス名：`RCG_ResourceTypeData`

## 用途

**ゲーム内蓄積可能リソースタイプの定義**。Gold（金貨）、Supply（補給 / 暗霧減少アイテム）、Spirit（魂）、Blessing（祝福）等は異なる ID の `RCG_ResourceTypeData`。本データはそれらの表示名、説明、アイコンを定義。

`RCG_Asset<RCG_ResourceTypeData>` を継承。

## エディタ上の見た目

```
RCG_ResourceTypeData: <ID>
    Name(多言語)
    Description(多言語)
    IconSprite      ← 対応する RCG_IconSprite 参照
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Name** | はい | リソース名（多言語） |
| **Description** | いいえ | リソース説明（tooltip 用） |
| **IconSprite** | はい | アイコン参照（`RCG_IconSpriteGenData`） |

## 動作説明

### LocalizeName 切替
*   `RCG_BattleSetting.IsShowOnUI = false` → `m_Name.Name`（純テキスト名）返却。
*   `IsShowOnUI = true` → `m_IconSprite.TMPKey`（TextMeshPro 用 sprite tag）返却。

### デフォルト 4 種リソース（プログラム定数）
*   `Gold` (`ResourceType_Gold`)
*   `Supply` (`ResourceType_Supply`) — 「闇霧」関連用途にも対応
*   `Spirit` (`ResourceType_Spirit`)
*   `Blessing` (`ResourceType_Blessing`)

## 注意事項

*   **ID 命名規則**：`ResourceType_<EnumName>`（如 `ResourceType_Gold`）。
*   **`enum ResourceType`（同ファイル）** には3種のみ列挙（Gold / Supply / Spirit）；Blessing は文字列直接使用で enum に入っていない。
*   **`RCG_DifficultyData.m_PriceMult / m_SoulPriceMult`** 等の難度倍率との対応：価格倍率は Gold / Spirit のショップ価格に影響。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ResourceTypeData.cs`
*   **継承**：`RCG_Asset<RCG_ResourceTypeData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |

### A.3 主要 Property

*   **`LocalizeName`** — UI モードで TMPKey 返却；それ以外で Name 返却。
*   **`Description`** — `m_Description.Name`。
*   **`Icon`** — `m_IconSprite.GetData().IconSprite`。

### A.4 他システムとの連携

*   **`RCG_ResourceTypeGenData`** — Asset Entry；4つの static デフォルト含む（Gold / Supply / Spirit / Blessing）。
*   **`RCG_IconSpriteGenData`** — アイコンリソース参照。
*   **`RCG_DataService.Ins.GetResource(...)`** — runtime リソース量取得入口。
*   **`RCG_BattleSetting.IsShowOnUI`** — LocalizeName 表示モード制御。
