Minecraft 中的端
===================

在 Minecraft 模组开发中，理解两个端（sides）是一个非常重要的概念：*客户端*和*服务器*。关于端存在许多常见的误解和错误，这可能导致不会使游戏崩溃，但会产生意外且常常是不可预测的后果的 Bug。

不同类型的端
------------------------

当我们说"客户端"或"服务器"时，通常对我们在谈论游戏的哪个部分有一个相当直观的理解。毕竟，客户端是用户与之交互的部分，而服务器是用户为多人游戏连接的地方。很简单，对吧？

事实证明，即使是这样两个术语也可能存在歧义。以下是对"客户端"和"服务器"的四种可能含义的澄清：

* **物理客户端** — *物理客户端*是当你从启动器启动 Minecraft 时运行的整个程序。在游戏图形化、可交互的生命周期内运行的所有线程、进程和服务都是物理客户端的一部分。
* **物理服务器** — 通常称为专用服务器（dedicated server），*物理服务器*是当你启动任何不提供可玩 GUI 的 `minecraft_server.jar` 时运行的整个程序。
* **逻辑服务器** — *逻辑服务器*运行游戏逻辑：生物生成、天气、更新物品栏、生命值、AI 以及所有其他游戏机制。逻辑服务器存在于物理服务器中，但它也可以在物理客户端内与逻辑客户端一起运行（即单人世界）。逻辑服务器总是在一个名为 `Server Thread` 的线程中运行。
* **逻辑客户端** — *逻辑客户端*接受来自玩家的输入并将其中继到逻辑服务器。此外，它还从逻辑服务器接收信息，并以图形方式呈现给玩家。逻辑客户端在 `Render Thread` 中运行，但通常还会生成其他几个线程来处理音频和区块渲染批处理等任务。

在 MinecraftForge 代码库中，物理端由名为 `Dist` 的枚举表示，而逻辑端由名为 `LogicalSide` 的枚举表示。

执行端特定操作
-----------------------------------

### `Level#isClientSide`

此布尔值检查将是你最常用的端检查方式。在 `Level` 对象上查询此字段可确定该**逻辑端**所属的端。也就是说，如果此字段为 `true`，则该世界当前在逻辑客户端上运行。如果该字段为 `false`，则该世界在逻辑服务器上运行。因此，物理服务器始终包含 `false` 值，但我们不能假设 `false` 意味着物理服务器，因为此字段在物理客户端内的逻辑服务器上也可以为 `false`（换句话说，就是单人世界）。

每当你需要确定是否应运行游戏逻辑和其他机制时，请使用此检查。例如，如果你想在每次玩家点击你的方块时对玩家造成伤害，或者让你的机器将泥土加工成钻石，你只应在确保 `#isClientSide` 为 `false` 后执行此操作。将游戏逻辑应用于逻辑客户端，最好的情况下会导致不同步（幽灵实体、不同步的统计数据等），最坏的情况下会导致崩溃。

此检查应作为你的默认首选方法。除了 `DistExecutor` 之外，你很少需要使用其他方式来确定端和调整行为。

### `DistExecutor`

考虑到为客户端和服务器模组使用单个"通用"jar 文件，以及将物理端分离为两个 jar 文件，一个重要的问题出现了：我们如何使用仅存在于一个物理端上的代码？`net.minecraft.client` 中的所有代码仅存在于物理客户端上。如果你编写的任何类以任何方式引用了这些名称，当该类在不存这些名称的环境中被加载时，游戏将会崩溃。初学者一个非常常见的错误是在方块或方块实体类中调用 `Minecraft.getInstance().<doStuff>()`，这将在类加载时立即导致任何物理服务器崩溃。

如何解决这个问题？幸运的是，FML 提供了 `DistExecutor`，它提供了各种方法，可以在不同的物理端上运行不同的方法，或者仅在某一端上运行单个方法。

!!! note
    理解 FML 是基于**物理端**进行检查非常重要。单人世界（物理客户端内的逻辑服务器 + 逻辑客户端）始终使用 `Dist.CLIENT`！

`DistExecutor` 的工作原理是接收一个执行方法的供应者，通过利用 [`invokedynamic` JVM 指令][invokedynamic] 有效防止类加载。被执行的方法应为静态方法，并且位于不同的类中。此外，如果静态方法没有参数，则应使用方法引用，而不是执行方法的供应者。

`DistExecutor` 中有两个主要方法：`#runWhenOn` 和 `#callWhenOn`。这些方法接受应执行方法的物理端以及分别运行或返回结果的供应执行方法。

这两个方法进一步细分为 `#safe*` 和 `#unsafe*` 变体。安全和 unsafe 变体对其用途来说都是误称。主要区别在于，在开发环境中，`#safe*` 变体会验证提供的执行方法是否为返回对另一个类的方法引用的 lambda，否则会抛出错误。在生产环境中，`#safe*` 和 `#unsafe*` 功能相同。

```java
// 在客户端类中：ExampleClass
public static void unsafeRunMethodExample(Object param1, Object param2) {
  // ...
}

public static Object safeCallMethodExample() {
  // ...
}

// 在某个通用类中
DistExecutor.unsafeRunWhenOn(Dist.CLIENT, () -> ExampleClass.unsafeRunMethodExample(var1, var2));

DistExecutor.safeCallWhenOn(Dist.CLIENT, () -> ExampleClass::safeCallMethodExample);

```

!!! warning
    由于 Java 9+ 中 `invokedynamic` 的工作方式发生了变化，`DistExecutor` 方法的所有 `#safe*` 变体在开发环境中都会将原始异常包装在 `BootstrapMethodError` 中抛出。应改用 `#unsafe*` 变体或检查 [`FMLEnvironment#dist`][dist]。

### 线程组

如果 `Thread.currentThread().getThreadGroup() == SidedThreadGroups.SERVER` 为 `true`，则当前线程很可能在逻辑服务器上。否则，很可能在逻辑客户端上。当无法访问 `Level` 对象来检查 `isClientSide` 时，这有助于获取**逻辑端**。它通过查看当前运行线程的组来*猜测*你所在的逻辑端。因为这是一种猜测，此方法只应在其他选项都已用尽时使用。在几乎所有情况下，应优先检查 `Level#isClientSide`。

### `FMLEnvironment#dist` 和 `@OnlyIn`

`FMLEnvironment#dist` 保存了你的代码正在运行的**物理端**。由于它是在启动时确定的，因此不依赖于猜测来返回结果。然而，它的使用场景有限。

使用 `@OnlyIn(Dist)` 注解标记方法或字段，表示加载器应完全从不在指定**物理端**上的定义中剥离该成员。通常，只有在浏览反编译的 Minecraft 代码时才会看到这些注解，表示 Mojang 混淆器剥离的方法。**没有**直接使用此注解的理由。请改用 `DistExecutor` 或检查 `FMLEnvironment#dist`。

常见错误
---------------

### 跨逻辑端访问

每当你想将信息从一个逻辑端发送到另一个逻辑端时，你**必须**始终使用网络数据包。在单人游戏场景中，直接将数据从逻辑服务器传输到逻辑客户端是非常诱人的做法。

这实际上通常通过静态字段无意中完成。由于在单人游戏场景中逻辑客户端和逻辑服务器共享同一个 JVM，两个线程同时写入和读取静态字段将导致各种竞态条件和与线程相关的经典问题。

这种错误也可能显式地发生，例如从可以在逻辑服务器上运行（或确实运行）的通用代码中访问物理客户端独有的类（如 `Minecraft`）。初学者在物理客户端中调试时很容易忽略这个错误。代码在那里可以工作，但在物理服务器上会立即崩溃。

编写单端模组
----------------------

在最近的版本中，Minecraft Forge 已从 mods.toml 中移除了"端性"属性。这意味着无论你的模组是在物理客户端还是物理服务器上加载，都应能正常工作。因此，对于单端模组，你通常应在 `DistExecutor#safeRunWhenOn` 或 `DistExecutor#unsafeRunWhenOn` 中注册事件处理器，而不是在模组构造方法中直接调用相关的注册方法。基本上，如果你的模组在错误的端上加载，它应该什么都不做，不监听任何事件，等等。单端模组本质上不应注册方块、物品等，因为它们在另一端也需要可用。

此外，如果你的模组是单端的，它通常不应阻止用户加入缺少该模组的服务器。因此，你应该将 [mods.toml][structuring] 中的 `displayTest` 属性设置为必要的值。

```toml
[[mods]]
  # ...

  # MATCH_VERSION 意味着如果你的模组在客户端和服务器上的版本不同，将显示红 X。这是默认行为，如果你的模组同时有服务器和客户端元素，应选择此选项。
  # IGNORE_SERVER_VERSION 意味着如果模组存在于服务器但不在客户端上，不会显示红 X。如果你只是服务器端模组，应使用此选项。
  # IGNORE_ALL_VERSION 意味着如果模组存在于客户端或服务器上，不会显示红 X。这是一种特殊情况，仅在你的模组没有服务器组件时应使用。
  # NONE 表示你的模组未设置显示测试。你需要自行设置，更多信息请参阅 IExtensionPoint.DisplayTest。使用此值你可以定义任何你希望的方案。
  # 重要说明：这不是关于你的模组在哪些环境（CLIENT 或 DEDICATED SERVER）中加载的指示。你的模组应该在它所在的任何地方加载（并且可能什么都不做！）。
  displayTest="IGNORE_ALL_VERSION" # 如果未指定，MATCH_VERSION 是默认值（可选）
```

如果要使用自定义显示测试，则应将 `displayTest` 选项设置为 `NONE`，并注册 `IExtensionPoint$DisplayTest` 扩展：

```java
// 确保模组在另一方网络侧缺失时，不会导致客户端显示服务器不兼容
ModLoadingContext.get().registerExtensionPoint(IExtensionPoint.DisplayTest.class, () -> new IExtensionPoint.DisplayTest(() -> NetworkConstants.IGNORESERVERONLY, (a, b) -> true));
```

这告诉客户端，它应忽略服务器版本缺失，并告诉服务器，它不应告诉客户端此模组应存在。因此，此代码片段同时适用于仅客户端和仅服务器端的模组。

[invokedynamic]: https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-6.html#jvms-6.5.invokedynamic
[dist]: #fmlenvironmentdist-和-onlyin
[structuring]: ../gettingstarted/modfiles.md#modstoml
