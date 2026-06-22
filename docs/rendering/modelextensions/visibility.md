部件可见性
===============

在模型 JSON 顶层添加 `visibility` 条目，控制模型不同部分的可见性，决定它们是否应被烘焙到最终的 `BakedModel` 中。"部件"定义取决于模型加载器。Forge 的[复合模型加载器][composite]和 [OBJ 模型加载器][obj] 使用此功能。

可见性条目指定为 `"part name": boolean`：

```js
// 复合模型：part_two 默认不可见
{ "loader": "forge:composite", "children": { "part_one": {...}, "part_two": {...} }, "visibility": { "part_two": false } }

// 子模型 1：仅 part_two 可见
{ "parent": "mymod:mycompositemodel", "visibility": { "part_one": false, "part_two": true } }

// 子模型 2：part_two 可见
{ "parent": "mymod:mycompositemodel", "visibility": { "part_two": true } }
```

可见性判定：先查自身，没有再递归查父级，直到找到条目或没有父级可查（默认为 true）。

[bakedmodel]: ../modelloaders/bakedmodel.md
[composite]: ../modelloaders/index.md
[obj]: ../modelloaders/index.md
