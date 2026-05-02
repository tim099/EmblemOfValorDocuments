---
title: 強化
description: 選択した手札に追加効果（OnPlay 時に発動）を付加して、カードの挙動を永続的に改造
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 強化

> クラス名：`RCG_EnhenceSetting`

## 用途
**選択した手札に追加効果を付加**：強化されたカードがプレイされると、本来の効果に加えて `EnhenceSetting` が**もう一度トリガーされる**。よくある用途：
*   「次の攻撃カードを強化：吸血効果を付加」
*   「全手札を強化：各 1 ドロー」

強化されたカードはこの効果を永久に持ち続けます（消滅または「**カード効果無効化**」で削られるまで）。

## 主なフィールド

| エディタ表示 | 必須 | 説明 |
|---|---|---|
| **SelectHandCardSetting** | はい | 強化対象を選ぶ設定。 |
| **EnhenceSetting** | はい | 付加する「**組み合わせ効果**」 — トリガータイミングは `OnPlay` 固定。 |

## 挙動
*   選択完了後、各カードに対し `aCard.Enhence(commonEffect)` を呼ぶ：`EnhenceSetting` を `RCG_CommonEffect`（`OnPlay` トリガー）でラップして付加。
*   説明形式：「**{選択} : {強化タイトル} {強化内容説明}**」（i18n key `EnhenceSettingDes`、`EnhenceTitle` 色を含む）。
*   付帯情報枠（`Infos`）：第 1 条目が「強化説明」概要、後続に強化効果自身の Infos。

### タグ
強化設定は自分が含むタグ（`m_EnhenceSetting.GetBattleTags`）を外側に露出しません — 設計選択で、タグが元カードに誤計入するのを回避するため。

## 注意点
*   **強化は永続的**：強化効果が一度付加されたら、カード消滅または弱体化まで持続。
*   **トリガータイミング `OnPlay` 固定**：現在は**変更不可**。他のタイミングが必要なら「条件判断」+「状態」での代替を検討。
*   **「カード効果無効化」との対応**：弱体化時には強化葉効果が優先消費；本設定の内容が真っ先にやられる。
*   **タグを外に出さない**：意図的設計 — 強化内含の「脆弱」「敏捷」などのタグがカード情報に出ることを期待しないでください。

---

## 付録：プログラマ向け (Programmer Reference)

### A.1 クラス情報
*   **パス**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_EnhenceSetting.cs`
*   **継承元**：`RCG_BattleSetting`
*   **i18n クラス名 key**：`RCG_EnhenceSetting` → 「強化」

### A.2 フィールド対照

| コード | エディタ表示 | 型 | Localize key | 備考 |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | `SelectHandCardSetting` | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_EnhenceSetting` | `EnhenceSetting` | `RCG_CombineSetting` | — | `[SerializeField] protected` |

### A.3 主なメソッド
*   **`AddAction`**：選択 → 各カードに対し `RCG_CommonEffect { m_CombineSetting = m_EnhenceSetting, m_EffectTriggerOn.m_TriggerOn = OnPlay }` を構築し、`aCard.Enhence(aEffect)` を呼ぶ。
*   **`GetBattleTags`** → 常に空リスト（**意図的 override**、`m_EnhenceSetting` のタグを露出しない）。
*   **`GetDescriptionFormat`**：「{選択 desc}\n{Title} : {Enhence desc + 内含 BattleTags}」。
*   **`GetEnhenceSettingDescription` (private)**：`m_EnhenceSetting.GetDescription` + その BattleTags 説明を取得。
*   **`Infos`**：`"RCG_EnhenceSetting" + "\n" + EnhenceSettingInfo(...)` を第 0 項目として挿入。

### A.4 他システムとの連携
*   **`RCG_CommonEffect`**：強化効果のラップコンテナ（`TriggerOn` を含む）。
*   **`RCG_CardBattleData.Enhence(commonEffect)`**：実際の強化付加エントリ。
*   **`RCG_Extensions.TagColors.EnhenceTitle`**：タイトル色。
*   **i18n keys**：`RCG_EnhenceSetting` / `EnhenceSettingDes` / `EnhenceSettingInfo`。

### A.5 既知の課題
*   旧版 `m_CostAlter` / `m_EffectTriggerTiming` フィールドはコメントアウト（強化系のリファクタ中）；現在はコスト変化やカスタムトリガータイミングが利用不可。
