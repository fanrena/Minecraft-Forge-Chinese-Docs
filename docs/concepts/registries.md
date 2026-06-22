注册表
==========

注册是将模组的对象（如物品、方块、声音等）告知游戏的过程。注册非常重要，因为如果不注册，游戏根本不知道这些对象的存在，这将导致无法解释的行为和崩溃。

游戏中大多数需要注册的内容都由 Forge 注册表处理。注册表类似于一个将值映射到键的对象。Forge 使用以 [`ResourceLocation`][ResourceLocation] 为键的注册表来注册对象。这使得 `ResourceLocation` 可以作为对象的"注册名称"。

每种可注册对象类型都有其自己的注册表。要查看 Forge 包装的所有注册表，请参阅 `ForgeRegistries` 类。同一注册表中的所有注册名称必须唯一。但是，不同注册表中的名称不会冲突。例如，有一个 `Block` 注册表和一个 `Item` 注册表。一个 `Block` 和一个 `Item` 可以以相同的名称 `example:thing` 注册而不会冲突；但是，如果两个不同的 `Block` 或两个不同的 `Item` 以完全相同的名称注册，则第二个对象将覆盖第一个。

注册方法
------------------

有两种正确的对象注册方式：`DeferredRegister` 类和 `RegisterEvent` 生命周期事件。

### DeferredRegister

`DeferredRegister` 是推荐的对象注册方式。它允许使用静态初始化器的便利性，同时避免了相关问题。它只是维护一个条目的供应者列表，并在 `RegisterEvent` 期间从这些供应者注册对象。

一个模组注册自定义方块的示例：

```java
private static final DeferredRegister<Block> BLOCKS = DeferredRegister.create(ForgeRegistries.BLOCKS, MODID);

public static final RegistryObject<Block> ROCK_BLOCK = BLOCKS.register("rock", () -> new Block(BlockBehaviour.Properties.of().mapColor(MapColor.STONE)));

public ExampleMod() {
  BLOCKS.register(FMLJavaModLoadingContext.get().getModEventBus());
}
```

### `RegisterEvent`

`RegisterEvent` 是第二种注册对象的方式。此[事件]在模组构造方法之后、配置加载之前为每个注册表触发一次。对象通过 `#register` 注册，传入注册表键、注册对象名称和对象本身。还有一个额外的 `#register` 重载，接受一个消费辅助器来注册具有给定名称的对象。建议使用此方法以避免不必要的对象创建。

示例如下：（事件处理器注册在*模组事件总线*上）

```java
@SubscribeEvent
public void register(RegisterEvent event) {
  event.register(ForgeRegistries.Keys.BLOCKS,
    helper -> {
      helper.register(new ResourceLocation(MODID, "example_block_1"), new Block(...));
      helper.register(new ResourceLocation(MODID, "example_block_2"), new Block(...));
      helper.register(new ResourceLocation(MODID, "example_block_3"), new Block(...));
      // ...
    }
  );
}
```

### 非 Forge 注册表

并非所有注册表都由 Forge 包装。这些可以是静态注册表，如 `LootItemConditionType`，使用是安全的。还有动态注册表，如 `ConfiguredFeature` 和其他一些世界生成注册表，通常以 JSON 表示。`DeferredRegister#create` 有一个重载，允许模组开发者指定要为其创建 `RegistryObject` 的原版注册表的注册表键。注册方法和附加到模组事件总线的方式与其他 `DeferredRegister` 相同。

!!! important
    动态注册表对象**只能**通过数据文件（例如 JSON）注册。它们**不能**在代码中注册。

```java
private static final DeferredRegister<LootItemConditionType> REGISTER = DeferredRegister.create(Registries.LOOT_CONDITION_TYPE, "examplemod");

public static final RegistryObject<LootItemConditionType> EXAMPLE_LOOT_ITEM_CONDITION_TYPE = REGISTER.register("example_loot_item_condition_type", () -> new LootItemConditionType(...));
```

!!! note
    有些类本身不能注册。相反，注册的是 `*Type` 类，并在前者的构造方法中使用。例如，[`BlockEntity`][blockentity] 有 `BlockEntityType`，`Entity` 有 `EntityType`。这些 `*Type` 类是按需创建包含类型的工厂。

    这些工厂通过使用对应的 `*Type$Builder` 类创建。例如：（`REGISTER` 指的是一个 `DeferredRegister<BlockEntityType>`）
    ```java
    public static final RegistryObject<BlockEntityType<ExampleBlockEntity>> EXAMPLE_BLOCK_ENTITY = REGISTER.register(
      "example_block_entity", () -> BlockEntityType.Builder.of(ExampleBlockEntity::new, EXAMPLE_BLOCK.get()).build(null)
    );
    ```

引用已注册对象
------------------------------

已注册对象在创建和注册时不应存储在字段中。它们应在每次 `RegisterEvent` 为该注册表触发时重新创建和注册。这是为了在未来的 Forge 版本中支持模组的动态加载和卸载。

已注册对象必须始终通过 `RegistryObject` 或使用 `@ObjectHolder` 的字段来引用。

### 使用 RegistryObject

`RegistryObject` 可用于在已注册对象可用时获取对它们的引用。`DeferredRegister` 使用它们返回对已注册对象的引用。它们的引用在 `RegisterEvent` 为其注册表调用后更新，同时更新的还有 `@ObjectHolder` 注解。

要获取 `RegistryObject`，使用一个 `ResourceLocation` 和可注册对象的 `IForgeRegistry` 调用 `RegistryObject#create`。也可以通过提供注册表名称来使用自定义注册表。将 `RegistryObject` 存储在 `public static final` 字段中，并在需要已注册对象时调用 `#get`。

使用 `RegistryObject` 的示例：

```java
public static final RegistryObject<Item> BOW = RegistryObject.create(new ResourceLocation("minecraft:bow"), ForgeRegistries.ITEMS);

// 假设 'neomagicae:mana_type' 是一个有效的注册表，且 'neomagicae:coffeinum' 是该注册表中的一个有效对象
public static final RegistryObject<ManaType> COFFEINUM = RegistryObject.create(new ResourceLocation("neomagicae", "coffeinum"), new ResourceLocation("neomagicae", "mana_type"), "neomagicae"); 
```

### 使用 @ObjectHolder

注册表中的已注册对象可以通过使用 `@ObjectHolder` 注解类或字段，并提供足够的信息来构造 `ResourceLocation` 以标识特定注册表中的特定对象，从而注入到 `public static` 字段中。

`@ObjectHolder` 的规则如下：

* 如果类使用 `@ObjectHolder` 注解，其值将是该类中所有字段的默认命名空间（如果未显式定义）
* 如果类使用 `@Mod` 注解，模组 ID 将是该类中所有注解字段的默认命名空间（如果未显式定义）
* 一个字段被视为注入对象需要满足以下条件：
  * 至少具有 `public static` 修饰符；
  * **字段**使用 `@ObjectHolder` 注解，且：
    * name 值已显式定义；并且
    * registry name 值已显式定义
  * _如果字段没有对应的注册表或名称，将在编译时抛出异常_
* _如果生成的 `ResourceLocation` 不完整或无效（路径中包含非法字符），将抛出异常_
* 如果没有其他错误或异常发生，字段将被注入
* 如果上述所有规则都不适用，则不采取任何操作（可能会记录一条消息）

使用 `@ObjectHolder` 注解的字段在 `RegisterEvent` 为其注册表触发后注入值，同时注入的还有 `RegistryObject`。

!!! note
    如果在注入时注册表中不存在该对象，将记录一条调试消息，并且不会注入任何值。

由于这些规则相当复杂，这里有一些示例：

```java
class Holder {
  @ObjectHolder(registryName = "minecraft:enchantment", value = "minecraft:flame")
  public static final Enchantment flame = null;     // 注解存在。需要 [public static]。[final] 是可选的。
                                                    // 注册表名称显式定义："minecraft:enchantment"
                                                    // 资源位置显式定义："minecraft:flame"
                                                    // 注入内容：来自 [Enchantment] 注册表的 "minecraft:flame"

  public static final Biome ice_flat = null;        // 字段上没有注解。
                                                    // 因此，该字段被忽略。

  @ObjectHolder("minecraft:creeper")
  public static Entity creeper = null;              // 注解存在。需要 [public static]。
                                                    // 字段上未指定注册表。
                                                    // 因此，这将产生编译时异常。

  @ObjectHolder(registryName = "potion")
  public static final Potion levitation = null;     // 注解存在。需要 [public static]。[final] 是可选的。
                                                    // 注册表名称显式定义："minecraft:potion"
                                                    // 字段上未指定资源位置
                                                    // 因此，这将产生编译时异常。
}
```

创建自定义 Forge 注册表
--------------------------------

自定义注册表通常可以只是一个简单的键到值的映射。这是一种常见的方式；但是，它强制对注册表的存在产生硬依赖。它还需要手动同步端之间的任何数据。自定义 Forge 注册表提供了一种简单的替代方案，可以实现软依赖以及更好的管理和端之间的自动同步（除非另有说明）。由于对象也使用 Forge 注册表，注册以相同的方式标准化。

自定义 Forge 注册表是使用 `RegistryBuilder` 创建的，可以通过 `NewRegistryEvent` 或 `DeferredRegister`。`RegistryBuilder` 类接受各种参数（例如注册表的名称、ID 范围以及注册表上不同事件的回调）。新的注册表在 `NewRegistryEvent` 完成触发后注册到 `RegistryManager`。

任何新创建的注册表应使用其关联的[注册方法][registration]来注册关联的对象。

### 使用 NewRegistryEvent

当使用 `NewRegistryEvent` 时，使用 `RegistryBuilder` 调用 `#create` 将返回一个供应者包装的注册表。提供的注册表可以在 `NewRegistryEvent` 完成发布到模组事件总线后访问。在 `NewRegistryEvent` 完成触发之前从供应者获取自定义注册表将返回 `null` 值。

#### 新数据包注册表

可以使用模组事件总线上的 `DataPackRegistryEvent$NewRegistry` 事件添加新的数据包注册表。通过传入表示注册表名称的 `ResourceKey` 和用于从 JSON 编解码数据的 `Codec`，使用 `#dataPackRegistry` 创建注册表。可以提供可选的 `Codec` 用于将数据包注册表同步到客户端。

!!! important
    数据包注册表不能使用 `DeferredRegister` 创建。它们只能通过事件创建。

### 使用 DeferredRegister

`DeferredRegister` 方法再次是对上述事件的包装。一旦使用接受注册表名称和模组 ID 的 `#create` 重载在常量字段中创建了 `DeferredRegister`，就可以通过 `DeferredRegister#makeRegistry` 构造注册表。这接受一个包含任何额外配置的 `RegistryBuilder` 供应者。该方法默认已填充 `#setName`。由于此方法可以随时返回，返回的是 `IForgeRegistry` 的供应者版本。在 `NewRegistryEvent` 触发之前从供应者获取自定义注册表将返回 `null` 值。

!!! important
    `DeferredRegister#makeRegistry` 必须在 `DeferredRegister` 通过 `#register` 添加到模组事件总线之前调用。`#makeRegistry` 也使用 `#register` 方法在 `NewRegistryEvent` 期间创建注册表。

处理缺失条目
------------------------

在某些情况下，当模组更新或（更常见的）移除时，某些注册表对象将不再存在。可以通过第三个注册表事件 `MissingMappingsEvent` 指定处理缺失映射的操作。在此事件中，可以通过 `#getMappings`（给定注册表键和模组 ID）或 `#getAllMappings`（给定注册表键）获取缺失映射的列表。

!!! important
    `MissingMappingsEvent` 在 **Forge** 事件总线上触发。

对于每个 `Mapping`，可以选择以下四种映射类型之一来处理缺失条目：

| 操作 | 描述 |
| :---: | :--- |
| IGNORE | 忽略缺失条目并放弃映射。 |
| WARN | 在日志中生成警告。 |
| FAIL | 阻止世界加载。 |
| REMAP | 将条目重新映射到已注册的非空对象。 |

如果未指定任何操作，将执行默认操作，通知用户关于缺失条目的信息，并询问是否仍要加载世界。除重新映射外的所有操作将阻止任何其他注册对象占用现有 ID 的位置，以防相关条目将来重新添加到游戏中。

[ResourceLocation]: ./resources.md#resourcelocation
[registration]: #注册方法
[event]: ./events.md
[blockentity]: ../blockentities/index.md
