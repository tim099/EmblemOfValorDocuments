---
title: 卡牌資料 (RCG_CardData) 說明
description: 遊戲中所有卡牌的資料模板：基本資訊、效果、強化分支、被動效果、解鎖條件等完整定義
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌資料

> 程式類別名稱：`RCG_CardData`

## 用途
**遊戲中每一張卡牌的資料模板**。從基本資訊（名稱、費用、稀有度、卡圖）到實際效果、強化分支、被動標籤、解鎖條件，全部定義在這個 Asset 上。在 Developer Page 的 `EditItems` 群組下，所有可玩 / 可商店販售 / 可掉落的卡牌都是這個類別的實例。

繼承自 `RCG_Asset<RCG_CardData>`，實作介面：`RCGI_Item`（可加入背包）/ `RCGI_CardData`（卡牌資料協定）/ `RCGI_Unloackable`（可解鎖）。

## 編輯器中的樣貌
編輯一張卡牌時，畫面分成三大區塊：
```
卡牌設定 (CardData)        ← 基本欄位：名稱 / 費用 / 類型 / 稀有度 / 標籤 / 解鎖等
效果 (N)                   ← m_Effects：實際的卡牌效果（OnPlay 等觸發）
被動效果 (N)               ← m_PassiveEffects：條件式 BattleTag（影響 Cost 計算等）
預覽                       ← 即時呈現卡牌外觀與描述
```

最右側「預覽」會即時顯示卡牌長相、Tooltip 標籤、卡圖、解鎖條件，並提供「**強化分支預覽**」與「**複製描述**」按鈕。

## 主要欄位（卡牌設定 / CardData）

### 基本資訊
| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **名稱(多國語言)** (`LocalizeName`) | 是 | 卡牌名稱（`RCG_LocalizeData`，支援多語系）。 |
| **費用** (`Cost`) | 是 | 打出此牌消耗的能量；整數。 |
| **卡牌類型** (`CardType`) | 是 | `Default`（普通卡）或 `Chant`（**詠唱卡**：需累積回合才能打出）。 |
| **詠唱回合數** (`ChantTurn`) | CardType=Chant 時 | 詠唱要等幾回合（變數可變）；條件顯示。 |
| **稀有度** (`Rarity`) | 是 | `Bronze` / `Silver` / `Gold` / `Legend` / `Cursed` 等（`RCG_RarityTagGenData`）。 |
| **目標範圍** (`TargetType`) | 是 | 17 種：`None`、`Friend`、`Allied(Front/Back)`、`Enemy(Front/Back)`、`All`、`AnyPos`、`AlliedNonLeader`、`AnySummoned` 等。 |
| **使用類型** (`UsedTypeTag`) | 是 | 例如「消耗」「可重複使用」「乙太」等使用後處理方式。 |
| **價格** (`Price`) | 是 | 商人販售價格；整數。預設 100。 |
| **卡圖** (`CardIcon`) | 是 | 卡牌圖案（`RCG_SpriteData`）。 |

### 標籤與專精
| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **專精** (`SkillTags`) | 否 | 角色需擁有的專精（戰士 / 法師 / 牧師…）；空 = 任何角色都能用。多項視為「**任一符合即可**」。 |
| **卡牌標籤** (`CardTags`) | 否 | 攻擊 / 技能 / 詠唱 / 消耗… 等分類用標籤。**重要**：判斷「是否為詠唱卡」請用 `HasTag()`，因為 `m_CardTags` 不會自動包含詠唱 tag。 |

### 卡牌效果
| 區塊 | 說明 |
|---|---|
| **效果** (`m_Effects`) | 主要的卡牌效果清單；每項是 `RCG_CommonEffect`，內含「觸發時機（OnPlay / OnDraw / OnDiscard / ...）+ 組合效果」。打出卡或滿足觸發條件時會跑這些。 |
| **被動效果** (`m_PassiveEffects`) | 條件式 `RCG_ConditionalPassiveBattleTag` 清單；用於**狀態查詢**（例如「手牌數 ≥ 5 時 -1 費」）— **不參與 OnTriggerEffect 流程**，純粹影響 Cost 等屬性計算。 |

### 強化系統
| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **禁用的強化分支** (`BannedEnhence`) | 否 | 不允許在此卡上掉落的強化分支 ID。 |
| **強化池** (`EnhencePool`) | 是 | 強化時隨機抽取的 `RCG_CardEnhenceDropPoolGenData`。 |
| **ApplyEnhenceOnAllTiming** | — | 是否把強化效果套用到此卡所有觸發時機（不只 OnPlay）。 |
| **強化等級** (`UpgradeLv`) | — | 顯示用：≥1 會在卡名後加 `+1` / `+2`；通常由系統自動設定，不建議手動改。 |
| **EnhenceLocalize** | 否 | 強化版的命名替代（例如 α、β、γ）；空時用 `+` 前綴。 |

> [!IMPORTANT]
> 目前**強化次數限制為 1**（`m_UpgradeLv < 1` 才允許再強化），且**詛咒卡 (Cursed) 無法強化**。`CanEnhence` 判斷會直接擋下。

### 描述
| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **卡牌描述方式** (`CardDescriptionType`) | 是 | `Auto`（自動由效果合成）或 `Manually`（手動撰寫）。 |
| **卡牌描述** (`CardDescription`) | Manually 時 | 手動撰寫的描述本體（多語系）；自動模式下隱藏。 |
| **描述範本** (`DescriptionTemplate`) | Manually 時 | 含 `{(OnPlay.0)}` 等佔位符的範本字串；按 `Generate Description Template` 按鈕可從當前效果自動生成。 |

> [!TIP]
> 自動模式下系統會把每個 effect 的描述、目標範圍、battle tags 自動串接。多數情況用自動就好；只有需要**精準控制句型** / **隱藏部分效果** / **加風味文字**時才切手動。

### 解鎖 / 雜項
| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **解鎖** (`Unlock`) | 否 | 解鎖條件（`RCG_UnlockEntry`）；未解鎖前不會出現在獎勵 / 商店中。 |
| **HideInCodex** | — | 在圖鑑中隱藏（測試用 / 廢卡）。 |
| **備註** (`Note`) | — | 純文字備註（無實際作用，給設計師自己看）。 |
| **作者** (`Author`) | — | 卡牌作者署名。 |

## 行為說明

### 卡牌顯示名稱
取自 `LocalizedName`：
*   普通卡：直接是 `m_LocalizeName.Name`
*   強化版（`UpgradeLv ≥ 1`）：
    *   `EnhenceLocalize` 非空 → `{Name}{α/β/γ}`
    *   `UpgradeLv == 1` → `{Name}+`
    *   `UpgradeLv ≥ 2` → `{Name}+{N}`
*   詠唱卡：在遊戲中以「**詠唱**」前綴 + 詠唱色顯示；Editor 內顯示 `詠唱[Name]`

### 詠唱卡（CardType = Chant）
*   需要先「詠唱」`m_ChantTurn` 個回合才能真正打出。
*   描述會在最上方加一行「**詠唱[N]**」（綠色詠唱色）。
*   `HasTag(TagChant)` 會回傳 true（即使 `CardTags` 沒明列）。

### 描述生成
*   **自動模式 (Auto)**：依「目標範圍 → 各 OnTriggerOn 群組 → 觸發效果 → battle tags → passive 效果」的順序自動串接。
*   **手動模式 (Manually)**：以 `m_CardDescription` 為本體，套用 `GetDescriptionParams()` 拿到的 `(OnPlay.0)` 等佔位符替換。

### 卡牌資訊面板（Tooltip 右側）
顯示順序：
1. 必要的專精（如果有）
2. 各效果的 `Infos`（去重）
3. 戰鬥標籤的解說
4. 被動效果的條件 + 標籤資訊
5. 使用類型（如果 `m_ShowTag = true`）
6. 詠唱卡額外加上詠唱說明於最頂

### 強化分支
*   `GetEnhenceBranchs(N)` 從 `EnhencePool` 抽出 N 個分支。
*   過濾條件：`BannedEnhence` 列出的不抽、`RCG_CardEnhenceCondition` 不通過的不抽。
*   每個分支會 clone 一份卡片並提升 `UpgradeLv`，再套用對應的強化邏輯。

### 卡牌融合
本類別**不直接定義融合規則**；融合時的「葉效果合併」交給各 `RCG_BattleSetting` 子類自己處理（見 `RCG_BattleSetting.GetFusionCandidateSettings / GetFusionBaseSetting`）。

## 注意事項

*   **詠唱卡的 CardTags 不需手動加入詠唱 Tag**：系統會在 `HasTag` 判斷時補上。但要確認 `CardType = Chant`。
*   **價格自動化**：編輯器頁面有「Auto Price」按鈕，會以「稀有度價值 × 5」覆蓋所有卡的 Price。**手動改過的價格會被覆蓋**，要保留請避免按。
*   **解鎖條件忘填**：`Unlock` 為空 = 預設解鎖；要做隱藏成就卡時記得填。
*   **m_Note / m_Author 對玩家不可見**：純粹開發者記事 / 署名用。
*   **Description 的多語系一致性**：手動描述模式要記得在所有語言都填；缺漏會 fallback 到預設語系。
*   **強化卡的 Enhance Pool 同源**：強化分支抽自卡片自身的 `EnhencePool`；同名卡的不同實例若要有不同強化路線需各自設定。

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardData.cs`
*   **繼承自**：`RCG_Asset<RCG_CardData>`
*   **實作介面**：`RCGI_Item` / `RCGI_CardData` / `RCGI_Unloackable`
*   **AssetGroup**：`AssetGroup.EditItems`，sort = `RCG_CardData`
*   **i18n 類別名 key**：未直接定義；`AssetGroup` 名稱會用於 EditItems 分類顯示。

### A.2 欄位對照（外層 `RCG_CardData`）

| 程式欄位 | 編輯器顯示 | 型別 | 說明 |
|---|---|---|---|
| `m_Data` | 卡牌設定 | `CardData` (內部巢狀類) | 主要資料；A.3 列詳細欄位 |
| `m_Effects` | 效果 | `List<RCG_CommonEffect>` | 觸發效果清單 |
| `m_PassiveEffects` | 被動效果 | `List<RCG_ConditionalPassiveBattleTag>` | 條件被動標籤；用於屬性查詢，不走 OnTriggerEffect |

### A.3 欄位對照（巢狀 `CardData`）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_LocalizeName` | 名稱(多國語言) | `RCG_LocalizeData` | `LocalizeName` | |
| `m_Cost` | 費用 | `int` | `Cost` | 預設 0 |
| `m_CardType` | 卡牌類型 | `CardType` (檔內 enum) | `CardType` | `Default` / `Chant` |
| `m_ChantTurn` | 詠唱回合數 | `IntVariable` | `ChantTurn` | `[Conditional(nameof(m_CardType), false, CardType.Chant)]` |
| `m_Rarity` | 稀有度 | `RCG_RarityTagGenData` | `Rarity` | |
| `m_TargetType` | 目標範圍 | `TargetType` (檔內 enum) | `TargetType` | 17 種值 |
| `m_UsedTypeTag` | 使用類型 | `RCG_UsedTypeTagGenData` | `UsedTypeTag` | |
| `m_Price` | 價格 | `int` | `Price` | 預設 100 |
| `m_SkillTags` | 專精 | `List<RCG_SkillTagGenData>` | `SkillTags` | |
| `m_CardTags` | 卡牌標籤 | `List<RCG_CardTagGenData>` | `CardTags` | |
| `m_BannedEnhence` | 禁用的強化分支 | `List<RCG_CardEnhenceGenData>` | `BannedEnhence` | |
| `m_EnhencePool` | 強化池 | `RCG_CardEnhenceDropPoolGenData` | — | |
| `m_ApplyEnhenceOnAllTiming` | `ApplyEnhenceOnAllTiming` | `bool` | — | |
| `m_UpgradeLv` | 強化等級 | `int` | `UpgradeLv` | 預設 0 |
| `m_EnhenceLocalize` | `EnhenceLocalize` | `RCG_LocalizeData` | — | 替代 `+1`/`+2` 顯示 |
| `m_CardIcon` | 卡圖 | `RCG_SpriteData` | `CardIcon` | 預設指向 `Card_AngelsWings` |
| `m_CardDescriptionType` | 卡牌描述方式 | `CardDescriptionType` (檔內 enum) | `CardDescriptionType` | `Auto` / `Manually` |
| `m_CardDescription` | 卡牌描述 | `RCG_LocalizeData` | `CardDescription` | `[Conditional(... Manually)]` |
| `m_DescriptionTemplate` | 描述範本 | `string` | `DescriptionTemplate` | `[Conditional(... Manually)]` |
| `m_Unlock` | 解鎖 | `RCG_UnlockEntry` | `Unlock` | |
| `m_HideInCodex` | 在圖鑑中隱藏 | `bool` | `HideInCodex` | |
| `m_Note` | 備註 | `string` | `Note` | |
| `m_Author` | `Author` | `string` | — | |
| `m_DataType` | （隱藏） | `DataType` | — | `[UCL_HideOnGUI]`，`BuiltIn` / `InGameRuntime` 等 |

### A.4 重要 Method 摘要

#### 介面實作
*   **`AddItem()`** → `RCG_DataService.Ins.m_DeckData.AddCard(ID)`，回傳 self；玩家獲得卡牌即執行此入口。
*   **`ToCardData`** → `this`；介面要求實作。
*   **`ToCardBattleData`** → `null`；**已棄用**，改用 `RCG_CardBattleData.CreateCard()` 統一生成。
*   **`Icon` / `IconSpriteData` / `IconTexture`** → `m_Data.m_CardIcon` 對應屬性。
*   **`ItemName` / `LocalizedName`** → `m_Data.LocalizeName`（含強化等級後綴邏輯）。
*   **`ShowInfo()`** → 開啟 `RCG_CardInfoPanel`。
*   **`GetPreviewDamage(target)`** → 預設 -1（非攻擊牌；CardData 層級不算傷害）。
*   **`UnlockEntry`** → `m_Data.m_Unlock`。

#### 描述系統
*   **`Description` (property)** → `DisplayDescription()`。
*   **`DisplayDescription(iData)`** → 詠唱卡先加詠唱標題、後接 `GetDescription`，最末附使用類型（若 `m_ShowTag`）。
*   **`GetDescription(iData, iShowBattleTags)`** → 主邏輯：
    *   `Manually` 模式 + `m_CardDescription` 非空 → 套參數後直接返回。
    *   `Auto` 模式 → 「目標描述 + 各 effect 描述 + battle tags」串接。
*   **`GetDescriptionTemplate(iData)`** → 產生 `(OnPlay.0)` 形式的範本字串（給 Manually 模式的 `Generate` 按鈕用）。
*   **`GetDescriptionParams(iData)`** → 拿出所有 effect 的參數列表（給 Manually 模板替換用）。
*   **`CardDisplayName`** → 卡名（詠唱卡會帶詠唱前綴 / 顏色）。
*   **`CostStr`** → `Cost.ToString()`。
*   **`ChantTitle(iData)`** → 「詠唱[X]」字串（綠色）。

#### 標籤 / 詞條 / 協力
*   **`CardTags`** → `m_Data.m_CardTags`（非詠唱 Tag）。
*   **`HasTag(tag)` / `HasTag(tags)`** → 內含「詠唱卡時自動視為含 `TagChant`」的特例邏輯。
*   **`HasTerm(term)`** → 對所有 `CardEffects` 遞迴查詢。
*   **`GetBattleTags()`** → 對所有 `CardEffects` 聚合。
*   **`GetPassiveTags(iData)`** → 回傳 `m_PassiveEffects`；用於 Cost 計算等屬性查詢。
*   **`GetCollaborators()`** → 對所有 `CardEffects` 聚合協力者，去重。
*   **`GetTagsDescription(showHiddenTags)`** → 標籤描述字串（隱藏標籤包 `[]`）。

#### 戰鬥與融合相關
*   **`CheckPlayable(iData)`** → 使用類型含 `Unplayable` 直接 false；其他對所有 `CardEffects.CheckPlayable` 取 AND。
*   **`CheckRequireSkill(skill)` / `CheckRequireSkill(skills)`** → 全條件 AND。
*   **`CanPlayThisCard(skills)`** → 「**任一專精符合即可**」（與 `CheckRequireSkill` 不同）。
*   **`GetBattleSettings<T>() / (Type)`** → 對所有 `m_Effects[i].m_CombineSetting` 遞迴。
*   **`OnTriggerEffect(triggerOn, iData)`** → 取 `m_Effects.GetEffects(triggerOn)`、若 iData 為 null 則建立 child，呼叫 `aEffects.TriggerEffects(iData)` 並 log 統計。

#### 強化分支
*   **`CanEnhence`** → `m_Data.m_UpgradeLv < 1 && !CardRarity.Equals(Cursed)`。
*   **`CreateEnhenceCard(ID)` (private)** → clone 自身，設新 ID，`m_UpgradeLv += 1`，標 `InGameRuntime`。
*   **`GetEnhenceBranchs(branchCount)` (yield)** → 從 `EnhencePool` 抽，過濾 `BannedEnhence` 與 `CheckCondition` 不通過的，逐個 clone + 套用強化資料。

#### 序列化 / 資料流
*   **`SaveToJson(iData)` (static)** → 寫入 `ID` + `IsRuntimeCardData` 兩欄位。
*   **`LoadFromJson(iData)` (static)** → `IsRuntimeCardData = true` → `RCG_DataService.Ins.m_GameRuntimeAsset.GetRuntimeData<RCG_CardData>(aID)`；否則 `Util.GetData(aID)`。
*   **`Save()`** → base + 清除 `RCG_CardFilter` cache。
*   **`CloneCard()`** → 透過 `SerializeToJson + DeserializeFromJson` 深拷貝。
*   **`CreateSelectAssetPage()`** → `RCG_CardDataEditorPage.Create()`。
*   **`OnLoadModule()` (static)** → `RCG_CardFilter.Clear()` + `Util.PrewarmAllAssets()`。
*   **`PreloadData(token)`** → 標記 `Preloaded` 後預載卡圖、稀有度圖示、各 effect 的 `PreloadData`。

#### 編輯 / 預覽
*   **`OnGUI(iDataDic)`** → 三段式繪製：CardData → Effects → PassiveEffects → Preview。Manually 模式下多顯示「Generate Description Template」按鈕與描述參數編輯器。
*   **`Preview(iDic, iIsShowEditButton)`** → 卡圖 + 名稱 + 標籤 + 描述 + 強化分支按鈕 + 解鎖條件 + DisplayCardInfos。
*   **`DisplayCardInfos(iShowOnUI)`** → 切換 `RCG_BattleSetting.IsShowOnUI` 後組裝完整 InfoPanel 內容（含詠唱資訊）。

### A.5 與其他系統的互動

*   **`RCG_CardBattleData`** — 戰場上實際運作的卡片實例；`RCG_CardData` 是模板，`RCG_CardBattleData.CreateCard(cardData, isChanted)` 產生實例。
*   **`RCG_CommonEffect`** — 卡牌效果單位；`m_Effects` 是它的 `List`，包含 `m_EffectTriggerOn` + `m_CombineSetting`。
*   **`RCG_ConditionalPassiveBattleTag`** — 被動條件式標籤；含 `m_Conditions` + `m_Tags`，影響屬性查詢但不參與 OnTrigger。
*   **`RCG_CardEnhenceData / RCG_CardEnhenceDropPoolGenData`** — 強化分支系統；`GetEnhenceBranchs` 的核心。
*   **`RCG_CardFilter`** — 卡片篩選 cache；`Save()` 與 `OnLoadModule()` 會清。
*   **`RCG_DataService.Ins.m_DeckData`** — 玩家牌組儲存；`AddItem` 寫入此處。
*   **`RCG_DataService.Ins.m_GameRuntimeAsset`** — runtime 卡片（如 CardToItem 產生的）儲存；`LoadFromJson` 在 `IsRuntimeCardData = true` 時讀取。
*   **`UI.RCG_CardInfoPanel` / `RCG_CardDataEditorPage` / `RCG_PreviewEnhenceCardPage`** — 主要 UI 入口。
*   **`RCG_CardDataComparer`** (檔內) — 排序 helper：先比專精類別 → 稀有度 → 費用降序。

### A.6 已知議題

*   `ToCardBattleData` 已棄用但保留 (`return null`)；註解明示「之後禁止用這個轉換為 CardBattleData」。
*   `CanEnhence` 暫時硬編碼「強化次數 ≤ 1」，未來解除限制需改此 property。
*   `m_PassiveEffects` 的 `GetPassiveTags` 中註解 `// QWQ23`、`// Append does not work QWQ?` 標示了實作疑慮（用 `Add` 取代 `Append`）。
*   `InitData()` 為 virtual 但目前是空實作；子類 `RCG_CardBattleData` 才真正用到。
*   舊版 `m_UpgradeCards` 欄位已註解掉（曾打算用清單方式列強化目標）。
