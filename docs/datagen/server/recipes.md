配方生成
=================

通过继承 `RecipeProvider` 并实现 `#buildRecipes` 来生成配方。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(event.includeServer(), MyRecipeProvider::new);
}
```

### 配方构建器
- **`ShapedRecipeBuilder`** — 有序合成：`.shaped(category, result)` → `.pattern(str)` → `.define(char, item)` → `.unlockedBy(name, trigger)` → `.save(writer)`
- **`ShapelessRecipeBuilder`** — 无序合成：`.shapeless(category, result)` → `.requires(item)` → `.unlockedBy(...)` → `.save(writer)`
- **`SimpleCookingRecipeBuilder`** — 烧炼/高炉/烟熏/营火：`.smelting(input, category, result, exp, time)`
- **`SingleItemRecipeBuilder`** — 切石机：`.stonecutting(input, category, result)`
- **`SmithingTransformRecipeBuilder`** — 锻造台转换：`.smithing(template, base, addition, category, result)`
- **`SmithingTrimRecipeBuilder`** — 锻造台盔甲纹饰：`.smithingTrim(template, base, addition, category)`
- **`SpecialRecipeBuilder`** — 特殊动态配方：`.special(serializer)`，仅生成空 JSON

### 条件配方
通过 `ConditionalRecipe.builder()` → `.addCondition(cond)` → `.addRecipe(recipe)` → `.generateAdvancement()` → `.build(writer, name)` 实现。

`IConditionBuilder` 接口简化条件构建：`and()`, `or()`, `not()`, `itemExists(mod, path)`, `FALSE()` 等。

### 自定义序列化器
创建实现 `FinishedRecipe` 的构建器，编码配方数据和解锁进度的 JSON。

[datagen]: ../index.md
[conditional]: ../../resources/server/conditional.md
