---
title: 物件池资料 (RCG_PoolingData) 说明
description: 预载与池化 GameObject / VFX 的设定：类型、数量、是否保留范本
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 物件池资料

> 程式类别名称：`RCG_PoolingData`

## 用途

**物件池的设定**。指定哪些 prefab / VFX 要被池化、预先建多少个（prewarm）、是否保留范本。减少 runtime 建立物件的开销（如战斗中频繁出现的伤害数字、VFX 粒子）。

继承自 `RCG_Asset<RCG_PoolingData>`。

## 编辑器中的样貌

```
RCG_PoolingData: <ID>
    ResourceType  ▾ PrefabRes / VFXRes
    Prefab    (ResourceType=PrefabRes)
    VFX       (ResourceType=VFXRes)
    PrewarmCount    ← 预先建立的数量
    PreserveTemplate ← 是否保留范本（true 比较安全）
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **ResourceType** | 是 | `PrefabRes`（一般 prefab）/ `VFXRes`（特效资源） |
| **Prefab** | PrefabRes 时 | 对应的 prefab 资料 |
| **VFX** | VFXRes 时 | 对应的 VFX 资料（预设 Addressable 路径 `AttackVFXs/VFX_ArcaneMissileEffect`） |
| **PrewarmCount** | 是 | 进场时预先建立的数量 |
| **PreserveTemplate** | — | 范本是否被借出。`true` = 不借出范本（避免被 destroy 或修改后再用会出错） |

## 行为说明

### `LoadAsync<T>` / `CreateAsync<T>`
依 `ResourceType` 分流到 `m_Prefab` 或 `m_VFX` 的对应 method。

### Pooling Service 接入
透过 `RCG_PoolingGenData`（外部引用包装）→ `RCG_ObjectPoolService.Ins` 取出实例：
*   `GetObjectTemplate<T>` — 拿范本（读，不借出）
*   `CreateObject(parent)` — 从池借出实例
*   `Delete(obj)` — 还回池子

### 预设 ID
`RCG_PoolingGenData.DefaultID = "ItemDisplay"`（道具显示用）；`ItemDisplayerSmallID = "ItemDisplayerSmall"`。

## 注意事项

*   **`PreserveTemplate = false`** 表示范本本身也会被借出——若范本被改 / destroy，下次取会出错。**预设 true 比较安全**。
*   **`PrewarmCount` 不包含范本**：实际初始物件数量 = PrewarmCount + 1（范本）。
*   **VFXRes 的预设值**指向 `VFX_ArcaneMissileEffect`：建立新池子要记得换成自己的 VFX。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_PoolingData.cs`
*   **继承自**：`RCG_Asset<RCG_PoolingData>`
*   **AssetGroup**：`EditGameSetting`

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_ResourceType` | ResourceType | `ResourceType` enum | `PrefabRes` / `VFXRes` |
| `m_Prefab` | Prefab | `RCG_PrefabResData` | `Conditional(PrefabRes)` |
| `m_VFX` | VFX | `RCG_VFXResData` | `Conditional(VFXRes)`；预设 `VFX_ArcaneMissileEffect` |
| `m_PrewarmCount` | PrewarmCount | `int` | 预设 1 |
| `m_PreserveTemplate` | PreserveTemplate | `bool` | 预设 true |

### A.3 重要 Method

*   **`LoadAsync<T>(token)`** — 载入资源。
*   **`CreateAsync<T>(token, parent)`** — 建立实例。
*   **`RCG_PoolingGenData.GetObjectTemplate<T>` / `CreateObject<T> / CreateObject / Delete`** — 对外实际使用的入口。

### A.4 与其他系统的互动

*   **`RCG_ObjectPoolService`** — runtime 池管理服务。
*   **`RCG_PrefabResData / RCG_VFXResData`** — 资源载入封装。
*   **`PathConst.AttackVFXs`** — VFX 预设路径常数。
