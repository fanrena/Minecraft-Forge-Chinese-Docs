`ItemOverrides`
==================

`ItemOverrides` 为 `BakedModel` 提供根据 `ItemStack` 状态切换模型的能力，是实现动态模型的关键。函数签名：`(BakedModel, ItemStack, ClientLevel, LivingEntity, int)` → `BakedModel`。

### 核心类

- **`ItemOverrides`** — 持有 `BakedOverride` 列表，通过 `#resolve` 方法解析物品状态并返回对应模型。
- **`BakedOverride`** — 表示一个原版物品覆盖，包含属性匹配器（predicate）和满足条件时使用的目标模型。

### 原版 JSON 格式
```js
{
  "overrides": [
    { "predicate": { "example:prop": 0.5 }, "model": "example:item/model1" },
    { "predicate": { "example:prop": 1 },   "model": "example:item/model2" }
  ]
}
```

[baked]: ./bakedmodel.md
