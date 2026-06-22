# 方块实体

`BlockEntity`（方块实体）类似于绑定到方块的简化的 `Entity`（实体）。
它们用于存储动态数据、执行基于 tick 的任务以及进行动态渲染。
原版 Minecraft 中的一些例子包括：箱子中物品栏的处理、熔炉的烧炼逻辑、信标的效果范围。
模组中还有更高级的例子，如采石场、分拣机、管道和显示器。

!!! note
    `BlockEntity` 并非万能的解决方案，使用不当会导致卡顿。
    尽可能避免使用它们。

## 注册

方块实体是动态创建和移除的，因此它们本身并不是注册表对象。

为了创建一个 `BlockEntity`，你需要继承 `BlockEntity` 类。相应地，会注册另一个对象来轻松创建和引用动态对象的*类型*。对于 `BlockEntity`，这些被称为 `BlockEntityType`。

`BlockEntityType` 可以像任何其他注册表对象一样被[注册][registration]。要构造一个 `BlockEntityType`，可以通过 `BlockEntityType$Builder#of` 使用其构建器形式。该方法接受两个参数：一个 `BlockEntityType$BlockEntitySupplier`（接受 `BlockPos` 和 `BlockState` 来创建关联 `BlockEntity` 的新实例），以及一个可变参数的 `Block` 数组，表示此 `BlockEntity` 可以附加到的方块。通过调用 `BlockEntityType$Builder#build` 来完成 `BlockEntityType` 的构建。这需要一个 `Type`，表示在 `DataFixer` 中用于引用此注册表对象的类型安全引用。由于 `DataFixer` 对于模组来说是一个可选系统，因此可以传入 `null`。

```java
// 对于某个 DeferredRegister<BlockEntityType<?>> REGISTER
public static final RegistryObject<BlockEntityType<MyBE>> MY_BE = REGISTER.register("mybe", () -> BlockEntityType.Builder.of(MyBE::new, validBlocks).build(null));

// 在 MyBE 中，一个 BlockEntity 的子类
public MyBE(BlockPos pos, BlockState state) {
  super(MY_BE.get(), pos, state);
}
```

## 创建 `BlockEntity`

要创建 `BlockEntity` 并将其附加到 `Block`，必须在你 `Block` 的子类上实现 `EntityBlock` 接口。必须实现 `EntityBlock#newBlockEntity(BlockPos, BlockState)` 方法并返回你的 `BlockEntity` 的新实例。

## 在 `BlockEntity` 中存储数据

为了保存数据，请重写以下两个方法：
```java
BlockEntity#saveAdditional(CompoundTag tag)

BlockEntity#load(CompoundTag tag)
```
每当包含 `BlockEntity` 的 `LevelChunk` 从标签加载或保存到标签时，都会调用这些方法。
使用它们来读取和写入你的方块实体类中的字段。

!!! note
    每当你的数据发生变化时，你需要调用 `BlockEntity#setChanged`；否则，包含你 `BlockEntity` 的 `LevelChunk` 可能会在保存世界时被跳过。

!!! important
    务必调用父类的 `super` 方法！

    `super` 方法保留了以下标签名称：`id`、`x`、`y`、`z`、`ForgeData` 和 `ForgeCaps`。

## 使 `BlockEntity` 进行 tick

如果你需要一个进行 tick 的 `BlockEntity`，例如跟踪烧炼过程中的进度，则必须在 `EntityBlock` 中实现并重写另一个方法：`EntityBlock#getTicker(Level, BlockState, BlockEntityType)`。这可以根据玩家所处的逻辑侧实现不同的 tick 方法，或者只实现一个通用的 tick 方法。无论哪种情况，都必须返回一个 `BlockEntityTicker`。由于这是一个函数式接口，因此可以直接传入一个表示 tick 方法的方法引用：

```java
// 在某个 Block 子类中
@Nullable
@Override
public <T extends BlockEntity> BlockEntityTicker<T> getTicker(Level level, BlockState state, BlockEntityType<T> type) {
  return type == MyBlockEntityTypes.MYBE.get() ? MyBlockEntity::tick : null;
}

// 在 MyBlockEntity 中
public static void tick(Level level, BlockPos pos, BlockState state, MyBlockEntity blockEntity) {
  // 执行操作
}
```

!!! note
    此方法每个 tick 都会被调用；因此，应避免在其中进行复杂计算。如果可能，你应该每 X 个 tick 执行一次更复杂的计算。（每秒的 tick 数可能低于 20（二十），但不会更高）

## 将数据同步到客户端

有三种方法可以将数据同步到客户端：在区块加载时同步、在方块更新时同步以及使用自定义网络消息同步。

### 在 LevelChunk 加载时同步

为此，你需要重写
```java
BlockEntity#getUpdateTag()

IForgeBlockEntity#handleUpdateTag(CompoundTag tag)
```
同样，这相当简单，第一个方法收集应发送到客户端的数据，
而第二个方法处理该数据。如果你的 `BlockEntity` 包含的数据不多，你可能可以直接使用[在 `BlockEntity` 中存储数据][storing-data]章节中的方法。

!!! important
    为方块实体同步过多/无用的数据会导致网络拥塞。你应该优化网络使用，只发送客户端需要的信息，并在客户端需要时发送。例如，在更新标签中发送方块实体的物品栏通常是不必要的，因为这可以通过其 [`AbstractContainerMenu`][menu] 进行同步。

### 在方块更新时同步

这种方法稍微复杂一些，但你仍然只需要重写两个或三个方法。
以下是一个简单的实现示例：
```java
@Override
public CompoundTag getUpdateTag() {
  CompoundTag tag = new CompoundTag();
  // 将你的数据写入标签
  return tag;
}

@Override
public Packet<ClientGamePacketListener> getUpdatePacket() {
  // 将从 #getUpdateTag 获取标签
  return ClientboundBlockEntityDataPacket.create(this);
}

// 可以重写 IForgeBlockEntity#onDataPacket。默认情况下，这将委托给 #load 方法。
```
静态构造方法 `ClientboundBlockEntityDataPacket#create` 接受：

* `BlockEntity`。
* 一个可选函数，用于从 `BlockEntity` 获取 `CompoundTag`。默认情况下，使用 `BlockEntity#getUpdateTag`。

现在，要发送数据包，必须在服务器端发送更新通知。
```java
Level#sendBlockUpdated(BlockPos pos, BlockState oldState, BlockState newState, int flags)
```
`pos` 应为你的 `BlockEntity` 的位置。
对于 `oldState` 和 `newState`，你可以传入该位置的当前 `BlockState`。
`flags` 是一个位掩码，应包含 `2`，这将把更改同步到客户端。更多信息以及其余标志请参见 `Block`。标志 `2` 等同于 `Block#UPDATE_CLIENTS`。

### 使用自定义网络消息同步

这种同步方式可能是最复杂的，但通常是最优化的，
因为你可以确保只有需要同步的数据才会被实际同步。
在尝试此方法之前，你应该先查看[`网络`][networking]章节，特别是 [`SimpleImpl`][simple_impl]。
一旦你创建了自定义网络消息，就可以使用 `SimpleChannel#send(PacketDistributor$PacketTarget, MSG)` 将其发送给所有加载了该 `BlockEntity` 的玩家。

!!! warning
    务必进行安全检查，当消息到达玩家时，`BlockEntity` 可能已经被销毁/替换！你还应检查区块是否已加载（`Level#hasChunkAt(BlockPos)`）。

[registration]: ../concepts/registries.md#methods-for-registering
[storing-data]: #storing-data-within-your-blockentity
[menu]: ../gui/menus.md
[networking]: ../networking/index.md
[simple_impl]: ../networking/simpleimpl.md
