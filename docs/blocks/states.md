方块状态
============

遗留行为
---------------------------------------

在 Minecraft 1.7 及更早版本中，需要存储放置或状态数据但没有 BlockEntity 的方块使用**元数据**。元数据是与方块一起存储的额外数字，允许在同一方块内实现不同的旋转、朝向甚至完全独立的行为。

然而，元数据系统令人困惑且有限，因为它仅仅是作为数字与方块 ID 一起存储，除了代码中的注释外没有任何意义。例如，要实现一个可以朝向某个方向并且可以位于方块空间上半部分或下半部分的方块（如楼梯）：

```Java
switch (meta) {
  case 0: { ... } // 朝南，位于方块下半部分
  case 1: { ... } // 朝南，位于方块上半部分
  case 2: { ... } // 朝北，位于方块下半部分
  case 3: { ... } // 朝北，位于方块上半部分
  // ... 以此类推 ...
}
```

因为这些数字本身没有任何意义，除非拥有源代码和注释，否则没有人知道它们代表什么。

状态的引入
---------------------------------------

在 Minecraft 1.8 及以上版本中，元数据系统和方块 ID 系统已被弃用，并最终被**方块状态系统**取代。方块状态系统将方块属性的细节从方块的其他行为中抽象出来。

方块的每个*属性*由 `Property<?>` 实例描述。方块属性的例子包括乐器（`EnumProperty<NoteBlockInstrument>`）、朝向（`DirectionProperty`）、是否被激活（`Property<Boolean>`）等。每个属性具有由 `Property<T>` 参数化的类型 `T` 的值。

可以从 `Block` 和从 `Property<?>` 到其关联值的映射构造一个唯一的配对。这个唯一的配对称为 `BlockState`。

先前无意义的元数据值系统已被方块属性系统取代，后者更易于解释和处理。以前，一个朝东且被激活或按下的石质按钮表示为 "`minecraft:stone_button`，元数据为 `9`"。现在，这表示为 "`minecraft:stone_button[facing=east,powered=true]`"。

方块状态的正确使用
---------------------------------------

`BlockState` 系统是一个灵活且强大的系统，但它也有局限性。`BlockState` 是不可变的，并且其所有属性的组合在游戏启动时生成。这意味着拥有具有许多属性和可能值的 `BlockState` 会减慢游戏的加载速度，并使试图理解你的方块逻辑的人感到困惑。

并非所有方块和情况都需要使用 `BlockState`；只有方块最基本的属性才应放入 `BlockState`，而其他情况更适合使用 `BlockEntity` 或作为单独的 `Block`。始终考虑你是否真的需要为你的目的使用方块状态。

!!! note
    一个好的经验法则是：**如果它有不同名称，则应是一个单独的方块**。

一个例子是制作椅子方块：椅子的*方向*应是一个*属性*，而不同的*木材种类*应分为不同的方块。
一个朝东的"橡木椅子"（`oak_chair[facing=east]`）与一个朝西的"云杉椅子"（`spruce_chair[facing=west]`）是不同的。

实现方块状态
---------------------------------------

在你的 Block 类中，为你 Block 的每个属性创建或引用 `static final` 的 `Property<?>` 对象。你可以自由创建自己的 `Property<?>` 实现，但本文不涉及具体的实现方法。原版代码提供了几个便利的实现：

* `IntegerProperty`
    * 实现 `Property<Integer>`。定义一个持有整数值的属性。
    * 通过调用 `IntegerProperty#create(String propertyName, int minimum, int maximum)` 创建。
* `BooleanProperty`
    * 实现 `Property<Boolean>`。定义一个持有 `true` 或 `false` 值的属性。
    * 通过调用 `BooleanProperty#create(String propertyName)` 创建。
* `EnumProperty<E extends Enum<E>>`
    * 实现 `Property<E>`。定义一个可以取枚举类值的属性。
    * 通过调用 `EnumProperty#create(String propertyName, Class<E> enumClass)` 创建。
    * 也可以只使用枚举值的一个子集（例如 16 种 `DyeColor` 中的 4 种）。参见 `EnumProperty#create` 的重载方法。
* `DirectionProperty`
    * 这是 `EnumProperty<Direction>` 的便利实现。
    * 还提供了几个便利的谓词。例如，要获取表示基本方向的属性，调用 `DirectionProperty.create("<name>", Direction.Plane.HORIZONTAL)`；要获取 X 轴方向，调用 `DirectionProperty.create("<name>", Direction.Axis.X)`。

`BlockStateProperties` 类包含了共享的原版属性，应尽可能使用或引用它们，而不是创建你自己的属性。

当你有了所需的 `Property<>` 对象后，在你的 Block 类中重写 `Block#createBlockStateDefinition(StateDefinition$Builder)`。在该方法中，调用 `StateDefinition$Builder#add(...);`，参数为你希望方块拥有的每个 `Property<?>`。

每个方块还会有一个自动为你选择的"默认"状态。你可以通过在构造函数中调用 `Block#registerDefaultState(BlockState)` 方法来更改这个"默认"状态。当你的方块被放置时，它将变为这个"默认"状态。来自 `DoorBlock` 的一个示例：

```Java
this.registerDefaultState(
  this.stateDefinition.any()
    .setValue(FACING, Direction.NORTH)
    .setValue(OPEN, false)
    .setValue(HINGE, DoorHingeSide.LEFT)
    .setValue(POWERED, false)
    .setValue(HALF, DoubleBlockHalf.LOWER)
);
```

如果你希望更改放置方块时使用的 `BlockState`，可以重写 `Block#getStateForPlacement(BlockPlaceContext)`。例如，这可以用来根据玩家放置方块时所处的位置来设置方块的方向。

由于 `BlockState` 是不可变的，并且所有属性组合都在游戏启动时生成，调用 `BlockState#setValue(Property<T>, T)` 将简单地转到 `Block` 的 `StateHolder` 并请求具有你想要的值组合的 `BlockState`。

由于所有可能的 `BlockState` 都在启动时生成，你可以并且被鼓励使用引用相等运算符（`==`）来检查两个 `BlockState` 是否相等。

使用 `BlockState`
---------------------

你可以通过调用 `BlockState#getValue(Property<?>)` 来获取属性的值，传入你想要获取值的属性。
如果你想要获取具有不同值集的 `BlockState`，只需调用 `BlockState#setValue(Property<T>, T)`，传入属性和其值。

你可以使用 `Level#setBlockAndUpdate(BlockPos, BlockState)` 和 `Level#getBlockState(BlockPos)` 在世界中获取和放置 `BlockState`。如果你要放置一个 `Block`，调用 `Block#defaultBlockState()` 获取"默认"状态，然后按照上述方法使用 `BlockState#setValue(Property<T>, T)` 来实现所需的状态。
