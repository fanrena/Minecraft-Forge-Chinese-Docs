物品属性
===============

物品属性将物品的"属性"暴露给模型系统。例如弓的拉弓程度，用于选择模型创建动画。

### 注册物品属性
`ItemProperties#register(item, name, propertyFunction)` — 在 `FMLClientSetupEvent` 中调用，需使用 `#enqueueWork`。
- `item`：目标物品
- `name`：属性名称，建议使用模组 ID 作为命名空间
- `propertyFunction`：`(ItemStack, ClientLevel, LivingEntity, int) → float`

`ItemProperties#registerGeneric` — 为所有物品注册属性。

### 使用覆盖
在模型 JSON 中使用 `overrides` 数组，谓词匹配所有**大于等于**给定值的属性。

```js
{
  "overrides": [
    { "predicate": { "examplemod:power": 0.75 }, "model": "examplemod:item/example_powered" }
  ]
}
```

多个匹配时选择列表中的最后一个。

[format]: https://minecraft.wiki/w/Tutorials/Models#Item_models
