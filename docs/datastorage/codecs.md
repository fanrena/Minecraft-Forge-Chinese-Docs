编解码器（Codec）
===================

Codec 是 Mojang 的 [DataFixerUpper] 提供的序列化工具，用于在 JSON（`JsonElement`）和 NBT（`Tag`）等格式之间转换对象。

### 使用 Codec
- `Codec#encodeStart(ops, obj)` — 编码
- `Codec#parse(ops, json)` — 解码
- `DynamicOps`：`JsonOps.INSTANCE`（标准 JSON）、`JsonOps.COMPRESSED`（压缩）、`NbtOps.INSTANCE`（NBT）
- 返回 `DataResult`，使用 `.resultOrPartial(err -> {})` 获取结果

### 现有 Codec
- 基础类型：`Codec.INT`, `Codec.STRING`, `Codec.BOOL`, `Codec.FLOAT` 等
- 原版：`ResourceLocation#CODEC`, `CompoundTag#CODEC`
- 注册表：`Registry#byNameCodec`, `IForgeRegistry#getCodec`

### 创建 Codec

**Record**（记录类型）：
```java
public static final Codec<SomeObject> CODEC = RecordCodecBuilder.create(instance ->
  instance.group(
    Codec.STRING.fieldOf("s").forGetter(SomeObject::s),
    Codec.INT.optionalFieldOf("i", 0).forGetter(SomeObject::i)
  ).apply(instance, SomeObject::new)
);
```

**转换器**：
- `#xmap(A→B, B→A)` — 双向完全等价
- `#comapFlatMap(A→DataResult<B>, B→A)` — 解码有约束
- `#flatComapMap(A→B, B→DataResult<A>)` — 编码有约束
- `#flatXMap(A→DataResult<B>, B→DataResult<A>)` — 双向有约束
- `#intRange(min, max)` — 整数范围约束

**其他**：
- `#orElse(default)` — 编解码失败时使用默认值
- `#unit(supplier)` — 仅提供代码内值，不编解码
- `#listOf()` — 列表
- `#unboundedMap(keyCodec, valueCodec)` — 映射
- `#pair(codec1, codec2)` — 对
- `#either(codec1, codec2)` — 二选一
- `#dispatch(typeGetter, codecGetter)` — 分发，根据 `type` 字段选择子 codec

[DataFixerUpper]: https://github.com/Mojang/DataFixerUpper
