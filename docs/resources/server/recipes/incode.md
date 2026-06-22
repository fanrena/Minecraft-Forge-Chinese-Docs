非数据包配方
====================

部分配方子系统尚未迁移到数据驱动方式，仍需要在代码中实现。

### 酿造配方
通过 `BrewingRecipeRegistry#addRecipe` 在 `FMLCommonSetupEvent` 中添加（注意线程安全）。
- 默认实现：输入 + 催化剂 + 输出
- 也可提供 `IBrewingRecipe` 实例（实现 `isInput`、`isIngredient`、`getOutput`）

### 铁砧配方
使用 `AnvilUpdateEvent`（Forge 事件总线），包含左右物品、输出、经验等级和材料数量。可通过取消事件阻止输出。

### 织布机配方
织布机负责将染料和图案应用于旗帜。自定义旗帜图案通过注册 `BannerPattern` 实现。
- `minecraft:no_item_required` 标签中的图案可直接在织布机中使用
- 不在该标签中的图案需要配套的 `BannerPatternItem`

[recipe]: ./custom.md
[cancel]: ../../../concepts/events.md
