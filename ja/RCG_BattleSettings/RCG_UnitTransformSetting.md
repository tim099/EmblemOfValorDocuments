---
title: 変身
description: ターゲットユニットを別タイプに変身；デフォルト変身・転生・再生の 3 モード対応
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 変身

> クラス名：`RCG_UnitTransformSetting`

## 用途
**ユニットを別タイプのユニットに変身**。よくある用途：
*   ボスのフォーム切替（HP 保持で外観変化）
*   「転生」「再生」式の復活メカニズム
*   特殊変身カード

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **対象** (`Target`) | はい | 変身対象セレクタ。 |
| **UnitData** | はい | 変身先のユニット（`RCG_UnitGenData`）。 |
| **TransformSetting** | はい | 変身詳細設定（ネスト）。 |

### TransformSetting 内のフィールド
| フィールド | デフォルト | 説明 |
|---|---|---|
| **TransformType** | `Default` | 変身タイプ：<br>• **Default** — 一般変身。**HP と状態を保持**、行動モードとモデルのみ変更（**ボスのフォーム切替**用）。<br>• **Reincarnation** — 転生。**状態のみ保持**；HP、最大 HP と他の数値は変身先ユニットに；初期行動をトリガー。<br>• **Rebirth** — 再生。状態を保持しない；HP、最大 HP と他の数値は変身先ユニットに；初期行動をトリガー。 |
| **ClearAllStatus** | `false` | 全状態をクリアするか（TransformType の「状態保持」ロジックと独立）。 |

## 挙動
*   説明形式：`UnitTransformDes_{TransformType}`、対象とユニット名で組立。
*   実行時に TransformType でどの属性を保持するかを決定し、視覚切替（Spine モデル / AI / 行動の変更）を行う。

## 注意点
*   **Default vs Reincarnation の違い**：両方ともモデル変更だが、Default は当前数値全保持（ボスフォーム切替に適す）；Reincarnation は HP リセット + 状態保持（「転生」に適す）。
*   **初期行動のトリガー**：Reincarnation / Rebirth は変身先ユニットの `OnBattleStart` を走らせる（入場状態 / 入場攻撃などを含む）。
*   **ClearAllStatus と TransformType の衝突**：Default + ClearAllStatus は「HP 保持で状態クリア」と等価 — 珍しい組み合わせ、慎重に使用を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_UnitTransformSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_UnitTransformSetting` → 「変身」
*   **同ファイルトップレベルクラス**：`TransformSetting`（`TransformType` enum と `m_ClearAllStatus` を含む）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_Target` | 対象 | `RCG_SelectTargetData` | `Target` | |
| `m_UnitData` | `UnitData` (=「戦闘キャラクター」) | `RCG_UnitGenData` | `UnitData` | |
| `m_TransformSetting` | `TransformSetting` | `TransformSetting` | — | `TransformType` と `ClearAllStatus` を含む |

### A.3 主なメソッド
*   **`AddAction`**（ファイル 80 行外）：`TransformType` で実際の変身ロジックを切替。
*   **`GetDescriptionFormat`** → `UnitTransformDes_{TransformType}` i18n key、target / unit を流入。
*   **`Info`** → `new CardInfoData(m_UnitData.GetData())`。

### A.4 他システムとの連携
*   **`RCG_Unit` の変身エントリ**（具体メソッド名は後続実装を参照）。
*   **`RCG_UnitGenData`**：変身先ユニットテンプレート。
*   **`OnBattleStart`**：Reincarnation / Rebirth の初期行動トリガーポイント。
