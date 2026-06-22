方块
======

方块显然是 Minecraft 世界的基础。它们构成了所有的地形、结构和机械。如果你有兴趣制作模组，那么你很可能会想要添加一些方块。本页面将指导你完成方块的创建，以及你可以用它们做的一些事情。

创建方块
----------------

### 基础方块

对于简单的方块，不需要特殊功能（比如圆石、木板等），则不需要自定义类。你可以通过使用 `BlockBehaviour$Properties` 对象实例化 `Block` 类来创建方块。这个 `BlockBehaviour$Properties` 对象可以使用 `BlockBehaviour$Properties#of` 创建，并可以通过调用其方法进行自定义。例如：

- `strength` - 硬度控制破坏方块所需的时间。这是一个任意值。作为参考，石头的硬度为 1.5，泥土为 0.5。如果方块应该是不可破坏的，则应将硬度设为 -1.0，参见 `Blocks#BEDROCK` 的定义作为示例。抗性控制方块的爆炸抗性。作为参考，石头的抗性为 6.0，泥土为 0.5。
- `sound` - 控制方块在被击打、破坏或放置时发出的声音。需要 `SoundType` 参数，更多详情请参见[声音]页面。
- `lightLevel` - 控制方块的发光亮度。接受一个以 `BlockState` 为参数、返回 0 到 15 之间值的函数。
- `friction` - 控制方块的滑动程度。作为参考，冰的滑动系数为 0.98。

所有这些方法都是**可链式调用**的，这意味着你可以按顺序调用它们。有关示例，请参见 `Blocks` 类。

!!! note
    方块没有设置其 `CreativeModeTab` 的设值方法。如果方块有关联的物品（例如 `BlockItem`），这由 [`BuildCreativeModeTabContentsEvent`][creativetabs] 处理。此外，方块也没有翻译键的设值方法，因为翻译键是通过注册表名称经由 `Block#getDescriptionId` 自动生成的。

### 高级方块

当然，上述方法只允许创建非常基础的方块。如果你想要添加功能，比如玩家交互，就需要自定义类。然而，`Block` 类有许多方法，不幸的是无法在此一一记录。关于你可以用方块做的事情，请参见本节中的其他页面。

注册方块
-------------------

方块必须被[注册][registering]才能正常工作。

!!! important
    世界中的方块和物品栏中的"方块"是完全不同的概念。世界中的方块由 `BlockState` 表示，其行为由 `Block` 实例定义。与此同时，物品栏中的物品是 `ItemStack`，由 `Item` 控制。作为连接 `Block` 和 `Item` 这两个不同世界的桥梁，存在着 `BlockItem` 类。`BlockItem` 是 `Item` 的子类，它有一个 `block` 字段，持有对其所代表的 `Block` 的引用。`BlockItem` 定义了"方块"作为物品时的一些行为，比如右键点击放置方块。一个 `Block` 可以没有对应的 `BlockItem`（例如 `minecraft:water` 作为方块存在，但不作为物品存在，因此无法在物品栏中持有它）。

    当注册一个方块时，*仅仅*注册了一个方块。该方块并不会自动拥有 `BlockItem`。要为方块创建一个基本的 `BlockItem`，应将 `BlockItem` 的注册表名称设置为其对应 `Block` 的注册表名称。也可以使用 `BlockItem` 的自定义子类。一旦为方块注册了 `BlockItem`，就可以通过 `Block#asItem` 来获取它。如果没有为 `Block` 注册 `BlockItem`，`Block#asItem` 将返回 `Items#AIR`，因此如果你不确定所使用的 `Block` 是否有对应的 `BlockItem`，请检查 `Block#asItem` 是否返回 `Items#AIR`。

#### 可选注册方块

过去有一些模组允许用户通过配置文件禁用方块/物品。但是，你不应该这样做。可注册的方块数量没有限制，因此请注册你模组中的所有方块！如果你希望通过配置文件禁用某个方块，你应该禁用其合成配方。如果你希望在创造模式标签页中禁用该方块，请在 [`BuildCreativeModeTabContentsEvent`][creativetabs] 中构建内容时使用 `FeatureFlag`。

延伸阅读
---------------

关于方块属性的信息，例如原版中用于栅栏、墙等方块的属性，请参见[方块状态]章节。

[sounds]: ../gameeffects/sounds.md
[creativetabs]: ../items/index.md#creative-tabs
[registering]: ../concepts/registries.md#methods-for-registering
[blockstates]: states.md
