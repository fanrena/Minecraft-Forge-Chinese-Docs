根变换
===============

在模型 JSON 顶层添加 `transform` 条目，告知加载器应在方块模型的 blockstate 文件中的旋转之前，或在物品模型的显示变换之前，对所有几何体应用变换。该变换可通过 `IUnbakedGeometry#bake()` 中的 `IGeometryBakingContext#getRootTransform()` 获取。

自定义模型加载器可以完全忽略此字段。

根变换可以以两种格式指定：

1. **矩阵格式**：包含 `matrix` 条目，含原始变换矩阵（3×4，行主序）：
    ```js
    "transform": { "matrix": [ [0,0,0,0], [0,0,0,0], [0,0,0,0] ] }
    ```
2. **元素格式**：包含以下条目的任意组合：
    * `origin`：旋转/缩放原点。值：`"corner"`(0,0,0) / `"center"`(.5,.5,.5) / `"opposing-corner"`(1,1,1) 或 `[x,y,z]`
    * `translation`：相对平移 `[x,y,z]`
    * `rotation`/`left_rotation`：缩放前的旋转（绕平移后原点）
    * `scale`：缩放 `[x,y,z]`
    * `right_rotation`/`post_rotation`：缩放后的旋转

应用顺序：translation → left_rotation → scale → right_rotation，最后移动到原点。

```js
{ "transform": { "origin": "center", "translation": [0, 0.5, 0], "rotation": { "y": 45 } } }
```

旋转格式支持：单轴 `{"x":90}`、轴数组 `[{"x":90},{"y":45}]`、欧拉角 `[90,180,45]`、四元数 `[0.382,0,0,0.924]`。

[blockstate]: https://minecraft.wiki/w/Tutorials/Models#Block_states
[displaytransform]: ../modelloaders/transform.md
