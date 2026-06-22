模型
======

模型系统是 Minecraft 赋予方块和物品形状的方式。模型通过 `ResourceLocation` 与纹理关联，存储在 `ModelManager` 中。

方块使用 `ModelResourceLocation`（注册名 + 序列化的 BlockState），物品使用注册名 + `inventory`。

### 纹理
- UV 坐标 (0,0) 为**左上角**，范围 0-16
- 纹理应为正方形，边长为 2 的幂（1x1, 2x2, 16x16, 128x128 等）
- 非正方形纹理仅用于[动画][animated]

### 注意
JSON 模型仅支持长方体元素，无法表达三角形等复杂形状。如需复杂模型，需使用其他格式（如 OBJ）。

[models]: https://minecraft.wiki/w/Tutorials/Models
[resloc]: ../../../concepts/resources.md
[animated]: https://minecraft.wiki/w/Resource_Pack#Animation
[state]: ../../../blocks/states.md
