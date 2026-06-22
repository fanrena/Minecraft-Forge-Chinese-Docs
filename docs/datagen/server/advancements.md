进度生成
======================

通过构建 `AdvancementProvider` 并提供 `AdvancementSubProvider` 来生成[进度]。Forge 提供 `ForgeAdvancementProvider` 以获得更好的集成。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new ForgeAdvancementProvider(output, event.getLookupProvider(), event.getExistingFileHelper(),
            List.of(myGenerator))
    );
}
```

`Advancement$Builder` 用法：
- `.parent(adv)` — 设置父进度
- `.display(...)` — 设置显示信息
- `.rewards(...)` — 设置奖励
- `.addCriterion(name, trigger)` — 添加解锁条件
- `.requirements(...)` — 条件组合方式（AND/OR）
- `.save(writer, name, efh)` — 保存

```java
Advancement example = Advancement.Builder.advancement()
  .addCriterion("example_criterion", triggerInstance)
  .save(writer, name, existingFileHelper);
```

[advancements]: ../../resources/server/advancements.md
[datagen]: ../index.md
