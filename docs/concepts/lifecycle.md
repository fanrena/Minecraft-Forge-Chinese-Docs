模组生命周期
==============

在模组加载过程中，各种生命周期事件会在模组特定的事件总线上触发。许多操作在这些事件期间执行，例如[注册对象][registering]、准备[数据生成][datagen]或[与其他模组通信][imc]。

事件监听器应使用 `@EventBusSubscriber(bus = Bus.MOD)` 或在模组构造方法中注册：

```Java
@Mod.EventBusSubscriber(modid = "mymod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class MyModEventSubscriber {
  @SubscribeEvent
  static void onCommonSetup(FMLCommonSetupEvent event) { ... }
}

@Mod("mymod")
public class MyMod {
  public MyMod() {
    FMLModLoadingContext.get().getModEventBus().addListener(this::onCommonSetup);
  } 

  private void onCommonSetup(FMLCommonSetupEvent event) { ... }
}
```

!!! warning
    大多数生命周期事件是并行触发的：所有模组将同时接收到相同的事件。

    模组**必须**注意线程安全，例如在调用其他模组的 API 或访问原版系统时。通过 `ParallelDispatchEvent#enqueueWork` 将代码推迟到以后执行。

注册事件
---------------

注册事件在模组实例构造之后触发。共有三个：`NewRegistryEvent`、`DataPackRegistryEvent$NewRegistry` 和 `RegisterEvent`。这些事件在模组加载期间同步触发。

`NewRegistryEvent` 允许模组开发者使用 `RegistryBuilder` 类注册自定义注册表。

`DataPackRegistryEvent$NewRegistry` 允许模组开发者通过提供 `Codec` 来编解码 JSON 中的对象，从而注册自定义数据包注册表。

`RegisterEvent` 用于[注册对象][registering]到注册表中。该事件会为每个注册表触发一次。

数据生成
---------------

如果游戏设置为运行[数据生成器][datagen]，那么 `GatherDataEvent` 将是最后触发的事件。此事件用于将模组的数据提供者注册到关联的数据生成器。此事件也是同步触发的。

通用设置
------------

`FMLCommonSetupEvent` 用于物理客户端和服务器通用的操作，例如注册[能力（capabilities）][capabilities]。

端侧设置
-----------

端侧设置事件在各自的[物理端][sides]上触发：`FMLClientSetupEvent` 在物理客户端上触发，`FMLDedicatedServerSetupEvent` 在专用服务器上触发。这是进行物理端特定初始化（例如注册客户端按键绑定）的地方。

模组间通信（InterModComms）
-------------

这是用于跨模组兼容性而向模组发送消息的地方。有两个事件：`InterModEnqueueEvent` 和 `InterModProcessEvent`。

`InterModComms` 是负责保存模组消息的类。这些方法在生命周期事件期间调用是安全的，因为它由 `ConcurrentMap` 支持。

在 `InterModEnqueueEvent` 期间，使用 `InterModComms#sendTo` 向不同模组发送消息。这些方法接受将接收消息的模组 ID、消息数据关联的键以及包含消息数据的供应者（supplier）。此外，也可以指定消息的发送者，但默认情况下会是调用者的模组 ID。

然后在 `InterModProcessEvent` 期间，使用 `InterModComms#getMessages` 获取所有接收到的消息的流。提供的模组 ID 几乎总是调用该方法的模组的模组 ID。此外，可以指定一个谓词来过滤消息键。这将返回一个 `IMCMessage` 流，其中包含数据的发送者、数据的接收者、数据键以及提供的实际数据。

!!! note
    还有另外两个生命周期事件：`FMLConstructModEvent`，在模组实例构造之后、`RegisterEvent` 之前触发；以及 `FMLLoadCompleteEvent`，在 `InterModComms` 事件之后、模组加载过程完成时触发。

[registering]: ./registries.md#注册方法
[capabilities]: ../datastorage/capabilities.md
[datagen]: ../datagen/index.md
[imc]: ./lifecycle.md#模组间通信-intermodcomms
[sides]: ./sides.md
