---
title: カウンターをリセット
description: トリガーソース（装備 / ユニットスキル）の全カウンターを一括リセット
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# カウンターをリセット

> クラス名：`RCG_CounterClearSetting`

## 用途
**トリガーソースの全カウンターを一括クリア**。「カウンター効果」より強引：単一でなく**全部**を 0 化。よくある用途：
*   「装備能力発動後に全累積層をリセット」
*   「特定条件で装備チャージを強制クリア」

> [!IMPORTANT]
> 同様に**装備またはユニットスキルのトリガー効果内でのみ使用可能**（`EffectTriggerSource` が `RCGI_EffectCounter` であること）。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **CounterEffectType** | はい | カウンターの所属：`UnitSkill` または `Equipment`。**現在は説明表示用のみ**で、実際のクリアは EffectTriggerSource の全カウンターに対し行われ、タイプは選別されない。 |

## 挙動
*   `counter.ClearAllCounters()` を呼ぶ — そのトリガーソースの**全**カウンターをクリア。
*   説明：「**{CounterEffectType} カウンターをクリア**」（i18n key `CounterClearDes`）。

## 注意点
*   **CounterId フィールドなし**：クリアは「全部」式で、特定カウンターを選別不可。単一カウンターをクリアしたい場合は「**カウンター効果**」+ `Set 0` を使ってください。
*   **EffectTriggerSource 不一致のエラー**：「カウンター効果」と同様、console にエラーは出るが crash せず。
*   **常に展開**：`[AlwaysExpendOnGUI]`。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterClearSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **`[System.Serializable] + [AlwaysExpendOnGUI]`** 標記
*   **i18n クラス名 key**：`RCG_CounterClearSetting` → 「カウンターをリセット」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_CounterEffectType` | `CounterEffectType` | enum | — | `RCG_CounterAlterSetting` と共有 |

### A.3 主なメソッド
*   **`AddAction`** (async)：`iData.EffectTriggerSource` を取得、`RCGI_EffectCounter` への is-check 後 `counter.ClearAllCounters()` を呼ぶ。
*   **`GetDescriptionFormat / GetDescriptionShort`**：いずれも i18n key `CounterClearDes` を使い `m_CounterEffectType.GetLocalizeName()` を含む。

### A.4 他システムとの連携
*   **`RCGI_EffectCounter.ClearAllCounters`**：全カウンタークリアエントリ。
