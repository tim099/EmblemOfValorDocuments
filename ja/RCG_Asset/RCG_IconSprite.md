---
title: TMP アイコン (RCG_IconSprite)
description: TextMeshPro 用の inline アイコンリソース；UI 説明内の小アイコン（HP / 護甲 / カード等）のソース
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# TMP アイコン

> クラス名：`RCG_IconSprite`

## 用途

**TextMeshPro 用の inline アイコンリソース**。カード説明、状態説明、戦闘ログによく出る「+5 HP」「-3 護甲」等のテキスト、HP / 護甲等の小アイコンは TMP の `<sprite name=...>` タグで挿入；本 Asset がそれらアイコンのソース。システムが全 `RCG_IconSprite` を1つの `TMP_SpriteAsset` にパッケージしてテキスト描画に提供。

`RCG_Asset<RCG_IconSprite>` を継承。

## エディタ上の見た目

```
RCG_IconSprite: <ID>
    Icon       ← アイコン sprite
    Scale      ← スケール（TMP 内表示用）
    BearingX   ← 水平ベースラインオフセット
    BearingY   ← 垂直ベースラインオフセット（デフォルト 0.85）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **Icon** | はい | アイコン sprite（`RCG_SpriteData`）、デフォルト `Status_BloodDrain` |
| **Scale** | はい | TMP 内のスケール（デフォルト 1） |
| **BearingX** | — | 水平ベースラインオフセット |
| **BearingY** | — | 垂直ベースラインオフセット（デフォルト 0.85、TMP テキストベースラインに対応） |

## 動作説明

### TMPKey
各 IconSprite が自動的に TMP タグ生成：`<sprite name={ID}>`。テキストでこれを使うと inline で対応アイコン表示可。

### TMP_SpriteAsset への自動パッケージ
`InitSpriteAsset(token)` が起動時に全 `RCG_IconSprite` をスキャンし単一 `TMP_SpriteAsset` にパッケージ；モジュール再読込時 (`UCL_ModuleService.OnLoadedModule`) に自動 refresh。

### `Save()` で RefreshSpriteAsset 発動
エディタ内で本 Asset を保存すると自動的に `RefreshSpriteAsset()` で再パッケージし変更を即時確認可。

### 内蔵 static 参照
*   `EffectIcon_Target` / `EffectIcon_CountDown` / `EffectIcon_Armor` / `EffectIcon_Health` / `EffectIcon_Card` / `EffectIcon_Die` / `EffectIcon_Energy` / `EffectIcon_Magnifier`
これらはプログラム内でよく使うアイコン、static property で直接取得。

### Editor ツール
*   **`Output sprite sheet`**（エディタ専用）：sprite sheet 画像（合成画像）一式を出力。
*   **`Refresh SpriteAsset`**：手動で再パッケージ。

## 注意事項

*   **パッケージは async**：`InitSpriteAsset` が `s_IsRefreshingSpriteAsset` flag で互斥、並行パッケージ回避；refresh 中の再呼出は前回終了まで待機。
*   **`BearingY = 0.85` デフォルト**：TMP フォントベースラインに整列、調整するとアイコンが上下にずれる可能性。
*   **デフォルト ID `AbsorbShield`**（`RCG_IconSpriteGenData.DefaultID`）。
*   **`m_Disable` はコメントアウト済**：以前 disabled アイコンスキップ計画あり；現状は全アイコンがパッケージされる。
*   **`LocalizeName` の動作**：UI モードで TMPKey 返却（アイコン）、非 UI モードで i18n 翻訳返却（純テキスト）；カード説明システムは `RCG_BattleSetting.IsShowOnUI` で切替。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_IconSprite.cs`
*   **継承**：`RCG_Asset<RCG_IconSprite>`
*   **AssetGroup**：`EditTags`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_Icon` | Icon | `RCG_SpriteData` | デフォルト `Status_BloodDrain` |
| `m_Scale` | Scale | `float` | デフォルト 1 |
| `m_BearingX` | BearingX | `float` | デフォルト 0 |
| `m_BearingY` | BearingY | `float` | デフォルト 0.85 |

### A.3 主要メソッド（含 static）

*   **`InitSpriteAsset(token)` (static async)** — 起動時パッケージ；OnLoadedModule 購読含む。
*   **`GenerateSpriteAsset(token)` (static)** — 全 icons 読込し `TMP_SpriteAsset` 構築。
*   **`RefreshSpriteAsset / RefreshSpriteAssetAsync`** — 再パッケージ（重複呼出は互斥）。
*   **`LoadIconSprites(token)` (private static)** — 全 textures + names + icons 読込の helper。
*   **`Save()` (override)** — RefreshSpriteAsset 発動。
*   **`TMPKey` (property)** — `<sprite name={ID}>`。
*   複数の static `EffectIcon_*` ショートカット参照。

### A.4 他システムとの連携

*   **`TMP_SpriteAsset`** — TextMeshPro アイコンリソース。
*   **`RCG_TMPTools`** — パッケージとツール（`CreateSpriteAsset / RefreshSpriteAsset / CreateIconSpriteSheetEditor`）。
*   **`RCG_SpriteData`** — ソース sprite。
*   **`UCL_ModuleService.OnLoadedModule`** — モジュール再読込時に自動 refresh。
*   **`RCG_BattleSetting.IsShowOnUI`** — テキスト説明モード切替。
*   **`RCG_IconSpriteGenData`** — Asset Entry；デフォルト `AbsorbShield`。

### A.5 既知の問題

*   `m_Disable` / `CheckSpriteAsset / GenerateSpriteAsset (sync ver)` 等の複数箇所コメントアウト、旧版同期パッケージフローを async に変更したことを示す。
*   `s_IsRefreshingSpriteAsset` はグローバル static flag、極端な状況で task が例外をスローし復位しないと将来の refresh が hang する可能性。
