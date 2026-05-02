---
title: 特殊効果設定
description: 純粋な視覚特効を再生（ゲーム挙動への影響なし）；VFXGenData / Timeline 2 モード対応
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 特殊効果設定

> クラス名：`RCG_VFXSetting`

## 用途
**純粋な視覚特効を再生**。ゲーム挙動への影響なし、純粋に演出。よくある用途：
*   攻撃前の閃光 / 蓄力光輪
*   特殊スキルの場面特効
*   背景の時系列演出（Timeline モード）

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **VFXType** | はい | 特効タイプ：<br>• **VFXGenData** — `RCG_CommonVFXGenData` で単一特効生成（デフォルト）<br>• **Timeline** — 時系列で複数の効果音 / 特効を再生 |
| **VFXGenData** | VFXType=VFXGenData 必須 | 特効テンプレート。 |
| **Settings** | VFXType=Timeline 必須 | 時系列設定リスト（各項目は `VFXSetting`、時間と内容を指定可能）。 |

## 挙動
*   **VFXGenData モード**：直接 `m_VFXGenData.GetData().PlayVFX(iData, token)`。
*   **Timeline モード**：各 `VFXSetting` を並列で `Play(token)`、すべて `await` 完了後に終了。
*   **フュージョンに参加しない**：`GetFusionBaseSetting()` は null を返す。

## 注意点
*   **VFX が空**：`VFXGenData = null` のとき crash しないが何の効果も再生しない。
*   **Timeline は並列再生で順次ではない**：すべての設定が**並列**で再生され、**順次ではない**；厳密な順序が欲しいなら「組み合わせ効果」+ 複数の VFXSetting を使ってください。
*   **戦闘テンポ**：本設定は特効が再生完了するまで await し後続を待たせる — 長時間の特効は戦闘テンポを遅くします。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_VFXSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_VFXSetting` → 「特殊効果設定」
*   **同ファイルトップレベル enum**：`VFXType` (`VFXGenData`, `Timeline`)

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_VFXType` | `VFXType` | `VFXType` (内部 enum) | — | デフォルト `VFXGenData` |
| `m_VFXGenData` | `VFXGenData` | `RCG_CommonVFXGenData` | — | `[Conditional(... VFXGenData)]` |
| `m_Settings` | `Settings` | `List<VFXSetting>` | — | `[Conditional(... Timeline)]` |

### A.3 主なメソッド
*   **`AddAction`** (async)：`m_VFXType` で分岐：
    *   `VFXGenData` → `await m_VFXGenData.GetData().PlayVFX(iData, iToken)`。
    *   `Timeline` → 各 `setting.Play(iToken)` を `UniTask` として収集、`UniTask.WhenAll(tasks)` で並列待機。
*   **`GetFusionCandidateSettings`** → 空リスト；**`GetFusionBaseSetting`** → null。

### A.4 他システムとの連携
*   **`RCG_CommonVFXGenData / VFXSetting`**：特効テンプレートと時系列設定コンテナ。
*   **`PlayVFX(iData, token)`**：実際の再生エントリ。

### A.5 既知の課題
*   旧版 `m_VFX` (`RCG_VFXResData`) と `m_VFXTime` は新アーキテクチャ（VFXGenData）に置換されたが、デシリアライズ移行ロジックはコメントとして保存。
