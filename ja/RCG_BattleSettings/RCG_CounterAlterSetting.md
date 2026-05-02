---
title: カウンター効果
description: トリガーソース（装備 / ユニットスキル）の内部カウンター数値を修正
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カウンター効果

> クラス名：`RCG_CounterAlterSetting`

## 用途
**トリガーソースのカウンターを修正**。「カウンター」は装備またはユニットスキルの内部数値（例：「使用回数」「累積層数」）。よくある用途：
*   「装備：被弾 3 回ごとに発動」 — カウンターが累積とリセットを繰り返す
*   「装備：1 戦闘あたり X 回まで使用可」 — カウンターが残使用回数を追跡

> [!IMPORTANT]
> この設定は**装備またはユニットスキルのトリガー効果内でのみ使用可能** — `iData.EffectTriggerSource` が `RCGI_EffectCounter` の必要があります。一般カードトリガー時は**カウンター対象がない**。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CounterEffectType** | はい | カウンターの所属：`UnitSkill`（ユニットスキル）または `Equipment`（装備）。 |
| **CounterId** | はい | カウンターのインデックス（同装備に複数カウンターがあり、0、1、2…）。 |
| **CounterAlter** | はい | 変化量（変数可）。 |
| **CounterAlterType** | はい | 変化モード：`Add` / `Sub` / `Set`。 |

## 挙動
*   `iData.EffectTriggerSource` を読み、`RCGI_EffectCounter` であることを確認し、`counter.AlterCounter(CounterId, CounterAlterType, CounterAlter)` を呼ぶ。
*   説明形式：「**{CounterEffectType} カウンター {Id+1} {CounterAlterType} {CounterAlter}**」（i18n key `CounterAlterDes`）。

## 注意点
*   **EffectTriggerSource 不一致でエラー**：カード効果でこの設定をトリガーすると console にエラーが出るが crash せず。**上位が装備またはスキル効果であることを必ず確認**。
*   **常に展開**：この設定は `[AlwaysExpendOnGUI]` 属性、Inspector 上で**折りたたまれない**。
*   **CounterId が範囲外**：その装備のカウンター数を超えた ID を指定すると未定義動作；ID と装備設定が一致するか確認を。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterAlterSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n クラス名 key**：`RCG_CounterAlterSetting` → 「カウンター効果」
*   **同ファイル内 enum**：`CounterEffectType { UnitSkill, Equipment }` / `CounterAlterType { Add, Sub, Set }`

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CounterEffectType` | `CounterEffectType` | enum | — | |
| `m_CounterAlterType` | `CounterAlterType` | enum | — | |
| `m_CounterId` | `CounterId` | `int` | — | デフォルト 0 |
| `m_CounterAlter` | `CounterAlter` | `IntVariable` | — | |

### A.3 主なメソッド
*   **`AddAction`** (async)：`iData.EffectTriggerSource` を取得、`RCGI_EffectCounter` への is-check 後 `counter.AlterCounter(m_CounterId, m_CounterAlterType, m_CounterAlter.GetValue(iData))` を呼ぶ。
*   **`GetDescriptionShort`** → i18n key `RCG_CounterAlterSetting`（=「カウンター効果」）。
*   旧 `m_TriggerData.m_Equipment / m_UnitSkill` の分岐ロジックはコメントアウト。

### A.4 他システムとの連携
*   **`RCGI_EffectCounter`**：カウンターインターフェース、装備 / ユニットスキルが実装。
*   **`iData.EffectTriggerSource`**：トリガーソース；装備のトリガー効果時に対応インスタンスが入る。
