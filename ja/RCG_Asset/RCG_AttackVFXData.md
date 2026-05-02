---
title: 攻撃エフェクト (RCG_AttackVFXData)
description: カード攻撃時のエフェクト設定：発射 VFX、命中 VFX、敵接近移動；preload メカニクス含む
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 攻撃エフェクト

> クラス名：`RCG_AttackVFXData`

## 用途

**カード攻撃時のエフェクトテンプレート**。1枚の攻撃カードが指定可能：
*   **発射 VFX**（`m_VFX`）：攻撃の瞬間に使用者位置で再生
*   **命中 VFX**（`m_HitVFX`）：ダメージ集計時に敵位置で再生
*   **敵接近移動の有無** + アニメーション時間配置（接近時の遅延、移動時間、停留、後退）

`RCG_Asset<RCG_AttackVFXData>` を継承。

## エディタ上の見た目

```
RCG_AttackVFXData: <ID>
    AttackVFXSetting (m_AttackVFXSetting)
        VFX                       ← 発射エフェクト
        HitVFX                    ← 命中エフェクト
        MoveTowardEnemy           ← 敵接近移動するか
        MoveTowardEnemySetting    (true 時) ← アニメーション時間配置
    Note                          ← メモ
```

## 主要フィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **VFX** | はい | 発射 VFX；デフォルト Addressable `AttackVFXs/VFX_AttackEffect` |
| **HitVFX** | いいえ | 命中 VFX |
| **MoveTowardEnemy** | — | 攻撃時に敵に接近移動するか |
| **MoveTowardEnemySetting** | MoveTowardEnemy=true 時 | アニメーション時間：`AttackAnimDelay` / `MoveTime` (0.5) / `WaitTime` (0.8) / `MoveBackTime` (0.25) / `OffsetX` |
| **Note** | いいえ | メモ（編集時自用、runtime 表示なし） |

## 動作説明

### Preload (`PreloadData`)
カードデータ読込時に呼出される `AttackVFXSetting.PreloadData(token)`：`m_VFX` と `m_HitVFX` をメモリに事前読込、戦闘中の初回再生時のカクつき回避。

### 敵接近移動
`m_MoveTowardEnemy = true` 時：
1. `m_AttackAnimDelay` 待機。
2. 敵に移動（`m_MoveTime`）。
3. 敵位置に停留（`m_WaitTime`）— 期間中に VFX 再生、ダメージ集計。
4. 元位置に後退（`m_MoveBackTime`）。
5. `m_OffsetX` が停留位置の水平オフセットを制御（敵味方で逆 — プレイヤーが敵を攻撃時に +X、敵は -X）。

## 注意事項

*   **デフォルト ID `AttackEffectPlain`**（`RCG_AttackVFXGenData.DefaultID`）：内蔵「平凡攻撃」エフェクト。
*   **`HitVFX` パス定数**：`PathConst.HitVFXs`、デフォルト空（多くの攻撃で個別命中エフェクト不要）。
*   **`Note` フィールドは純粋メモ**：実作用なし、デザイナー自分用。
*   **Preload 後でも戦闘初発でカクつく可能性**：Resource パス（Addressable でなく）使用時に preload 効果が限定的。

---

## 付録：プログラマ参考 (Programmer Reference)

### A.1 クラス情報
*   **ファイル**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AttackVFXData.cs`
*   **継承**：`RCG_Asset<RCG_AttackVFXData>`
*   **AssetGroup**：`EditVFX`

### A.2 フィールドマッピング

| コードフィールド | エディタ表示 | 型 | 備考 |
|---|---|---|---|
| `m_AttackVFXSetting` | AttackVFXSetting | `AttackVFXSetting`（入れ子） | 主データ |
| `m_Note` | Note | `string` | |

`AttackVFXSetting` に `m_VFX` / `m_HitVFX` / `m_MoveTowardEnemy` / `m_MoveTowardEnemySetting` を含む。
`MoveTowardEnemySetting` に `m_AttackAnimDelay` / `m_MoveTime` / `m_WaitTime` / `m_MoveBackTime` / `m_OffsetX` を含む。

### A.3 主要メソッド

*   **`AttackVFXSetting.PreloadData(token)`** — m_VFX + m_HitVFX を事前読込。

### A.4 他システムとの連携

*   **`RCG_VFXResData`** — VFX リソースラッパー。
*   **`PathConst.AttackVFXs / HitVFXs`** — Addressable パス定数。
*   **`RCG_AttackVFXGenData`** — Asset Entry；デフォルト `AttackEffectPlain`。
*   **戦闘アニメーションシステム** — runtime に `MoveTowardEnemySetting` の時間パラメータ適用。

### A.5 既知の問題

*   旧 ID 移行ロジック（`m_ID == "AttackEffect"` → `DefaultID`）はコメントアウト済。
