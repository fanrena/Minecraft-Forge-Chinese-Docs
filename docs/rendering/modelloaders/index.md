自定义模型加载器
====================

"模型"是一个形状，由 `IGeometryLoader` 在运行时加载为 `IUnbakedGeometry`，最终烘焙为 `BakedModel`。

Forge 提供的内置加载器：

- **WaveFront OBJ** (`forge:obj`) — 加载 `.obj` 文件，支持 `flip_v` 翻转 V 轴
- **桶模型** — 原版桶的专用加载器
- **复合模型** (`forge:composite`) — 组合多个子模型
- **不同渲染层** — 使用 `render_type` 指定
- **`builtin/generated` 重实现** — 原版物品模型生成器的 Forge 版本

!!! warning
    指定 `loader` 后，`elements` 条目将被忽略（除非被自定义加载器消费）。

WaveFront OBJ 模型示例：
```js
{
  "loader": "forge:obj",
  "flip_v": true,
  "model": "examplemod:models/block/model.obj",
  "textures": { "texture0": "minecraft:block/dirt", "particle": "minecraft:block/dirt" }
}
```
