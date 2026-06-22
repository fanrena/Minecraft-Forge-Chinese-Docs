持久化数据（SavedData）
==========

SavedData（SD）系统是 Level Capabilities 的替代方案，用于为每个维度附加持久化数据。

### 声明
继承 `SavedData`，实现两个方法：
- `#save(tag)` — 将数据写入 NBT
- `#setDirty()` — 数据变更后调用，否则 `#save` 不会被调用

### 附加到维度
通过 `DimensionDataStorage#computeIfAbsent(loader, factory, name)` 创建/加载 SD：
- `loader`：从 NBT 加载数据的函数
- `factory`：创建新实例的供应者
- `name`：`.dat` 文件名（不包含扩展名）

```java
// 在 Nether（DIM-1）中创建 ./<level_folder>/DIM-1/data/example.dat
netherDataStorage.computeIfAbsent(this::load, this::create, "example");
```

### 跨维度持久化
将 SD 附加到主世界（Overworld），通过 `MinecraftServer#overworld` 获取，因为主世界是唯一永远不会完全卸载的维度。
