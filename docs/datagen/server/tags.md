标签生成
==============

通过继承 `TagsProvider` 并实现 `#addTags` 来生成[标签][tags]。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new MyBlockTagsProvider(output, event.getLookupProvider(), MOD_ID, event.getExistingFileHelper())
    );
}
```

### TagAppender 方法
- `#add(object)` — 通过资源键添加对象
- `#addOptional(name)` — 通过名称添加（可选依赖）
- `#addTag(tag)` — 添加整个标签
- `#addOptionalTag(name)` — 通过名称添加标签（可选）
- `#replace(true)` — 替换而非追加
- `#remove(object)` — 从标签中移除

### 现有标签提供者
| 注册表对象 | 标签提供者 |
|:---:|:---:|
| Block | `BlockTagsProvider`* |
| Item | `ItemTagsProvider` |
| EntityType | `EntityTypeTagsProvider` |
| Fluid | `FluidTagsProvider` |
| Biome | `BiomeTagsProvider` |
| 更多 | ... |

\* Forge 添加

`ItemTagsProvider#copy(blockTag, itemTag)` — 一键复制方块标签到物品标签。

### 自定义标签提供者
继承 `TagsProvider`，传入注册表键。若希望使用对象本身添加标签（而非 ResourceKey），继承 `IntrinsicHolderTagsProvider`。

[tags]: ../../resources/server/tags.md
[datagen]: ../index.md
