# Minecraft Forge 1.20.1 文档索引 (AI速查)

生成:2026-06-22 | zh:`D:\Projects\ForgeDocs\Documentation-1.20.1-zh\`

## 术语表
Block=方块, Item=物品, Entity=实体, BlockEntity=方块实体(BE), BlockState=方块状态, Registry=注册表, DeferredRegister=延迟注册器, RegistryObject=注册对象, Event Bus=事件总线, Capability=能力, Codec=编解码器, ResourceLocation=资源位置(ns:path), Loot Table=战利品表, Tag=标签, Advancement=进度, Recipe=配方, Menu=菜单容器, Screen=屏幕, Dist=分发端(CLIENT/SERVER), Logical Side=逻辑端, Physical Side=物理端, BakedModel=烘焙模型, GLM=全局战利品修改器, AT=访问转换器, BEWLR=方块实体级物品渲染, I18n=国际化, AO=环境光遮蔽, DataGen=数据生成

## 文件路径速查

| 分类 | 文件 | 内容 |
|------|------|------|
| 入口 | `index.md` | 首页 |
| 入口 | `contributing.md` | 贡献指南 |
| 入门 | `gettingstarted/index.md` | 环境搭建(MDK) |
| 入门 | `gettingstarted/modfiles.md` | mods.toml 配置 |
| 入门 | `gettingstarted/structuring.md` | 包结构规范 |
| 入门 | `gettingstarted/versioning.md` | 版本号格式 |
| 概念 | `concepts/events.md` | 事件系统 |
| 概念 | `concepts/registries.md` | 注册表 |
| 概念 | `concepts/resources.md` | 资源系统 |
| 概念 | `concepts/sides.md` | 端区分 |
| 概念 | `concepts/lifecycle.md` | 模组生命周期 |
| 概念 | `concepts/internationalization.md` | 国际化/本地化 |
| 方块 | `blocks/index.md` | 方块创建 |
| 方块 | `blocks/states.md` | BlockState 属性 |
| 方块实体 | `blockentities/index.md` | BlockEntity |
| 方块实体 | `blockentities/ber.md` | 方块实体渲染器 |
| 物品 | `items/index.md` | Item + CreativeTab |
| 物品 | `items/bewlr.md` | 物品动态渲染 |
| GUI | `gui/menus.md` | Menu容器 |
| GUI | `gui/screens.md` | Screen渲染 |
| 网络 | `networking/index.md` | 网络概述 |
| 网络 | `networking/simpleimpl.md` | SimpleChannel |
| 网络 | `networking/entities.md` | 实体同步 |
| 渲染 | `rendering/modelextensions/` | 面数据/RenderType/变换/可见性 |
| 渲染 | `rendering/modelloaders/` | BakedModel/Transform/Override |
| 效果 | `gameeffects/particles.md` | 粒子系统 |
| 效果 | `gameeffects/sounds.md` | 声音系统 |
| 数据生成 | `datagen/index.md` | DataGenerator概览 |
| 数据生成 | `datagen/client/` | 语言/模型/声音生成 |
| 数据生成 | `datagen/server/` | 进度/GLM/战利品表/配方/标签生成 |
| 数据存储 | `datastorage/capabilities.md` | Capability系统 |
| 数据存储 | `datastorage/codecs.md` | Codec编解码 |
| 数据存储 | `datastorage/saveddata.md` | 持久化数据 |
| 资源 | `resources/client/` | 资源包/模型/纹理着色/物品属性 |
| 资源 | `resources/server/` | 数据包/配方/战利品表/GLM/标签/进度/条件 |
| 高级 | `advanced/accesstransformers.md` | AT |
| Forge开发 | `forgedev/` | 贡献Forge本身 |
| 遗留 | `legacy/` | 旧版本/移植 |
| 杂项 | `misc/config.md` | 配置系统 |
| 杂项 | `misc/keymappings.md` | 按键映射 |
| 杂项 | `misc/gametest.md` | 游戏测试 |
| 杂项 | `misc/debugprofiler.md` | 调试分析器 |
| 杂项 | `misc/updatechecker.md` | 更新检查器 |

## 关键API模式

### 注册
```java
// DeferredRegister (推荐)
DeferredRegister<Block> BLOCKS = DeferredRegister.create(ForgeRegistries.BLOCKS, MODID);
RegistryObject<Block> MY_BLOCK = BLOCKS.register("name", () -> new Block(properties));
BLOCKS.register(modBus); // 在构造函数中

// RegisterEvent (备选)
@SubscribeEvent void register(RegisterEvent e) {
  e.register(ForgeRegistries.Keys.BLOCKS, helper -> helper.register(loc, block));
}
```

### 事件
```java
// 方式1: 注解(自动注册)
@Mod.EventBusSubscriber(modid="mymod", bus=Bus.FORGE)
class H { @SubscribeEvent static void on(Event e) {} }

// 方式2: 手动注册
MinecraftForge.EVENT_BUS.register(instance/class);
modBus.addListener(this::handler); // 模组总线
```

### 网络
```java
SimpleChannel INST = NetworkRegistry.newSimpleChannel(loc, ()->"1", "1"::equals, "1"::equals);
INST.registerMessage(id, Msg.class, Msg::encode, Msg::decode, Msg::handle);
INST.sendToServer(new Msg()); // C→S
INST.send(PacketDistributor.PLAYER.with(player), new Msg()); // S→C
```

### GUI
```java
// Menu + Screen
MenuType<MyMenu> TYPE = IForgeMenuType.create(MyMenu::new);  // IContainerFactory
NetworkHooks.openScreen(serverPlayer, new SimpleMenuProvider(MyMenu::new, title));
MenuScreens.register(TYPE.get(), MyScreen::new); // FMLClientSetupEvent
```

### 能力
```java
// 暴露
@Override <T> LazyOptional<T> getCapability(Capability<T> cap, Direction side) {
  return ForgeCapabilities.ITEM_HANDLER.orEmpty(cap, inventory.cast());
}
// 注册自定义能力
@SubscribeEvent void regCaps(RegisterCapabilitiesEvent e) { e.register(IMyCap.class); }
```

### 端隔离
```java
// 条件检查
level.isClientSide // 逻辑端判断
// 物理端分发
DistExecutor.unsafeRunWhenOn(Dist.CLIENT, () -> () -> clientOnlyMethod());
DistExecutor.safeCallWhenOn(Dist.CLIENT, () -> ClientClass::methodRef);
```

### 数据生成
```java
@SubscribeEvent void gatherData(GatherDataEvent e) {
  e.getGenerator().addProvider(e.includeClient(), output -> new MyProvider(output, MODID, e.getExistingFileHelper()));
}
```

### Codec
```java
// Record
RecordCodecBuilder.create(instance -> instance.group(
  Codec.STRING.fieldOf("name").forGetter(O::getName)
).apply(instance, O::new));
```

## 重要类速查

| 类/接口 | 用途 |
|---------|------|
| `MinecraftForge.EVENT_BUS` | Forge主事件总线 |
| `FMLJavaModLoadingContext.get().getModEventBus()` | 模组事件总线 |
| `DeferredRegister` | 延迟注册器 |
| `RegistryObject` | 注册对象引用 |
| `ForgeRegistries` | Forge包装的注册表 |
| `ResourceLocation` | `ns:path` 格式标识符 |
| `Dist` / `LogicalSide` | 物理端/逻辑端枚举 |
| `SimpleChannel` | 网络通道 |
| `MenuType` / `AbstractContainerMenu` | 菜单容器 |
| `AbstractContainerScreen` | 容器屏幕 |
| `BakedModel` | 烘焙模型 |
| `Codec` | 编解码器 |
| `IItemHandler` / `IFluidHandler` / `IEnergyStorage` | Forge提供的能力 |
| `ForgeConfigSpec` | 配置规范 |
| `SavedData` | 持久化数据 |
| `GameTestHelper` | 游戏测试辅助 |

## 阅读顺序
```
gettingstarted → concepts → (blocks/items/blockentities/networking/gui) → (rendering/gameeffects/datagen/datastorage/resources/advanced/misc)
```
`datagen/server/`(自动生成) 与 `resources/server/`(手动编写) 内容重叠，互为参考。

## 翻译状态: 72/72 ✓ 全部完成
PDF: Forge文档_1.20.1_中文版.pdf(907KB) + ForgeDocs_1.20.1_EN.pdf(618KB)
