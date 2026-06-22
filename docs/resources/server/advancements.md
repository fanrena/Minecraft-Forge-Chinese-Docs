进度
============

进度是玩家可以达成的任务，由 JSON 数据驱动。

### 进度条件
- 使用 `criteria` 对象定义条件
- `requirements` 定义条件组合方式（AND/OR）
- 条件触发器定义在 `CriteriaTriggers` 中

### 自定义条件触发器
1. 继承 `AbstractCriterionTriggerInstance` — 持有条件数据，实现 `#matches` 和 `#serializeToJson`
2. 继承 `SimpleCriterionTrigger<T>` — 指定 `#getId`，实现 `#createInstance`，提供 `#trigger` 方法
3. 在 `FMLCommonSetupEvent` 中使用 `CriteriaTriggers#register` 注册（注意线程安全）

### 进度奖励
经验值、战利品表、配方（配方书）、函数。

```js
"rewards": { "experience": 10, "loot": ["..."], "recipes": ["..."], "function": "..." }
```

[wiki]: https://minecraft.wiki/w/Advancement/JSON_format
[conditional]: ./conditional.md
