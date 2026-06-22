渲染类型
============

在 JSON 顶层添加 `render_type` 条目，告知加载器模型应使用的渲染类型。如果未指定，加载器会选择使用的渲染类型，通常回退到 `ItemBlockRenderTypes#getRenderLayers()` 返回的渲染类型。

自定义模型加载器可以完全忽略此字段。

!!! note
    自 1.19 起，对于方块，这优先于通过 `ItemBlockRenderTypes#setRenderLayer()` 设置适用渲染类型的已弃用方法。

使用玻璃纹理的 cutout 方块模型示例：

```js
{
  "render_type": "minecraft:cutout",
  "parent": "block/cube_all",
  "textures": {
    "all": "block/glass"
  }
}
```

原版值
--------------

以下是 Forge 提供的具有相应区块和实体渲染类型的选项：

* `minecraft:solid` — 完全实心（石头）
* `minecraft:cutout` — 完全透明或不透明（玻璃）
* `minecraft:cutout_mipped` — 带 mipmapping 的 cutout（树叶）
* `minecraft:cutout_mipped_all` — 物品也应用 mipmapping 的 cutout
* `minecraft:translucent` — 半透明（染色玻璃）
* `minecraft:tripwire` — 需要天气渲染目标的特殊类型（绊线钩）

自定义值
-------------

自定义命名渲染类型可以在 `RegisterNamedRenderTypesEvent`（模组事件总线）中注册。由区块渲染类型 + 实体渲染类型（必需）+ 实体渲染类型 *Fabulous!* 模式（可选）组成。

```java
event.register("special_cutout", RenderType.cutout(), Sheets.cutoutBlockSheet());
event.register("special_translucent", RenderType.translucent(), Sheets.translucentCullBlockSheet(), Sheets.translucentItemSheet());
```

在 JSON 中引用为 `<your_mod_id>:special_cutout`。

[mipmapping]: https://en.wikipedia.org/wiki/Mipmap
