能力系统
=====================

Capability（能力）系统允许以动态灵活的方式暴露功能，无需直接实现大量接口。

Forge 提供的能力：
- `IItemHandler` — 物品栏处理（替代旧的 `Container`/`WorldlyContainer`）
- `IFluidHandler` — 流体处理
- `IEnergyStorage` — 能量处理（基于 RedstoneFlux API）

### 使用能力
通过 `#getCapability(cap, direction)` 查询。返回 `LazyOptional<T>`。
- `cap`：能力实例，通过 `CapabilityManager.get(new CapabilityToken<>(){})` 获取
- `direction`：方向，`null` 表示无方向要求
- 使用前检查 `Capability#isRegistered`

### 暴露能力
1. 创建能力接口的实例（如 `ItemStackHandler`）
2. 重写 `#getCapability`，比较能力实例并返回对应 `LazyOptional`
3. 在 `#invalidateCaps` 或 `AttachCapabilitiesEvent#addListener` 中失效 `LazyOptional`

```java
@Override
public <T> LazyOptional<T> getCapability(Capability<T> cap, Direction side) {
  if (cap == ForgeCapabilities.ITEM_HANDLER) {
    return inventoryHandlerLazyOptional.cast();
  }
  return super.getCapability(cap, side);
}
```

### 附加能力（给非自有对象）
使用 `AttachCapabilitiesEvent`（在 Forge 总线上触发）为 Entity、BlockEntity、ItemStack、Level、LevelChunk 附加能力。实现 `ICapabilitySerializable` 以支持持久化。

### 注册自定义能力
- `RegisterCapabilitiesEvent`（模组事件总线）：`event.register(IExampleCapability.class)`
- `@AutoRegisterCapability`：注解在能力接口上

### 数据同步
能力数据默认不发送到客户端。需要通过自定义网络数据包实现同步。

### 玩家死亡持久化
在 `PlayerEvent$Clone` 中手动复制数据，通过 `#isWasDeath` 区分死亡重生和从末地返回。

[network]: ../networking/index.md
