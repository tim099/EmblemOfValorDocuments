---
title: オブジェクト池データ (RCG_PoolingData)
description: GameObject / VFX のプリロードとプーリング設定：タイプ、数量、テンプレート保持
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# オブジェクト池データ

> クラス名：`RCG_PoolingData`

## 用途

**オブジェクト池の設定**。どの prefab / VFX をプール化するか、事前にいくつ構築するか（prewarm）、テンプレートを保持するかを指定。runtime のオブジェクト構築コストを削減（戦闘中に頻出するダメージ数値、VFX パーティクル等）。

`RCG_Asset<RCG_PoolingData>` を継承。

## エディタ上の見た目

```
RCG_PoolingData: <ID>
    ResourceType  ▾ PrefabRes / VFXRes
    Prefab    (ResourceType=PrefabRes)
    VFX       (ResourceType=VFXRes)
    PrewarmCount    ← 事前構築数量
    PreserveTemplate ← テンプレート保持するか（true で安全）
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **ResourceType** | はい | `PrefabRes`（一般 prefab）/ `VFXRes`（特効リソース） |
| **Prefab** | PrefabRes 時 | 対応する prefab データ |
| **VFX** | VFXRes 時 | 対応する VFX データ（デフォルト Addressable パス `AttackVFXs/VFX_ArcaneMissileEffect`） |
| **PrewarmCount** | はい | 突入時に事前構築する数量 |
| **PreserveTemplate** | — | テンプレート貸出されるか。`true` = テンプレート貸出されない（destroy または改修されてから再使用してエラーを防ぐ） |

## 動作説明

### `LoadAsync<T>` / `CreateAsync<T>`
`ResourceType` に応じて `m_Prefab` または `m_VFX` の対応 method に分岐。

### Pooling Service 連携
`RCG_PoolingGenData`（外部参照ラッパー）→ `RCG_ObjectPoolService.Ins` 経由でインスタンス取得：
*   `GetObjectTemplate<T>` — テンプレート取得（読込のみ、貸出しない）
*   `CreateObject(parent)` — 池からインスタンス借用
*   `Delete(obj)` — 池に返却

### デフォルト ID
`RCG_PoolingGenData.DefaultID = "ItemDisplay"`（アイテム表示用）；`ItemDisplayerSmallID = "ItemDisplayerSmall"`。

## 注意事項

*   **`PreserveTemplate = false`** はテンプレート自体も貸出可能を意味 — テンプレートが改修 / destroy されると次回取得でエラー。**デフォルト true で安全**。
*   **`PrewarmCount` はテンプレート含まない**：実初期オブジェクト数 = PrewarmCount + 1（テンプレート）。
*   **VFXRes のデフォルト値**は `VFX_ArcaneMissileEffect` を指す：新池構築時は自分の VFX に変更を忘れずに。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_PoolingData.cs`
*   **継承**：`RCG_Asset<RCG_PoolingData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_ResourceType` | ResourceType | `ResourceType` enum | `PrefabRes` / `VFXRes` |
| `m_Prefab` | Prefab | `RCG_PrefabResData` | `Conditional(PrefabRes)` |
| `m_VFX` | VFX | `RCG_VFXResData` | `Conditional(VFXRes)`；デフォルト `VFX_ArcaneMissileEffect` |
| `m_PrewarmCount` | PrewarmCount | `int` | デフォルト 1 |
| `m_PreserveTemplate` | PreserveTemplate | `bool` | デフォルト true |

### A.3 主要メソッド

*   **`LoadAsync<T>(token)`** — リソース読込。
*   **`CreateAsync<T>(token, parent)`** — インスタンス構築。
*   **`RCG_PoolingGenData.GetObjectTemplate<T>` / `CreateObject<T> / CreateObject / Delete`** — 外部実使用入口。

### A.4 他システムとの連携

*   **`RCG_ObjectPoolService`** — runtime 池管理サービス。
*   **`RCG_PrefabResData / RCG_VFXResData`** — リソース読込ラッパー。
*   **`PathConst.AttackVFXs`** — VFX デフォルトパス定数。
