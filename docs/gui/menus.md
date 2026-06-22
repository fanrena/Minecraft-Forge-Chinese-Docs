# 菜单

菜单是图形用户界面（GUI）的一种后端类型；它们处理与某个表示数据持有者交互的逻辑。菜单本身不是数据持有者，而是允许用户间接修改内部数据持有者状态的视图。因此，数据持有者不应与任何菜单直接耦合，而是应传入数据引用以进行调用和修改。

## `MenuType`

菜单是动态创建和移除的，因此它们本身不是注册表对象。相应地，会注册另一个工厂对象来轻松创建和引用菜单的*类型*。对于菜单，这些被称为 `MenuType`。

`MenuType` 必须[注册][registered]。

### `MenuSupplier`

`MenuType` 通过向其构造函数传入 `MenuSupplier` 和 `FeatureFlagSet` 来创建。`MenuSupplier` 是一个函数，接受容器的 ID 和查看菜单的玩家的物品栏，并返回一个新创建的 [`AbstractContainerMenu`][acm]。

```java
// 对于某个 DeferredRegister<MenuType<?>> REGISTER
public static final RegistryObject<MenuType<MyMenu>> MY_MENU = REGISTER.register("my_menu", () -> new MenuType(MyMenu::new, FeatureFlags.DEFAULT_FLAGS));

// 在 MyMenu 中，一个 AbstractContainerMenu 的子类
public MyMenu(int containerId, Inventory playerInv) {
  super(MY_MENU.get(), containerId);
  // ...
}
```

!!! note
    容器标识符对于单个玩家是唯一的。这意味着在两个不同玩家上的相同容器 ID 将代表两个不同的菜单，即使他们正在查看相同的数据持有者。

`MenuSupplier` 通常负责在客户端上创建菜单，使用虚拟数据引用来存储和与来自服务器数据持有者的同步信息进行交互。

### `IContainerFactory`

如果客户端上需要额外信息（例如数据持有者在世界中的位置），则可以使用子类 `IContainerFactory`。除了容器 ID 和玩家物品栏之外，它还提供了一个 `FriendlyByteBuf`，可以存储从服务器发送的额外信息。可以通过 `IForgeMenuType#create` 使用 `IContainerFactory` 创建 `MenuType`。

```java
// 对于某个 DeferredRegister<MenuType<?>> REGISTER
public static final RegistryObject<MenuType<MyMenuExtra>> MY_MENU_EXTRA = REGISTER.register("my_menu_extra", () -> IForgeMenuType.create(MyMenu::new));

// 在 MyMenuExtra 中，一个 AbstractContainerMenu 的子类
public MyMenuExtra(int containerId, Inventory playerInv, FriendlyByteBuf extraData) {
  super(MY_MENU_EXTRA.get(), containerId);
  // 从缓冲区存储额外数据
  // ...
}
```

## `AbstractContainerMenu`

所有菜单都继承自 `AbstractContainerMenu`。菜单接受两个参数：[`MenuType`][mt]（表示菜单本身的类型）和容器 ID（表示当前访问者的菜单唯一标识符）。

!!! important
    玩家一次最多只能打开 100 个唯一的菜单。

每个菜单应包含两个构造函数：一个用于在服务器上初始化菜单，另一个用于在客户端上初始化菜单。用于在客户端上初始化菜单的构造函数是提供给 `MenuType` 的那个。服务器菜单构造函数包含的任何字段都应在客户端菜单构造函数中具有一些默认值。

```java
// 客户端菜单构造函数
public MyMenu(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory);
}

// 服务器菜单构造函数
public MyMenu(int containerId, Inventory playerInventory) {
  // ...
}
```

每个菜单实现必须实现两个方法：`#stillValid` 和 [`#quickMoveStack`][qms]。

### `#stillValid` 和 `ContainerLevelAccess`

`#stillValid` 确定菜单是否应对给定玩家保持打开状态。通常这指向静态的 `#stillValid`，它接受一个 `ContainerLevelAccess`、玩家和此菜单所附加的 `Block`。客户端菜单必须始终返回 `true`，而静态 `#stillValid` 默认也是如此。此实现检查玩家是否在数据存储对象所在位置的 8 个方块范围内。

`ContainerLevelAccess` 在封闭的作用域内提供当前世界和方块的位置。在服务器上构建菜单时，可以通过调用 `ContainerLevelAccess#create` 创建新的访问对象。客户端菜单构造函数可以传入 `ContainerLevelAccess#NULL`，它不会执行任何操作。

```java
// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory, ContainerLevelAccess.NULL);
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, ContainerLevelAccess access) {
  // ...
}

// 假设此菜单附加到 RegistryObject<Block> MY_BLOCK
@Override
public boolean stillValid(Player player) {
  return AbstractContainerMenu.stillValid(this.access, player, MY_BLOCK.get());
}
```

### 数据同步

某些数据需要在服务器和客户端上都存在以显示给玩家。为此，菜单实现了一个基本的数据同步层，每当当前数据与上次同步到客户端的数据不匹配时进行同步。对于玩家，每 tick 检查一次。

Minecraft 默认支持两种形式的数据同步：通过 `Slot` 的 `ItemStack` 和通过 `DataSlot` 的整数。`Slot` 和 `DataSlot` 是持有对数据存储引用的视图，玩家可以在屏幕中修改它们（假设操作有效）。这些可以在构造函数中通过 `#addSlot` 和 `#addDataSlot` 添加到菜单中。

!!! note
    由于 `Slot` 使用的 `Container` 已被 Forge 废弃，推荐使用 [`IItemHandler` 能力][cap]，因此其余的说明将围绕使用能力变体：`SlotItemHandler`。

`SlotItemHandler` 包含四个参数：表示栈所在物品栏的 `IItemHandler`、此槽位具体代表的栈的索引，以及槽位左上角在屏幕上相对于 `AbstractContainerScreen#leftPos` 和 `#topPos` 渲染的 x 和 y 位置。客户端菜单构造函数应始终提供相同大小的空物品栏实例。

在大多数情况下，先添加菜单包含的所有槽位，然后是玩家物品栏，最后以玩家快捷栏结尾。要从菜单访问任何单个 `Slot`，必须根据添加槽位的顺序计算索引。

`DataSlot` 是一个抽象类，应实现 getter 和 setter 来引用存储在数据存储对象中的数据。客户端菜单构造函数应始终通过 `DataSlot#standalone` 提供一个新的实例。

这些与槽位一起，应在每次初始化新菜单时重新创建。

!!! warning
    尽管 `DataSlot` 存储一个整数，但由于其通过网络发送值的方式，实际上限制为 **short**（-32768 到 32767）。整数的高 16 位被忽略。

```java
// 假设我们有一个大小为 5 的数据对象物品栏
// 假设我们在每次初始化服务器菜单时构造了一个 DataSlot

// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory, new ItemStackHandler(5), DataSlot.standalone());
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, IItemHandler dataInventory, DataSlot dataSingle) {
  // 检查数据物品栏大小是否为某个固定值
  // 然后，为数据物品栏添加槽位
  this.addSlot(new SlotItemHandler(dataInventory, /*...*/));

  // 为玩家物品栏添加槽位
  this.addSlot(new Slot(playerInventory, /*...*/));

  // 为处理的整数添加数据槽位
  this.addDataSlot(dataSingle);

  // ...
}
```

#### `ContainerData`

如果需要将多个整数同步到客户端，可以使用 `ContainerData` 来引用这些整数。此接口作为一个索引查找表，每个索引代表一个不同的整数。`ContainerData` 也可以在数据对象本身中构造，如果 `ContainerData` 通过 `#addDataSlots` 添加到菜单中。该方法为接口指定的数据量为每个数据创建一个新的 `DataSlot`。客户端菜单构造函数应始终通过 `SimpleContainerData` 提供一个新的实例。

```java
// 假设我们有一个大小为 3 的 ContainerData

// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory, new SimpleContainerData(3));
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, ContainerData dataMultiple) {
  // 检查 ContainerData 大小是否为某个固定值
  checkContainerDataCount(dataMultiple, 3);

  // 为处理的整数添加数据槽位
  this.addDataSlots(dataMultiple);

  // ...
}
```

!!! warning
    由于 `ContainerData` 委托给 `DataSlot`，因此也限制为 **short**（-32768 到 32767）。

#### `#quickMoveStack`

`#quickMoveStack` 是每个菜单必须实现的第二个方法。每当堆叠的物品被 Shift 点击（即快速移动）出其当前槽位时，就会调用此方法，直到堆叠的物品完全移出其之前的槽位或没有其他地方可去为止。该方法返回被快速移动的槽位中堆叠物品的副本。

堆叠的物品通常使用 `#moveItemStackTo` 在槽位之间移动，该方法将堆叠物品移动到第一个可用的槽位。它接受要移动的堆叠物品、第一个槽位索引（包含）、最后一个槽位索引（不包含）以及是否从第一个到最后一个检查槽位（`false`）或从最后一个到第一个检查槽位（`true`）。

在 Minecraft 的实现中，此方法的逻辑相当一致：

```java
// 假设我们有一个大小为 5 的数据物品栏
// 该物品栏有 4 个输入槽（索引 1-4），输出到结果槽（索引 0）
// 我们还有 27 个玩家物品栏槽位和 9 个快捷栏槽位
// 因此，实际槽位的索引如下：
//   - 数据物品栏：结果（0），输入（1-4）
//   - 玩家物品栏（5-31）
//   - 玩家快捷栏（32-40）
@Override
public ItemStack quickMoveStack(Player player, int quickMovedSlotIndex) {
  // 快速移动的槽位堆叠
  ItemStack quickMovedStack = ItemStack.EMPTY;
  // 快速移动的槽位
  Slot quickMovedSlot = this.slots.get(quickMovedSlotIndex) 
  
   // 如果槽位在有效范围内且不为空
  if (quickMovedSlot != null && quickMovedSlot.hasItem()) {
    // 获取要移动的原始堆叠
    ItemStack rawStack = quickMovedSlot.getItem(); 
    // 将槽位堆叠设置为原始堆叠的副本
    quickMovedStack = rawStack.copy();

    // 如果快速移动是在数据物品栏结果槽位上执行的
    if (quickMovedSlotIndex == 0) {
      // 尝试将结果槽移动到玩家物品栏/快捷栏
      if (!this.moveItemStackTo(rawStack, 5, 41, true)) {
        return ItemStack.EMPTY;
      }
      slot.onQuickCraft(rawStack, quickMovedStack);
    }
    // 否则如果快速移动是在玩家物品栏或快捷栏槽位上执行的
    else if (quickMovedSlotIndex >= 5 && quickMovedSlotIndex < 41) {
      // 尝试将物品栏/快捷栏槽位移入数据物品栏输入槽
      if (!this.moveItemStackTo(rawStack, 1, 5, false)) {
        // 如果在玩家物品栏槽位中且无法移动，尝试移动到快捷栏
        if (quickMovedSlotIndex < 32) {
          if (!this.moveItemStackTo(rawStack, 32, 41, false)) {
            return ItemStack.EMPTY;
          }
        }
        // 否则尝试将快捷栏移动到玩家物品栏槽位
        else if (!this.moveItemStackTo(rawStack, 5, 32, false)) {
          return ItemStack.EMPTY;
        }
      }
    }
    // 否则如果快速移动是在数据物品栏输入槽上执行的，尝试移动到玩家物品栏/快捷栏
    else if (!this.moveItemStackTo(rawStack, 5, 41, false)) {
      return ItemStack.EMPTY;
    }

    if (rawStack.isEmpty()) {
      quickMovedSlot.set(ItemStack.EMPTY);
    } else {
      quickMovedSlot.setChanged();
    }

    if (rawStack.getCount() == quickMovedStack.getCount()) {
      return ItemStack.EMPTY;
    }
    quickMovedSlot.onTake(player, rawStack);
  }

  return quickMovedStack;
}
```

## 打开菜单

一旦菜单类型已注册、菜单本身已完成并且已附加了[屏幕][screen]，玩家就可以打开菜单。可以通过在逻辑服务器上调用 `NetworkHooks#openScreen` 来打开菜单。该方法接受打开菜单的玩家、服务端菜单的 `MenuProvider`，以及可选的 `FriendlyByteBuf`（如果需要将额外数据同步到客户端）。

!!! note
    只有在使用 [`IContainerFactory`][icf] 创建菜单类型时，才应使用带有 `FriendlyByteBuf` 参数的 `NetworkHooks#openScreen`。

#### `MenuProvider`

`MenuProvider` 是一个包含两个方法的接口：`#createMenu`（创建菜单的服务器实例）和 `#getDisplayName`（返回包含菜单标题的组件，传递给[屏幕][screen]）。`#createMenu` 方法包含三个参数：菜单的容器 ID、打开菜单的玩家的物品栏以及打开菜单的玩家。

可以使用 `SimpleMenuProvider` 轻松创建 `MenuProvider`，它接受一个创建服务器菜单的方法引用和菜单的标题。

```java
// 在某些实现中
NetworkHooks.openScreen(serverPlayer, new SimpleMenuProvider(
  (containerId, playerInventory, player) -> new MyMenu(containerId, playerInventory),
  Component.translatable("menu.title.examplemod.mymenu")
));
```

### 常见实现

菜单通常在某种玩家交互时打开（例如，右键点击方块或实体时）。

#### 方块实现

方块通常通过重写 `BlockBehaviour#use` 来实现菜单。如果在逻辑客户端上，交互返回 `InteractionResult#SUCCESS`。否则，它打开菜单并返回 `InteractionResult#CONSUME`。

`MenuProvider` 应通过重写 `BlockBehaviour#getMenuProvider` 来实现。原版方法使用它来在旁观者模式下查看菜单。

```java
// 在某些 Block 子类中
@Override
public MenuProvider getMenuProvider(BlockState state, Level level, BlockPos pos) {
  return new SimpleMenuProvider(/* ... */);
}

@Override
public InteractionResult use(BlockState state, Level level, BlockPos pos, Player player, InteractionHand hand, BlockHitResult result) {
  if (!level.isClientSide && player instanceof ServerPlayer serverPlayer) {
    NetworkHooks.openScreen(serverPlayer, state.getMenuProvider(level, pos));
  }
  return InteractionResult.sidedSuccess(level.isClientSide);
}
```

!!! note
    这是实现此逻辑的最简单方式，但不是唯一方式。如果你希望方块仅在特定条件下打开菜单，则需要事先将某些数据同步到客户端，以便在条件不满足时返回 `InteractionResult#PASS` 或 `#FAIL`。

#### 生物实现

生物通常通过重写 `Mob#mobInteract` 来实现菜单。这与方块实现类似，唯一的区别在于 `Mob` 本身应实现 `MenuProvider` 以支持旁观者模式查看。

```java
public class MyMob extends Mob implements MenuProvider {
  // ...

  @Override
  public InteractionResult mobInteract(Player player, InteractionHand hand) {
    if (!this.level.isClientSide && player instanceof ServerPlayer serverPlayer) {
      NetworkHooks.openScreen(serverPlayer, this);
    }
    return InteractionResult.sidedSuccess(this.level.isClientSide);
  }
}
```

!!! note
    同样，这是实现此逻辑的最简单方式，但不是唯一方式。

[registered]: ../concepts/registries.md#注册方法
[acm]: #abstractcontainermenu
[mt]: #menutype
[qms]: #quickmovestack
[cap]: ../datastorage/capabilities.md#forge-提供的能力
[screen]: ./screens.md
[icf]: #icontainerfactory
