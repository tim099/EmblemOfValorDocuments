---
title: ゲームタグ
description: ゲームレベルのタグ効果（実績・プレイヤーイベントなど）をトリガー；薄いラッパー層
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# ゲームタグ

> クラス名：`RCG_GameTagBattleSetting`

## 用途
**バトル中にゲームレベルのタグイベントをトリガー** — `RCG_GameTagSetting` の薄いラッパー層で、その挙動をバトル動作キューに転送。よくある用途：
*   実績条件をトリガー（例：「最初のボスを倒す」）
*   プレイヤーバトル行動の記録をマーク

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **GameTagSetting** | はい | ネストした `RCG_GameTagSetting` — 実際のゲームタグ設定。 |

## 挙動
*   発動時に `m_GameTagSetting.Trigger(iData)` を呼ぶ。
*   説明・カード情報はすべて `m_GameTagSetting` に委譲。

## 注意点
*   **この設定はラッパーに過ぎない**：実際のロジックは `RCG_GameTagSetting` にある — フィールド設定はそのクラス説明を参照。
*   **戦闘数値に影響なし**：純粋にイベント報告 / 記録系効果。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_GameTagBattleSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_GameTagBattleSetting` 未登録（エディタ表示は stripped name `GameTagBattle`）

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_GameTagSetting` | `GameTagSetting` | `RCG_GameTagSetting` | — | ネスト；ラッパー唯一のフィールド |

### A.3 主なメソッド
*   **`AddAction`**：`m_GameTagSetting.Trigger(iData)`。
*   **`Info / GetDescriptionParams / GetDescriptionFormat`**：すべて `m_GameTagSetting` に委譲。

### A.4 他システムとの連携
*   **`RCG_GameTagSetting`**：実際のロジック保持先；ラッパー層は Battle Setting インターフェース対応のみ。
