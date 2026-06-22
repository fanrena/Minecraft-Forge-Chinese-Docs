战利品表
===========

战利品表是决定各种动作或场景发生时应该产生什么结果的逻辑文件。原版系统主要用于物品生成，但可以扩展为执行任意操作。

### 数据驱动
大多数原版战利品表由 JSON 数据驱动。详见 [Minecraft Wiki][wiki]。

### 使用战利品表
- `ResourceLocation` 指向 `data/<namespace>/loot_tables/<path>.json`
- 通过 `MinecraftServer#getLootData` → `LootDataResolver#getLootTable` 获取
- 使用 `LootParams$Builder` 构建参数，`LootContext$Builder` 构建上下文
- 方法：`getRandomItemsRaw`、`getRandomItems`、`fill`

### Forge 扩展
- **`LootTableLoadEvent`** — 加载时触发，取消则加载空表（不应用来修改掉落，应使用 GLM）
- **战利品池命名** — 使用 `name` 键命名
- **战利品等级** — `LootingLevelEvent` 影响战利品等级
- **烧炼多物品** — `SmeltItemFunction` 现在返回实际数量的物品
- **战利品表 ID 条件** — `forge:loot_table_id` 条件
- **工具操作条件** — `forge:can_tool_perform_action` 条件

[wiki]: https://minecraft.wiki/w/Loot_table
[glm]: ./glm.md
