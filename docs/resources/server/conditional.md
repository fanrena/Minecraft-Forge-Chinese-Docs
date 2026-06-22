条件加载数据
=========================

允许在模组间实现软依赖的条件加载系统。目前支持配方和进度。

### JSON 格式
使用 `"type": "forge:conditional"`，内嵌 `recipes` / `advancements` 数组，每个条目包含 `conditions`（AND 组合）和对应的 `recipe`/`advancement`。

### 内置条件
- `forge:true` / `forge:false` — 布尔常量
- `forge:not` / `forge:and` / `forge:or` — 布尔运算
- `forge:mod_loaded` — 检查模组是否加载
- `forge:item_exists` — 检查物品是否注册
- `forge:tag_empty` — 检查标签是否为空

### 自定义条件
实现 `ICondition` + `IConditionSerializer`，在 `RegisterEvent` 或 `FMLCommonSetupEvent` 中使用 `CraftingHelper#register` 注册。

[datagen]: ../../datagen/server/recipes.md
