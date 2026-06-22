战利品表生成
=====================

通过构建 `LootTableProvider` 并提供 `LootTableProvider$SubProviderEntry` 来生成[战利品表][loottable]。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new MyLootTableProvider(output, Collections.emptySet(), List.of(subProvider1, subProvider2))
    );
}
```

### 核心组件
- **`LootTableSubProvider`** — 实现 `#generate(writer)`，调用 `writer#accept` 添加战利品表
- **`BlockLootSubProvider`** — 方块战利品表，需实现 `#getKnownBlocks` 和 `#generate`
- **`EntityLootSubProvider`** — 实体战利品表，需实现 `#getKnownEntityTypes` 和 `#generate`

### 战利品表构建器
`LootTable#lootTable()` → `.withPool(pool)` → `.apply(function)`
`LootPool#lootPool()` → `.add(entry)` → `.when(condition)` → `.apply(function)` → `.setRolls(n)` → `.setBonusRolls(n)`

- **条目（`LootPoolEntryContainer`）**：定义操作，通常生成物品
- **条件（`LootItemCondition`）**：定义执行条件，可 `#or` 组合，`#invert` 取反
- **函数（`LootItemFunction`）**：修改执行结果
- **数值提供者（`NumberProvider`）**：决定战利品池执行次数

[loottable]: ../../resources/server/loottables.md
[datagen]: ../index.md
