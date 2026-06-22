面数据
=========

在原版"elements"模型中，关于元素面的额外数据可以在元素级别或面级别指定。未指定自己面数据的面将回退到元素的面数据，如果元素级别也未指定，则使用默认值。

要使用此扩展于生成的物品模型，模型必须通过 `forge:item_layers` 模型加载器加载，因为原版物品模型生成器未被扩展以读取这些额外数据。

所有面数据的值都是可选的。

Elements 模型
--------------

在原版"elements"模型中，面数据适用于指定了该数据的面对，或未指定面数据的元素的所有面。

!!!note
    如果在面上指定了 `forge_data`，它将不会从元素级别的 `forge_data` 声明继承任何参数。

额外数据可以通过以下示例中的两种方式指定：
```js
{
  "elements": [
    {
      "forge_data": {
        "color": "0xFFFF0000",
        "block_light": 15,
        "sky_light": 15,
        "ambient_occlusion": false
      },
      "faces": {
        "north": {
          "forge_data": {
            "color": "0xFFFF0000",
            "block_light": 15,
            "sky_light": 15,
            "ambient_occlusion": false
          }
        }
      }
    }
  ]
}
```

生成的物品模型
--------------------

在使用 `forge:item_layers` 加载器生成的物品模型中，面数据为每个纹理层指定，并应用于所有几何体（正面/背面四边形和边缘四边形）。

`forge_data` 字段必须位于模型 JSON 的顶层，每个键值对将一个面数据对象关联到一个层索引。

在以下示例中，第 1 层将被着色为红色并以最大亮度发光：
```js
{
  "textures": {
    "layer0": "minecraft:item/stick",
    "layer1": "minecraft:item/glowstone_dust"
  },
  "forge_data": {
    "1": {
      "color": "0xFFFF0000",
      "block_light": 15,
      "sky_light": 15,
      "ambient_occlusion": false
    }
  }
}
```

参数
----------

### 颜色

使用 `color` 条目指定颜色值将把该颜色作为着色应用于四边形。默认值为 `0xFFFFFFFF`（白色，完全不透明）。颜色必须采用 `ARGB` 格式，打包为 32 位整数，可以指定为十六进制字符串（`"0xAARRGGBB"`）或十进制整数（JSON 不支持十六进制整数）。

!!! warning
    四个颜色分量会与纹理的像素相乘。省略 alpha 分量相当于将其设为 0，这将使几何体完全透明。

如果颜色值是恒定的，这可以用作 [`BlockColor` 和 `ItemColor`][tinting] 着色的替代方案。

### 方块光和天空光

通过 `block_light` 和 `sky_light` 条目分别指定方块光和/或天空光值，将覆盖四边形的相应光照值。两个值默认为 0。值必须在 0-15（含）范围内，当面被渲染时，它们被视为对应光照类型的最小值，意味着世界中该光照类型的更高值将覆盖指定的值。

指定的光照值纯粹是客户端的，既不影响服务器的光照级别，也不影响周围方块的亮度。

### 环境光遮蔽

指定 `ambient_occlusion` 标志将为四边形配置 [AO]。默认值为 `true`。此标志的行为等同于原版格式的顶层 `ambientocclusion` 标志。

!!! note
    如果顶层 AO 标志设置为 false，在元素或面上将此标志指定为 true 将无法覆盖顶层标志。

[tinting]: ../../resources/client/models/tinting.md
[AO]: https://en.wikipedia.org/wiki/Ambient_occlusion
