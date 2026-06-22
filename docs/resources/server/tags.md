标签
====

标签是游戏中对象的广义集合，用于将相关内容分组并提供快速成员检查。

### 声明标签
标签在数据包中声明。`TagKey<Block>` 指向 `/data/<modid>/tags/blocks/<path>.json`。
- `replace: true` — 替换而非追加
- `required: false` — 条目不存在时不报错
- Forge 扩展：`remove` 数组 — 精细移除

### 代码中使用
- 通过 `*Tags#create`、`ITagManager#createTagKey`、`DeferredRegister#createTagKey`、`TagKey#create` 创建标签包装器
- 对象通过 `Holder#is(tag)` 或 `ITag#contains` 检查是否属于标签

### 约定
- 存在原版/Forge 标签时优先使用
- 公共分组使用 `forge` 命名空间
- 物品/方块分组使用复数名称（`minecraft:logs`）
- 物品标签按类型分目录（`forge:ingots/iron`）

### OreDictionary 迁移
- 配方中可直接使用标签
- 代码中使用 `stack.is(tag)` 替代旧版 OreDictionary 检查

[tags]: https://minecraft.wiki/w/Tag#JSON_format
