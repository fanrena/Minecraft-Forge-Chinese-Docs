原料（Ingredient）
===========

`Ingredient` 是物品输入的谓词处理器，检查 `ItemStack` 是否满足配方输入条件。

### Forge 提供的类型
- **`forge:nbt`**（`StrictNBTIngredient`）— 精确匹配物品、伤害和标签
- **`forge:partial_nbt`**（`PartialNBTIngredient`）— 仅匹配指定的标签键
- **`forge:intersection`** — 必须匹配所有子原料（AND）
- **`forge:difference`** — 必须在 base 中但不在 subtracted 中（SUB）
- **CompoundIngredient** — 无 `type` 字段，原料数组 OR 组合

### 自定义原料
1. 继承 `AbstractIngredient`，实现 `test`、`isSimple`、`getSerializer`、`toJson`
2. 实现 `IIngredientSerializer`（`parse` JSON/网络、`write`）
3. 使用 `CraftingHelper#register` 在 `RegisterEvent` 或 `FMLCommonSetupEvent` 中注册

[recipes]: https://minecraft.wiki/w/Recipe#List_of_recipe_types
[datagen]: ../../../datagen/server/recipes.md
