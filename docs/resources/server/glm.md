全局战利品修改器
===========

全局战利品修改器（GLM）是一种数据驱动的方法，用于处理掉落物修改，无需覆盖数十到数百个原版战利品表。

### 需要 4 个步骤
1. **`data/forge/loot_modifiers/global_loot_modifiers.json`** — 声明所有加载的修改器列表（`entries` + `replace`）
2. **序列化 JSON** — `data/<namespace>/loot_modifiers/<name>.json`，包含 `type`（Codec 注册名）、`conditions`（激活条件）和自定义属性
3. **`IGlobalLootModifier` 实现** — 建议继承 `LootModifier`，实现 `#doApply` 和 `#codec`
4. **Codec** — 注册 `Codec<? extends IGlobalLootModifier>`，`LootModifier#codecStart` 简化条件部分

### JSON 示例
```js
{
  "type": "examplemod:example_loot_modifier",
  "conditions": [ /* loot table conditions */ ],
  "prop1": "val1", "prop2": 10, "prop3": "minecraft:dirt"
}
```

### 注意
- `global_loot_modifiers.json` 必须在 `forge` 命名空间下
- GLM 是叠加的（类似标签），而非后加载覆盖
- 多个 GLM 按注册顺序依次处理掉落物

[tags]: ./tags.md
[codec]: ../../datastorage/codecs.md
[datagen]: ../../datagen/server/glm.md
