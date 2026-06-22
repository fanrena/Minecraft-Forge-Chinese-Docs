`BakedModel`
=============

`BakedModel` 是调用 `UnbakedModel#bake`（原版）或 `IUnbakedGeometry#bake`（自定义）的结果。代表经过优化、准备发送到 GPU 的几何体。

核心方法：

- **`getQuads`** — 主要方法。返回 `BakedQuad` 列表（包含底层顶点数据）。作为方块渲染时 `BlockState` 非 null；作为物品渲染时 `BlockState` 为 null。`Direction` 用于面剔除。此方法调用极频繁，应大量使用缓存。
- **`getOverrides`** — 返回 `ItemOverrides`，仅物品渲染时使用。
- **`useAmbientOcclusion`** — 是否使用环境光遮蔽。
- **`isGui3d`** — 物品渲染时是否显示为"立体"。
- **`isCustomRenderer`** — 返回 true 时回退到 `BlockEntityWithoutLevelRenderer` 渲染。
- **`getParticleIcon`** — 粒子纹理。
- **`applyTransform`** — 处理物品显示变换（替代已弃用的 `getTransforms`）。

!!! note
    `BakedQuad` 顶点原点为西北角底部，建议按逆时针顺序提供。

[overrides]: ./itemoverrides.md
[bewlr]: ../../items/bewlr.md
[transform]: ./transform.md
