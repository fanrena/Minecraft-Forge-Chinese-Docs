方块实体渲染器
==================

`BlockEntityRenderer`（方块实体渲染器，简称 `BER`）用于渲染无法用静态烘焙模型（JSON、OBJ、B3D 等）表示的方块。方块实体渲染器要求方块拥有 `BlockEntity`。

创建 BER
--------------

要创建 BER，请创建一个继承自 `BlockEntityRenderer` 的类。它接受一个泛型参数来指定方块的 `BlockEntity` 类。该泛型参数在 BER 的 `render` 方法中使用。

对于给定的 `BlockEntityType`，只存在一个 BER。因此，特定于世界中单个实例的值应存储在传递给渲染器的方块实体中，而不是存储在 BER 本身中。例如，一个每帧递增的整数，如果存储在 BER 中，将会为世界中此类型的所有方块实体每帧递增。

### `render`

此方法每帧被调用，用于渲染方块实体。

#### 参数
* `blockEntity`：正在被渲染的方块实体实例。
* `partialTick`：自上一个完整 tick 以来经过的时间，以 tick 的分数表示。
* `poseStack`：一个持有四维矩阵条目的堆栈，偏移到方块实体的当前位置。
* `bufferSource`：能够访问顶点消费者的渲染缓冲区。
* `combinedLight`：方块实体上当前光照值的整数。
* `combinedOverlay`：设置为方块实体当前叠加层的整数，通常为 `OverlayTexture#NO_OVERLAY` 或 655,360。

注册 BER
-----------------

为了注册 BER，你必须在模组事件总线上订阅 `EntityRenderersEvent$RegisterRenderers` 事件并调用 `#registerBlockEntityRenderer`。
