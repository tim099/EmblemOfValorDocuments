---
title: Runtime 结构定义 (RCG_RuntimeStructData) 说明
description: RCG_RuntimeData 用的「型别表」：定义 Int/Float/String/Bool/Json/Enum/Struct/List/Dic 结构
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Runtime 结构定义

> 程式类别名称：`RCG_RuntimeStructData`

## 用途

**`RCG_RuntimeData` 使用的「型别定义表」**。系统需要动态定义资料结构时（不靠 C# 的 class 而靠资料），就在这里建一个 Struct 描述：栏位有哪些、各自是什么型别。例如「玩家本局状态」这个 Struct 可以含 `Score: Int / IsBoss: Bool / Inventory: List<Item>`。

继承自 `RCG_Asset<RCG_RuntimeStructData>`。

## 编辑器中的样貌

```
RCG_RuntimeStructData: <ID>
    StructType  ▾ Int / Float / String / Bool / Json / Enum / Struct / Dic / List
    Name(多国语言)
    Fields       (StructType=Struct)  ← 各栏位定义 dic：name → struct type ref
    ElementType  (StructType=List/Dic) ← 元素型别 ref
    Enums        (StructType=Enum)     ← enum 值列表
```

## 主要栏位

| 编辑器显示 | 必填 | 说明 |
|---|---|---|
| **StructType** | 是 | 9 种型别：5 种原始型别（Int/Float/String/Bool/Json）+ Enum + Struct + List + Dic |
| **Name** | 否 | 显示名（多语系） |
| **Fields** | StructType=Struct | 结构栏位 dic：栏位名 → 对另一个 RuntimeStructData 的引用 |
| **ElementType** | StructType=List/Dic | 元素型别引用 |
| **Enums** | StructType=Enum | 列举值的字串列表 |

## 行为说明

### 预设原始型别自动产生 (`PrimitiveDic`)
5 种原始型别会在第一次存取时懒建：`{ "Int": ..., "Float": ..., "String": ..., "Bool": ..., "Json": ... }`。**它们不需要手动建 Asset**，使用时直接引用 ID `"Int"` / `"Float"` 等即可。

### 泛型字串解析 (`CreateData`)
ID 带尖括号（如 `"List<Int>"` 或 `"Dic<String,Float>"`）时自动解析：
*   切出 StructType 字串（List / Dic）。
*   切出 element type 字串（取最后一个逗号后）。
*   建立并 cache 到 `GenericDic`。

### 预览 (`PreviewData`)
依 StructType 用对应 UI 绘制资料；递回展开 Struct / List / Dic。

### 编辑 (`DataOnGUI`)
依 StructType 绘制对应的编辑介面（IntField / NumField / TextArea / BoolField / Popup / 巢状资料展开等）。

### Json 序列化 (`SerializeDataToJson` / `DeserializeDataFromJson`)
全结构支援 JSON 来回转换；递回处理 Struct/List/Dic。

### 资料变动侦测
`DataOnGUI` 用 `iDataDic[PrevDataKey]` 比对当前资料；不一致就清掉 GUI 子 dic（避免 IntField cache 残留错误输入）。

## 注意事项

*   **`Event` 型别已被注解**（程式内 `// Event,`）：曾规划但未实作。
*   **原始型别 Asset 不要手动建**：用 `PrimitiveDic` 自动懒建即可，手动建会跟自动产生的冲突。
*   **List / Dic 的泛型语法**：用 `"List<元素型别ID>"` / `"Dic<KeyID,ValueID>"`（其中 Key 目前只支援 String，第一个值会被忽略只读最后一个）。
*   **递回上限 10 层**：`CreateData(layer)` 防无限递回；过深的巢状 struct 会回 null。
*   **修改 Struct 的 Fields**：原本资料不会自动迁移；改 schema 后旧资料的 dictionary 可能保留多余 key 或缺漏 key。

---

## 附录：程式人员参考 (Programmer Reference)

### A.1 类别资讯
*   **档案路径**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeStructData.cs`
*   **继承自**：`RCG_Asset<RCG_RuntimeStructData>`
*   **AssetGroup**：`Runtime`
*   **常数**：`s_PrimitiveTypes` = 5 种；`GenericTypeDic` = List/Dic（dic 的元素型别数）

### A.2 栏位对照

| 程式栏位 | 编辑器显示 | 型别 | 备注 |
|---|---|---|---|
| `m_StructType` | StructType | `StructType` enum | 9 种 |
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Fields` | Fields | `Dictionary<string, RCG_RuntimeStructGenData>` | `Conditional(Struct)` |
| `m_ElementType` | ElementType | `RCG_RuntimeStructGenData` | `Conditional(List/Dic)` |
| `m_Enums` | Enums | `List<string>` | `Conditional(Enum)` |

### A.3 重要 Method 摘要

*   **`PrimitiveDic` / `GenericDic` (static)** — 懒建快取。
*   **`GetAllIDs(useCache)`** — 含 primitives + generics + base IDs。
*   **`CreateData(string)` (override)** — 含泛型字串解析。
*   **`CreateData(int layer = 0)` (instance)** — 建初始值（含递回上限）。
*   **`DataOnGUI(data, fieldName, dataDic)`** — 主编辑介面，按 StructType 分流。
*   **`PreviewData(data, fieldName)`** — 主预览。
*   **`SerializeDataToJson / DeserializeDataFromJson`** — JSON 来回。
*   **`GetRuntimeObject(obj, key/int)` / `SetData(obj, key/int, val)` / `GetList / GetDictionary / GetFields / GetFieldInfo`** — 给外部存取资料的 helper。

### A.4 与其他系统的互动

*   **`RCG_RuntimeData`** — 引用此型别的 Asset。
*   **`RCG_RuntimeObject`** — 动态值容器。
*   **`RCG_RuntimeStructGenData`** — Asset Entry 包装；含 `GenericTypeLeft = '<'` 等常数。

### A.5 已知议题

*   `Event` 型别在 enum 中注解，存在但不会被使用。
*   `IsNone` 只判断 Struct 类型的 Fields 为空；其他类型不会被视为 None。
*   `CreateData` 的泛型解析仅取**最后一个** type 参数（适用 Dic 但未来若扩展可能改）。
