SimpleImpl
==========

SimpleImpl 是围绕 `SimpleChannel` 类的数据包系统的名称。使用此系统是在客户端和服务器之间发送自定义数据的最简单方式。

入门
---------------

首先，你需要创建你的 `SimpleChannel` 对象。我们建议在一个单独的类中执行此操作，可能类似于 `ModidPacketHandler`。将你的 `SimpleChannel` 创建为此类中的静态字段，如下所示：

```java
private static final String PROTOCOL_VERSION = "1";
public static final SimpleChannel INSTANCE = NetworkRegistry.newSimpleChannel(
  new ResourceLocation("mymodid", "main"),
  () -> PROTOCOL_VERSION,
  PROTOCOL_VERSION::equals,
  PROTOCOL_VERSION::equals
);
```

第一个参数是通道的名称。第二个参数是返回当前网络协议版本的 `Supplier<String>`。第三个和第四个参数分别是 `Predicate<String>`，用于检查传入连接协议版本是否与客户端或服务器网络兼容。

在这里，我们直接与 `PROTOCOL_VERSION` 字段进行比较，这意味着客户端和服务器的 `PROTOCOL_VERSION` 必须始终匹配，否则 FML 将拒绝登录。

版本检查器
-------------------

如果你的模组不需要另一端具有特定的网络通道，或者根本不要求另一端是 Forge 实例，你应注意正确定义版本兼容性检查器（`Predicate<String>` 参数），以处理版本检查器可能接收到的额外"元版本"（在 `NetworkRegistry` 中定义）。这些包括：

* `ABSENT` — 如果另一端缺少此通道。注意，在这种情况下，另一端仍然是 Forge 端点，并且可能具有其他模组。
* `ACCEPTVANILLA` — 如果端点是原版（或非 Forge）端点。

两者都返回 `false` 意味着此通道必须存在于另一端。如果你只是复制上面的代码，这就是它所做的。注意，这些值也在列表 ping 兼容性检查期间使用，该检查负责在多人服务器选择屏幕中显示绿色勾选或红色叉号。

注册数据包
-------------------

接下来，我们必须声明我们想要发送和接收的消息类型。这通过 `INSTANCE#registerMessage` 完成，它接受 5 个参数：

- 第一个参数是数据包的判别符（discriminator）。这是每个通道唯一的数据包 ID。我们建议使用局部变量来保存 ID，然后使用 `id++` 调用 `registerMessage`。这将保证 100% 唯一的 ID。
- 第二个参数是实际的数据包类 `MSG`。
- 第三个参数是 `BiConsumer<MSG, FriendlyByteBuf>`，负责将消息编码到提供的 `FriendlyByteBuf` 中。
- 第四个参数是 `Function<FriendlyByteBuf, MSG>`，负责从提供的 `FriendlyByteBuf` 解码消息。
- 最后一个参数是 `BiConsumer<MSG, Supplier<NetworkEvent.Context>>`，负责处理消息本身。

最后三个参数可以是 Java 中静态方法或实例方法的方法引用。记住，实例方法 `MSG#encode(FriendlyByteBuf)` 仍然满足 `BiConsumer<MSG, FriendlyByteBuf>`；`MSG` 只是成为隐式的第一个参数。

处理数据包
----------------

在数据包处理器中有几点需要强调。数据包处理器可以访问消息对象和网络上下文。上下文允许访问发送数据包的玩家（如果在服务器上），以及一种排队线程安全工作的方法。

```java
public static void handle(MyMessage msg, Supplier<NetworkEvent.Context> ctx) {
  ctx.get().enqueueWork(() -> {
    // 需要线程安全的工作（大多数工作）
    ServerPlayer sender = ctx.get().getSender(); // 发送此数据包的客户端
    // 执行操作
  });
  ctx.get().setPacketHandled(true);
}
```

从服务器发送到客户端的数据包应在另一个类中处理，并通过 `DistExecutor#unsafeRunWhenOn` 包装。

```java
// 在数据包类中
public static void handle(MyClientMessage msg, Supplier<NetworkEvent.Context> ctx) {
  ctx.get().enqueueWork(() ->
    DistExecutor.unsafeRunWhenOn(Dist.CLIENT, () -> () -> ClientPacketHandlerClass.handlePacket(msg, ctx))
  );
  ctx.get().setPacketHandled(true);
}

// 在 ClientPacketHandlerClass 中
public static void handlePacket(MyClientMessage msg, Supplier<NetworkEvent.Context> ctx) {
  // 执行操作
}
```

注意 `#setPacketHandled` 的存在，它用于告知网络系统数据包已成功处理完成。

!!! warning
    从 Minecraft 1.8 开始，数据包默认在网络线程上处理。

    这意味着你的处理器**不能**直接与大多数游戏对象交互。Forge 通过提供的 `NetworkEvent$Context` 提供了一种便捷方式，让你的代码在主线程上执行。只需调用 `NetworkEvent$Context#enqueueWork(Runnable)`，它将在下一个机会在主线程上调用给定的 `Runnable`。

!!! warning
    在服务器上处理数据包时要保持防御性。客户端可能会尝试通过发送意外数据来利用数据包处理。

    一个常见的问题是**任意区块生成**漏洞。这通常发生在服务器信任客户端发送的方块位置来访问方块和方块实体时。当访问未加载区域中的方块和方块实体时，服务器会从磁盘生成或加载此区域，然后立即将其写入磁盘。这可以被利用来对服务器的性能和存储空间造成**灾难性损害**而不留痕迹。

    为避免此问题，一个通用的经验法则是仅在 `Level#hasChunkAt` 为 `true` 时访问方块和方块实体。

发送数据包
---------------

### 发送到服务器

只有一种方式可以向服务器发送数据包。这是因为客户端一次只能连接*一个*服务器。为此，我们必须再次使用之前定义的 `SimpleChannel`。只需调用 `INSTANCE.sendToServer(new MyMessage())`。如果存在该类型的处理器，消息将被发送到该处理器。

### 发送到客户端

可以使用 `SimpleChannel` 直接将数据包发送到客户端：`HANDLER.sendTo(new MyClientMessage(), serverPlayer.connection.getConnection(), NetworkDirection.PLAY_TO_CLIENT)`。然而，这可能相当不方便。Forge 提供了一些便利函数：

```java
// 发送给一个玩家
INSTANCE.send(PacketDistributor.PLAYER.with(serverPlayer), new MyMessage());

// 发送给所有追踪此区块的玩家
INSTANCE.send(PacketDistributor.TRACKING_CHUNK.with(levelChunk), new MyMessage());

// 发送给所有连接的玩家
INSTANCE.send(PacketDistributor.ALL.noArg(), new MyMessage());
```

还有其他可用的 `PacketDistributor` 类型；请查看 `PacketDistributor` 类的文档以获取更多详细信息。
