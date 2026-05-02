---
title: 物件池資料 (RCG_PoolingData) 說明
description: 預載與池化 GameObject / VFX 的設定：類型、數量、是否保留範本
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-en
---

> [!WARNING]
> Translation pending — this file needs an English translation.
The original zh-Hant content is included below for reference.


# 物件池資料

> 程式類別名稱：`RCG_PoolingData`

## 用途

**物件池的設定**。指定哪些 prefab / VFX 要被池化、預先建多少個（prewarm）、是否保留範本。減少 runtime 建立物件的開銷（如戰鬥中頻繁出現的傷害數字、VFX 粒子）。

繼承自 `RCG_Asset<RCG_PoolingData>`。

## 編輯器中的樣貌

```
RCG_PoolingData: <ID>
    ResourceType  ▾ PrefabRes / VFXRes
    Prefab    (ResourceType=PrefabRes)
    VFX       (ResourceType=VFXRes)
    PrewarmCount    ← 預先建立的數量
    PreserveTemplate ← 是否保留範本（true 比較安全）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **ResourceType** | 是 | `PrefabRes`（一般 prefab）/ `VFXRes`（特效資源） |
| **Prefab** | PrefabRes 時 | 對應的 prefab 資料 |
| **VFX** | VFXRes 時 | 對應的 VFX 資料（預設 Addressable 路徑 `AttackVFXs/VFX_ArcaneMissileEffect`） |
| **PrewarmCount** | 是 | 進場時預先建立的數量 |
| **PreserveTemplate** | — | 範本是否被借出。`true` = 不借出範本（避免被 destroy 或修改後再用會出錯） |

## 行為說明

### `LoadAsync<T>` / `CreateAsync<T>`
依 `ResourceType` 分流到 `m_Prefab` 或 `m_VFX` 的對應 method。

### Pooling Service 接入
透過 `RCG_PoolingGenData`（外部引用包裝）→ `RCG_ObjectPoolService.Ins` 取出實例：
*   `GetObjectTemplate<T>` — 拿範本（讀，不借出）
*   `CreateObject(parent)` — 從池借出實例
*   `Delete(obj)` — 還回池子

### 預設 ID
`RCG_PoolingGenData.DefaultID = "ItemDisplay"`（道具顯示用）；`ItemDisplayerSmallID = "ItemDisplayerSmall"`。

## 注意事項

*   **`PreserveTemplate = false`** 表示範本本身也會被借出——若範本被改 / destroy，下次取會出錯。**預設 true 比較安全**。
*   **`PrewarmCount` 不包含範本**：實際初始物件數量 = PrewarmCount + 1（範本）。
*   **VFXRes 的預設值**指向 `VFX_ArcaneMissileEffect`：建立新池子要記得換成自己的 VFX。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_PoolingData.cs`
*   **繼承自**：`RCG_Asset<RCG_PoolingData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_ResourceType` | ResourceType | `ResourceType` enum | `PrefabRes` / `VFXRes` |
| `m_Prefab` | Prefab | `RCG_PrefabResData` | `Conditional(PrefabRes)` |
| `m_VFX` | VFX | `RCG_VFXResData` | `Conditional(VFXRes)`；預設 `VFX_ArcaneMissileEffect` |
| `m_PrewarmCount` | PrewarmCount | `int` | 預設 1 |
| `m_PreserveTemplate` | PreserveTemplate | `bool` | 預設 true |

### A.3 重要 Method

*   **`LoadAsync<T>(token)`** — 載入資源。
*   **`CreateAsync<T>(token, parent)`** — 建立實例。
*   **`RCG_PoolingGenData.GetObjectTemplate<T>` / `CreateObject<T> / CreateObject / Delete`** — 對外實際使用的入口。

### A.4 與其他系統的互動

*   **`RCG_ObjectPoolService`** — runtime 池管理服務。
*   **`RCG_PrefabResData / RCG_VFXResData`** — 資源載入封裝。
*   **`PathConst.AttackVFXs`** — VFX 預設路徑常數。