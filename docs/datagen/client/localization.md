语言文件生成
===================

通过继承 `LanguageProvider` 并实现 `#addTranslations` 来生成模组的[语言文件][lang]。每个 `LanguageProvider` 子类代表一个独立的[语言环境][locale]（`en_us` 代表美式英语，`es_es` 代表西班牙语等）。实现后，必须[添加][datagen]到 `DataGenerator`。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeClient(),
        output -> new MyLanguageProvider(output, MOD_ID, "en_us")
    );
}
```

`LanguageProvider` 是一个字符串映射，将翻译键映射到本地化名称。使用 `#add` 添加翻译键映射，也有针对 `Block`、`Item`、`ItemStack`、`Enchantment`、`MobEffect`、`EntityType` 的便捷方法。

```java
// 在 LanguageProvider#addTranslations 中
this.addBlock(EXAMPLE_BLOCK, "示例方块");
this.add("object.examplemod.example_object", "示例对象");
```

!!! tip
    包含非美式英语字母数字字符的本地化名称可以直接提供。提供者会自动将字符转换为对应的 unicode 编码供游戏读取。

[lang]: ../../concepts/internationalization.md
[locale]: https://minecraft.wiki/w/Language#Languages
[datagen]: ../index.md
