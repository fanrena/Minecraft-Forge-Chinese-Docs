纹理着色
=================

许多方块和物品的纹理颜色会根据位置或属性变化（如草）。模型支持在面上指定"着色索引"（tint index），由 `BlockColor` 和 `ItemColor` 处理。

### BlockColor / ItemColor
均为单方法接口，接受 `tintIndex` 参数，返回颜色乘数（int，按 ARGB 格式打包）。
每个像素的计算：`(int)(base * multiplier / 255.0)`

### 注册颜色处理器
- `RegisterColorHandlersEvent$Block`（模组事件总线）：`event.register(myBlockColor, block1, block2...)`
- `RegisterColorHandlersEvent$Item`（模组事件总线）：`event.register(myItemColor, item1, item2...)`

注意：注册 `BlockColor` 不会自动为 `BlockItem` 着色，需要单独注册 `ItemColor`。

### 内置模型
物品使用 `builtin/generated` 模型时，每层（layer0, layer1...）的着色索引对应其层索引。

[wiki]: https://minecraft.wiki/w/Tutorials/Models#Block_models
