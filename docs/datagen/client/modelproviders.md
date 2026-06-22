模型生成
================

[模型]和方块状态可以通过继承相关提供者来生成。

### 模型文件（ModelFile）
- `ExistingModelFile` — 通过 `ExistingFileHelper` 检查模型是否已存在
- `UncheckedModelFile` — 假设模型存在（不推荐使用）

### 模型构建器（ModelBuilder）
`ModelBuilder` 表示一个待生成的 `ModelFile`。设置父模型、面、纹理、变换等。

- `BlockModelBuilder` — 方块模型。额外支持 `rootTransform`（根变换）
- `ItemModelBuilder` — 物品模型。额外支持 `overrides`（覆盖）

### 模型提供者
- `BlockModelProvider` — 在 `models/block` 中生成方块模型
- `ItemModelProvider` — 在 `models/item` 中生成物品模型。`#basicItem` 使用 `item/generated` 模板

### 方块状态提供者（BlockStateProvider）
负责生成 blockstate JSON、方块模型和物品模型。实现 `#registerStatesAndModels`。

- **变体（Variant）**：`VariantBlockStateBuilder` — 每个 `BlockState` 只能匹配一个变体
- **多部分（Multipart）**：`MultiPartBlockStateBuilder` — 多个部分可以同时叠加（如红石线）

### 自定义加载器构建器
Forge 提供的内置加载器数据生成：
- `DynamicFluidContainerModelBuilder` — 桶模型
- `CompositeModelBuilder` — 复合模型
- `ItemLayersModelBuilder` — 多层物品模型
- `SeparateTransformsModelBuilder` — 按变换切换模型
- `ObjModelBuilder` — OBJ 模型

```java
builder.customLoader(ObjModelBuilder::begin)
  .modelLocation(modLoc("models/block/model.obj"))
  .flipV(true)
  .end()
  .texture("particle", mcLoc("block/dirt"));
```

[models]: ../../resources/client/models/index.md
[datagen]: ../index.md
