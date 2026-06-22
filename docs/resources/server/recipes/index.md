配方
=======

配方是将一些对象转换为其他对象的机制。原版系统主要用于物品转换，但可以扩展为使用任何对象。

### 数据驱动
大多数原版配方由 JSON 数据驱动。通过 `RecipeManager` 加载和存储。
- `getRecipeFor` — 获取第一个匹配的配方
- `getRecipesFor` — 获取所有匹配的配方
- Forge 提供 `RecipeWrapper`（包装 `IItemHandler` → `Container`）

### Forge 扩展
- **物品堆结果** — 支持完整的 `ItemStack` 作为结果（含 count 和 nbt）
- **条件配方** — 见 [conditional] 页面
- **更大合成网格** — `ShapedRecipe#setCraftingSize`（注意线程安全）
- **额外原料类型** — 见 [ingredients] 页面

[wiki]: https://minecraft.wiki/w/Recipe
[advancement]: ../advancements.md
[datagen]: ../../../datagen/server/recipes.md
[conditional]: ../conditional.md
[ingredients]: ./ingredients.md
