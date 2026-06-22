全局战利品修改器生成
===============================

通过继承 `GlobalLootModifierProvider` 并实现 `#start` 来生成[全局战利品修改器][glm]。使用 `#add(name, instance)` 添加 GLM。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new MyGlobalLootModifierProvider(output, MOD_ID)
    );
}

// 在 GlobalLootModifierProvider#start 中
this.add("example_modifier", new ExampleModifier(
  new LootItemCondition[] { WeatherCheck.weather().setRaining(true).build() },
  "val1", 10, Items.DIRT
));
```

[glm]: ../../resources/server/glm.md
[datagen]: ../index.md
