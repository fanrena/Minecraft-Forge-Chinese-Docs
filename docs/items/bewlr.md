BlockEntityWithoutLevelRenderer
=======================
`BlockEntityWithoutLevelRenderer`（BEWLR）是一种处理物品动态渲染的方法。该系统比旧的 `ItemStack` 系统简单得多，旧系统需要 `BlockEntity`，并且不允许访问 `ItemStack`。

使用 BlockEntityWithoutLevelRenderer
--------------------------

BlockEntityWithoutLevelRenderer 允许你使用 `public void renderByItem(ItemStack itemStack, ItemDisplayContext ctx, PoseStack poseStack, MultiBufferSource bufferSource, int combinedLight, int combinedOverlay)` 来渲染物品。

要使用 BEWLR，`Item` 必须首先满足其模型对 `BakedModel#isCustomRenderer` 返回 `true` 的条件。如果没有，它将使用默认的 `ItemRenderer#getBlockEntityRenderer`。一旦返回 `true`，该物品的 BEWLR 将被用于渲染。

!!! note
    如果 `Block#getRenderShape` 设置为 `RenderShape#ENTITYBLOCK_ANIMATED`，`Block` 也会使用 BEWLR 进行渲染。

要为物品设置 BEWLR，必须在 `Item#initializeClient` 中消费 `IClientItemExtensions` 的匿名实例。在匿名实例中，应重写 `IClientItemExtensions#getCustomRenderer` 以返回你的 BEWLR 实例：

```java
// 在你的物品类中
@Override
public void initializeClient(Consumer<IClientItemExtensions> consumer) {
  consumer.accept(new IClientItemExtensions() {

    @Override
    public BlockEntityWithoutLevelRenderer getCustomRenderer() {
      return myBEWLRInstance;
    }
  });
}
```

!!! important
    每个模组应该只有一个自定义 BEWLR 实例。

就是这样，使用 BEWLR 不需要其他额外设置。
