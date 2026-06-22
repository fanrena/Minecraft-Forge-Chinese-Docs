数据包注册表对象生成
==================================

通过构建 `DatapackBuiltinEntriesProvider` 并提供 `RegistrySetBuilder` 来生成数据包注册表对象。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new DatapackBuiltinEntriesProvider(output, event.getLookupProvider(),
            new RegistrySetBuilder().add(/* ... */), Set.of(MOD_ID))
    );
}
```

`RegistrySetBuilder`：
- `.add(registryKey, bootstrap -> { ... })` — 添加注册表条目
- `BootstapContext#register(key, object)` — 注册对象
- `BootstapContext#lookup(registryKey)` — 获取 `HolderGetter` 以引用其他数据包注册表对象或标签

```java
new RegistrySetBuilder()
  .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
    bootstrap.register(EXAMPLE_CONFIGURED_FEATURE, new ConfiguredFeature<>(Feature.ORE, /* ... */));
  })
  .add(Registries.PLACED_FEATURE, bootstrap -> {
    HolderGetter<ConfiguredFeature<?, ?>> configured = bootstrap.lookup(Registries.CONFIGURED_FEATURE);
    bootstrap.register(EXAMPLE_PLACED_FEATURE, new PlacedFeature(configured.getOrThrow(EXAMPLE_CONFIGURED_FEATURE), List.of()));
  });
```

[datagen]: ../index.md
