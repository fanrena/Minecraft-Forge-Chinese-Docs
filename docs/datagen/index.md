数据生成器
===============

数据生成器是一种以编程方式生成模组资源和数据的方式。它允许在代码中定义这些文件的内容并自动生成，无需关心具体细节。

数据生成器系统由主类 `net.minecraft.data.Main` 加载。可以通过不同的命令行参数来定制收集哪些模组的数据、考虑哪些现有文件等。负责数据生成的类是 `net.minecraft.data.DataGenerator`。

MDK 的 `build.gradle` 中默认添加了 `runData` 任务用于运行数据生成器。

现有文件
--------------
所有对未通过数据生成生成的纹理或其他数据文件的引用，都必须引用系统上现有的文件。这是为了确保所有引用的纹理都在正确的位置，以便发现和纠正拼写错误。

`ExistingFileHelper` 是负责验证这些数据文件是否存在的类。可以从 `GatherDataEvent#getExistingFileHelper` 获取其实例。

`--existing <folderpath>` 参数允许在验证文件是否存在时使用指定的文件夹及其子文件夹。此外，`--existing-mod <modid>` 参数允许使用已加载模组的资源进行验证。默认情况下，只有原版数据包和资源可用于 `ExistingFileHelper`。

生成器模式
---------------

数据生成器可以配置运行 4 种不同的数据生成，通过命令行参数配置，并可通过 `GatherDataEvent#include***` 方法检查。

* **客户端资源（Client Assets）**
  * 在 `assets` 中生成仅客户端的文件：方块/物品模型、blockstate JSON、语言文件等。
  * `--client`，`#includeClient`
* **服务端数据（Server Data）**
  * 在 `data` 中生成仅服务端的文件：配方、进度、标签等。
  * `--server`，`#includeServer`
* **开发工具（Development Tools）**
  * 运行一些开发工具：SNBT 与 NBT 相互转换等。
  * `--dev`，`#includeDev`
* **报告（Reports）**
  * 导出所有已注册的方块、物品、命令等。
  * `--reports`，`#includeReports`

`--all` 可以包含所有生成器。

数据提供者（Data Providers）
--------------

数据提供者是实际定义生成哪些数据的类。所有数据提供者实现 `DataProvider`。Minecraft 为大多数资源和数据提供了抽象实现，模组开发者只需扩展并重写指定的方法。

`GatherDataEvent` 在创建数据生成器时在模组事件总线上触发，可以从事件中获取 `DataGenerator`。使用 `DataGenerator#addProvider` 创建并注册数据提供者。

### 客户端资源
* `LanguageProvider` — 用于[语言字符串][lang]；实现 `#addTranslations`
* `SoundDefinitionsProvider` — 用于 [`sounds.json`][sounds]；实现 `#registerSounds`
* `ModelProvider<?>` — 用于[模型]；实现 `#registerModels`
    * `ItemModelProvider` — 用于物品模型
    * `BlockModelProvider` — 用于方块模型
* `BlockStateProvider` — 用于 blockstate JSON 及其方块和物品模型；实现 `#registerStatesAndModels`

### 服务端数据
* `GlobalLootModifierProvider` — 用于[全局战利品修改器][glm]；实现 `#start`
* `DatapackBuiltinEntriesProvider` — 用于数据包注册表对象；向构造函数传入 `RegistrySetBuilder`
* `LootTableProvider` — 用于[战利品表][loottable]；向构造函数传入 `LootTableProvider$SubProviderEntry`
* `RecipeProvider` — 用于[配方]及其解锁进度；实现 `#buildRecipes`
* `TagsProvider` — 用于[标签]；实现 `#addTags`
* `AdvancementProvider` — 用于[进度]；向构造函数传入 `AdvancementSubProvider`

[lang]: ./client/localization.md
[sounds]: ./client/sounds.md
[modelgen]: ./client/modelproviders.md
[glm]: ./server/glm.md
[loottable]: ./server/loottables.md
[recipes]: ./server/recipes.md
[tags]: ./server/tags.md
[advancements]: ./server/advancements.md
