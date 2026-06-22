事件
======

Forge 使用事件总线（Event Bus）让模组能够拦截原版和各种模组行为中的事件。

例如：可以在右键点击原版木棍时，通过事件来执行某个操作。

大多数事件使用的主事件总线位于 `MinecraftForge#EVENT_BUS`。还有一个用于模组特定事件的事件总线，位于 `FMLJavaModLoadingContext#getModEventBus`，仅在特定情况下使用。更多关于此总线的信息见下文。

每个事件都在其中一个总线上触发：大多数事件在主 Forge 事件总线上触发，但有些事件在模组特定的事件总线上触发。

事件处理器是已注册到事件总线上的某个方法。

创建事件处理器
-------------------------

事件处理器方法接受单个参数且不返回结果。方法可以是静态的或实例方法，具体取决于实现。

事件处理器可以直接使用 `IEventBus#addListener` 注册，泛型事件（以继承 `GenericEvent<T>` 表示）则使用 `IEventBus#addGenericListener` 注册。这两种监听器添加方法都接受一个表示方法引用的消费者。泛型事件处理器还需要指定泛型的类。事件处理器必须在主模组类的构造方法中注册。

```java
// 在主模组类 ExampleMod 中

// 此事件在模组总线上
private void modEventHandler(RegisterEvent event) {
    // 在这里处理
}

// 此事件在 Forge 总线上
private static void forgeEventHandler(AttachCapabilitiesEvent<Entity> event) {
    // ...
}

// 在模组构造方法中
modEventBus.addListener(this::modEventHandler);
forgeEventBus.addGenericListener(Entity.class, ExampleMod::forgeEventHandler);
```

### 实例注解事件处理器

此事件处理器监听 `EntityItemPickupEvent`，顾名思义，当某个 `Entity` 拾起物品时会发布到事件总线。

```java
public class MyForgeEventHandler {
    @SubscribeEvent
    public void pickupItem(EntityItemPickupEvent event) {
        System.out.println("物品被拾取！");
    }
}
```

要注册此事件处理器，使用 `MinecraftForge.EVENT_BUS.register(...)` 并传入事件处理器所在类的实例。如果要注册到模组特定的事件总线，应使用 `FMLJavaModLoadingContext.get().getModEventBus().register(...)`。

### 静态注解事件处理器

事件处理器也可以是静态的。处理方法仍然使用 `@SubscribeEvent` 注解。与实例处理器的唯一区别在于它被标记为 `static`。要注册静态事件处理器，传入类的实例是不够的，必须传入 `Class` 本身。例如：

```java
public class MyStaticForgeEventHandler {
    @SubscribeEvent
    public static void arrowNocked(ArrowNockEvent event) {
        System.out.println("搭箭！");
    }
}
```

必须这样注册：`MinecraftForge.EVENT_BUS.register(MyStaticForgeEventHandler.class)`。

### 自动注册静态事件处理器

类可以使用 `@Mod$EventBusSubscriber` 注解进行标注。这样的类会在 `@Mod` 类本身构造完成后自动注册到 `MinecraftForge#EVENT_BUS`。这本质上相当于在 `@Mod` 类构造方法的末尾添加了 `MinecraftForge.EVENT_BUS.register(AnnotatedClass.class);`。

可以向 `@Mod$EventBusSubscriber` 注解传入你想要监听的总线。建议同时指定模组 ID（因为注解处理过程可能无法自动推断）和你注册到的总线（作为提醒，确保你在正确的总线上）。你还可以指定 `Dist` 或物理端（physical sides）来加载此事件订阅器。这可以用于不在专用服务器上加载客户端特有的事件订阅器。

以下是一个监听 `RenderLevelStageEvent` 的静态事件监听器示例，它仅在客户端被调用：

```java
@Mod.EventBusSubscriber(modid = "mymod", bus = Bus.FORGE, value = Dist.CLIENT)
public class MyStaticClientOnlyEventHandler {
    @SubscribeEvent
    public static void drawLast(RenderLevelStageEvent event) {
        System.out.println("绘制中！");
    }
}
```

!!! note
    这不会注册类的实例，而是注册类本身（即事件处理方法必须是静态的）。

取消事件
---------

如果一个事件可以被取消，它会被标记为 `@Cancelable` 注解，并且 `Event#isCancelable()` 方法会返回 `true`。可取消事件的取消状态可以通过调用 `Event#setCanceled(boolean canceled)` 来修改，传入 `true` 表示取消事件，传入 `false` 表示"取消取消"事件。但是，如果事件不可取消（由 `Event#isCancelable()` 定义），无论传入什么布尔值，都会抛出 `UnsupportedOperationException`，因为不可取消事件的取消状态被视为不可变的。

!!! important
    并非所有事件都可以被取消！尝试取消一个不可取消的事件将导致抛出未检查的 `UnsupportedOperationException`，这预计会导致游戏崩溃！在尝试取消事件之前，始终使用 `Event#isCancelable()` 检查事件是否可取消！

结果
-------

有些事件具有 `Event$Result`。结果可以是以下三种之一：`DENY` 阻止事件，`DEFAULT` 使用原版行为，`ALLOW` 强制采取行动，无论原本是否会发生。事件的结果可以通过在事件上调用 `#setResult` 并传入 `Event$Result` 来设置。并非所有事件都有结果；有结果的事件会使用 `@HasResult` 注解标注。

!!! important
    不同的事件可能以不同方式使用结果，在使用结果前请参考事件的 JavaDoc。

优先级
--------

事件处理器方法（使用 `@SubscribeEvent` 标记）具有优先级。可以通过设置注解的 `priority` 值来设定事件处理器方法的优先级。优先级可以是 `EventPriority` 枚举中的任意值（`HIGHEST`、`HIGH`、`NORMAL`、`LOW` 和 `LOWEST`）。优先级为 `HIGHEST` 的事件处理器最先执行，然后按降序执行，直到最后执行 `LOWEST` 事件。

子事件
----------

许多事件具有不同的变体。这些变体可能不同，但都基于一个共同因素（例如 `PlayerEvent`），或者可能是具有多个阶段的事件（例如 `PotionBrewEvent`）。请注意，如果你监听父事件类，你的方法将为**所有**子类接收调用。

模组事件总线
-------------

模组事件总线主要用于监听模组应进行初始化的生命周期事件。模组总线上的每个事件都需要实现 `IModBusEvent`。其中许多事件也是并行运行的，这样多个模组可以同时初始化。这意味着你不能在这些事件中直接执行来自其他模组的代码。请使用 `InterModComms` 系统来实现这一点。

以下是在模组初始化期间，模组事件总线上最常用的四个生命周期事件：

* `FMLCommonSetupEvent`
* `FMLClientSetupEvent` 和 `FMLDedicatedServerSetupEvent`
* `InterModEnqueueEvent`
* `InterModProcessEvent`

!!! note
    `FMLClientSetupEvent` 和 `FMLDedicatedServerSetupEvent` 仅在各自对应的分发端（distribution）上被调用。

这四个生命周期事件都是并行运行的，因为它们都是 `ParallelDispatchEvent` 的子类。如果希望在任意 `ParallelDispatchEvent` 期间在主线程上运行代码，可以使用 `#enqueueWork`。

除了生命周期事件外，还有一些在模组事件总线上触发的杂项事件，用于注册、设置或初始化各种内容。与生命周期事件不同，这些事件大多不是并行运行的。几个例子：

* `RegisterColorHandlersEvent`
* `ModelEvent$BakingCompleted`
* `TextureStitchEvent`
* `RegisterEvent`

一个好的经验法则是：当事件应该在模组初始化期间处理时，它们在模组事件总线上触发。
