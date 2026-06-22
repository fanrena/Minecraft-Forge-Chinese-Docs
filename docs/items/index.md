物品
=====

与方块一样，物品是大多数模组的关键组成部分。方块构成了你周围的地形，而物品存在于物品栏中。

创建物品
----------------

### 基础物品

不需要特殊功能的基础物品（例如木棍或糖）不需要自定义类。你可以通过使用 `Item$Properties` 对象实例化 `Item` 类来创建物品。这个 `Item$Properties` 对象可以通过构造函数创建，并通过调用其方法进行自定义。例如：

|      方法        |                  描述                  |
|:----------------:|:---------------------------------------|
| `requiredFeatures` | 设置需要在 `CreativeModeTab` 中看到此物品所需的 `FeatureFlag`。 |
| `durability`       | 设置此物品的最大耐久值。如果大于 `0`，则会添加两个物品属性"damaged"和"damage"。 |
| `stacksTo`         | 设置最大堆叠数量。你不能同时拥有可损耗和可堆叠的物品。 |
| `setNoRepair`      | 使此物品无法修复，即使它是可损耗的。 |
| `craftRemainder`   | 设置此物品的容器物品，类似于熔岩桶使用后返还空桶的方式。 |

以上方法是可链式调用的，意味着它们 `return this` 以便于连续调用。

### 高级物品

如上所述设置物品属性仅适用于简单物品。如果你想要更复杂的物品，你应该继承 `Item` 并重写其方法。

## 创造模式标签页

物品可以通过[模组事件总线][modbus]上的 `BuildCreativeModeTabContentsEvent` 添加到 `CreativeModeTab` 中。可以通过 `#accept` 添加物品，无需任何额外配置。

```java
// 在 MOD 事件总线上注册
// 假设我们有 RegistryObject<Item> 和 RegistryObject<Block> 分别名为 ITEM 和 BLOCK
@SubscribeEvent
public void buildContents(BuildCreativeModeTabContentsEvent event) {
  // 添加到材料标签页
  if (event.getTabKey() == CreativeModeTabs.INGREDIENTS) {
    event.accept(ITEM);
    event.accept(BLOCK); // 接受 ItemLike，假设方块已注册物品
  }
}
```

你还可以通过 `FeatureFlagSet` 中的 `FeatureFlag` 或一个布尔值（确定玩家是否有权限查看 Operator 创造标签页）来启用或禁用物品的添加。

### 自定义创造模式标签页

自定义 `CreativeModeTab` 必须[注册][registering]。构建器可以通过 `CreativeModeTab#builder` 创建。标签页可以设置标题、图标、默认物品以及其他一些属性。此外，Forge 还提供了额外的方法来自定义标签页的图片、标签和槽位颜色、标签页的排序位置等。

```java
// 假设我们有一个名为 REGISTRAR 的 DeferredRegister<CreativeModeTab>
// 假设我们有 RegistryObject<Item> 和 RegistryObject<Block> 分别名为 ITEM 和 BLOCK
public static final RegistryObject<CreativeModeTab> EXAMPLE_TAB = REGISTRAR.register("example", () -> CreativeModeTab.builder()
  // 设置标签页的显示名称
  .title(Component.translatable("item_group." + MOD_ID + ".example"))
  // 设置创造标签页的图标
  .icon(() -> new ItemStack(ITEM.get()))
  // 向标签页添加默认物品
  .displayItems((params, output) -> {
    output.accept(ITEM.get());
    output.accept(BLOCK.get());
  })
  .build()
);
```

注册物品
-------------------

物品必须[注册][registering]才能正常使用。

[modbus]: ../concepts/events.md#模组事件总线
[registering]: ../concepts/registries.md#注册方法
