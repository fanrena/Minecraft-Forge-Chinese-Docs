自定义配方
==============

每个配方定义由三个组件组成：

1. **`Recipe`** — 持有数据和处理执行逻辑。实现 `#matches`、`#assemble`、`#getType`、`#getSerializer`
2. **`RecipeType`** — 定义配方所属类别（如 `SMELTING`、`CRAFTING`），需要时注册新类型
3. **`RecipeSerializer`** — 编解码配方 JSON 和网络通信。实现 `fromJson`、`toNetwork`、`fromNetwork`

### JSON 格式
```js
{
  "type": "examplemod:example_serializer",
  "input": { /* ingredient */ },
  "data": 0,
  "output": { /* stack */ }
}
```

### 非物品逻辑
如果配方不涉及物品变换，可在自定义 `Recipe` 中添加额外方法，使用 `RecipeManager#getAllRecipesFor` 获取后手动检查。

### 数据生成
自定义配方可以通过创建 `FinishedRecipe` 实现数据生成。

[forge]: ../../../concepts/registries.md
[datagen]: ../../../datagen/server/recipes.md
