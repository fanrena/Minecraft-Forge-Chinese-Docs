# Minecraft Forge 1.20.1 文档索引

生成:2026-06-22 | src:https://docs.minecraftforge.net | zh:`D:\Projects\ForgeDocs\Documentation-1.20.1-zh\`

## 术语
Block=方块, Item=物品, Entity=实体, BlockEntity=方块实体(BE), Registry=注册表, DeferredRegister=延迟注册器, RegistryObject=注册对象, Event Bus=事件总线, Capability=能力, Codec=编解码器, ResourceLocation=资源位置(ns:path), Loot Table=战利品表, Tag=标签, Advancement=进度, Recipe=配方, Dist=分发端(CLIENT/SERVER), Logical Side=逻辑端, Physical Side=物理端, BakedModel=烘焙模型, GLM=全局战利品修改器, AT=访问转换器, I18n=国际化

## 结构
```
docs/
├── index.md + contributing.md   # 入口
├── gettingstarted/              # ★起点: 环境搭建
│   ├── index.md modfiles.md(structuring.md versioning.md
├── concepts/                    # ★核心: 必须先理解
│   ├── events.md                # 事件总线, @SubscribeEvent, EventPriority, cancel
│   ├── registries.md            # DeferredRegister, RegistryObject, @ObjectHolder, RegisterEvent
│   ├── resources.md             # ResourceLocation, assets vs data
│   ├── sides.md                 # Dist, LogicalSide, Level#isClientSide, DistExecutor
│   ├── lifecycle.md             # FMLCommonSetup, FMLClientSetup, InterModComms
│   └── internationalization.md  # TranslatableComponent, 语言JSON
├── blocks/                      # 方块创建/注册, BlockState属性系统
├── blockentities/               # BlockEntity, BER, 数据存储/同步
├── items/                       # Item创建, CreativeTab, BEWLR
├── gui/                         # Menu(服务端容器), Screen(客户端渲染), AbstractContainerScreen
├── networking/                  # SimpleChannel, 数据包编码解码, PacketDistributor
├── rendering/                   # 模型扩展(FaceData/RenderType/Transform/Visibility), 模型加载器(BakedModel/Override/Transform)
├── gameeffects/                 # 粒子, 声音
├── datagen/                     # 数据生成: client(本地化/模型/声音), server(进度/注册表/GLM/战利品表/配方/标签)
├── datastorage/                 # Capability, Codec, SavedData
├── resources/                   # 资源包(client/models: 物品属性/着色), 数据包(server: 进度/条件/GLM/战利品表/标签/配方)
├── advanced/                    # Access Transformer
├── forgedev/                    # Forge开发与PR指南
├── legacy/                      # 移植指南
└── misc/                        # 配置/调试/游戏测试/按键映射/更新检查
```

## 阅读顺序
gettingstarted → concepts → (blocks|items|blockentities|networking|gui) → (rendering|gameeffects|datagen|datastorage|resources)

## 关键API速查
- 注册: `DeferredRegister.create(ForgeRegistries.BLOCKS, MODID)` → `register(name, supplier)` → `BLOCKS.register(modBus)`
- 事件: `MinecraftForge.EVENT_BUS.register(instance)` / `@Mod.EventBusSubscriber` / `modBus.addListener(this::handler)`
- 网络: `NetworkRegistry.newSimpleChannel(loc, ver, ckSrv, ckCli)` → `INSTANCE.registerMessage(id, clz, enc, dec, hdl)` → `INSTANCE.sendToServer(msg)` / `PacketDistributor.PLAYER.with(p)`
- 菜单: `MenuType` + `IContainerFactory` + `AbstractContainerMenu` → `NetworkHooks.openScreen`
- 数据同步: `SlotItemHandler` / `DataSlot` / `ContainerData` / `SynchedEntityData#defineId`
- 端隔离: `DistExecutor.unsafeRunWhenOn(Dist.CLIENT, ...)` / `Level#isClientSide`

## 交叉引用
`datagen/server/`(自动生成) ↔ `resources/server/`(手动编写), 主题重叠
`lifecycle.md`→registries,capabilities,datagen,sides | `registries.md`→resources,events,blockentities | `sides.md`→modfiles

## 翻译状态: 29/73 ✓
已完成: README, CONTRIBUTING, docs/index, docs/contributing, concepts/*6, blocks/*2, blockentities/*2, gettingstarted/*4, items/*2, gui/*2, networking/*3
待翻译: rendering/*7, gameeffects/*2, advanced/*1, datagen/*10, datastorage/*3, resources/*14, forgedev/*2, legacy/*2, misc/*5
