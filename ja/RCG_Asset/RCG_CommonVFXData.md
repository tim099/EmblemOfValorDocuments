---
title: 汎用エフェクト (RCG_CommonVFXData)
description: 汎用 VFX 設定：エフェクトリソース、付随 SE、演出時間、付着位置（DisplayPos）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 汎用エフェクト

> クラス名：`RCG_CommonVFXData`

## 用途

**戦場上の各種汎用エフェクトテンプレート**。例：「状態層数増加」「チャージオーラ」「過負荷」「選択リング」「死亡」等、攻撃エフェクトに属さない視覚効果。各 `RCG_CommonVFXData` に VFX リソース、付随 SE、演出時間、ユニットへの貼付位置を含む。

`RCG_Asset<RCG_CommonVFXData>` を継承。

## エディタ上の見た目

```
RCG_CommonVFXData: <ID>
    VFX             ← VFX リソース（RCG_VFXResData）
    SE              ← 付随 SE（空可）
    VFXTime         ← 演出時間（秒）
    DisplayPos      ← エフェクト付着位置（EDisplayPos enum）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **VFX** | はい | VFX リソース |
| **SE** | いいえ | 付随 SE（VFX 構築時に自動再生） |
| **VFXTime** | はい | 演出時間（秒）、デフォルト 0.4f |
| **DisplayPos** | はい | 付着位置（`EDisplayPos.DisplayPos` がデフォルト） |

## 動作説明

### `CreateVFX(token)`
1. `m_VFX.IsEmpty` → return null。
2. `m_VFX.CreateVFX()` 経由で構築。
3. `vfx.CommonVFXData = this` 設定（runtime 逆引き用）。
4. `m_SE.GetData().PlaySE()` で同期再生。

### `PlayVFX(data, token)`
VFX 構築 → 位置設定（`SetPosition(iData.User)`）→ `m_VFXTime` 秒待機。

### デフォルト ID 定数
`RCG_CommonVFXGenData` が複数の内蔵 ID 提供：
*   `VFX_StatusLayer` — 状態層数増加
*   `VFX_ChargeEffect` / `VFX_OverloadEffect` — チャージ / 過負荷
*   `VFX_SelectionRing` — 選択リング
*   `CommonVFX_Dead` — 死亡

static インスタンス `s_ChargeEffect / s_OverloadEffect / s_SelectionRing / s_Dead` も提供、ID 文字列を手書きする必要なし。

## 注意事項

*   **`m_SE` は `CreateVFX` 時に自動再生**：外側で更に `PlaySE()` を呼ぶと二重再生になる。
*   **`m_VFXTime > 0` で待機**：0 設定は「fire and forget」（構築後すぐ返却）。
*   **`m_DisplayPos` enum**：VFX をユニットのどの階層に掛けるか（前景 / 背景 / 中央等）；具体的 enum 値はプログラム定義参照。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CommonVFXData.cs`
*   **継承**：`RCG_Asset<RCG_CommonVFXData>`
*   **AssetGroup**：`EditVFX`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_VFX` | VFX | `RCG_VFXResData` | |
| `m_SE` | SE | `RCG_SEGenData` | |
| `m_VFXTime` | VFXTime | `float` | デフォルト 0.4 |
| `m_DisplayPos` | DisplayPos | `EDisplayPos` enum | |

### A.3 主要メソッド

*   **`CreateVFX(token)`** — 構築 + CommonVFXData 設定 + SE 再生。
*   **`PlayVFX(data, token)`** — 構築 + 位置設定 + VFXTime 待機。

### A.4 他システムとの連携

*   **`RCG_VFXResData`** — VFX リソース。
*   **`RCG_SEGenData / RCG_SEData`** — SE システム。
*   **`RCG_VFX`** — runtime エフェクトインスタンス。
*   **`RCG_CommonVFXGenData`** — Asset Entry；複数の内蔵 ID と static インスタンス含む。
*   **`EDisplayPos` (enum)** — 表示位置定義。

### A.5 既知の問題

*   旧版 `DeserializeFromJson` の LoadType=Resource に対する LogError はコメントアウト済。
