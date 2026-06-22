变换
==========

当 `BakedModel` 作为物品渲染时，可以根据所处的变换上下文应用特殊处理。"变换"指模型被渲染时的上下文，由 `ItemDisplayContext` 枚举表示。

`ItemDisplayContext` 值：
- `NONE` — 无上下文（默认），Forge 用于 `ENTITYBLOCK_ANIMATED` 方块
- `THIRD_PERSON_*` / `FIRST_PERSON_*` — 第三人称/第一人称手持
- `HEAD` — 头盔槽佩戴
- `GUI` — 屏幕中渲染
- `GROUND` — 掉落物实体
- `FIXED` — 物品展示框

原版方式（已弃用）：`BakedModel#getTransforms` → `ItemTransforms` → `ItemTransform`（含旋转/平移/缩放）。

Forge 方式（推荐）：实现 `#applyTransform(ItemDisplayContext, PoseStack, boolean leftHand)` → 返回新的 `BakedModel`。更灵活，可以返回完全不同的模型（如纸在手中是平的，在地上是皱的）。

默认返回 `ItemTransforms#NO_TRANSFORMS` 并实现 `#applyTransform` 即可。

[bakedmodel]: ./bakedmodel.md
