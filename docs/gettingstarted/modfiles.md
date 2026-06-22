模组文件
=========

模组文件负责确定哪些模组被打包到你的 JAR 中、在 'Mods' 菜单中显示什么信息以及你的模组应如何在游戏中加载。

mods.toml
---------

`mods.toml` 文件定义了你的模组的元数据。它还包含在 'Mods' 菜单中显示的其他信息以及你的模组应如何加载到游戏中。

该文件使用 [Tom's Obvious Minimal Language，即 TOML][toml] 格式。该文件必须存放在所用源集的资源目录下的 `META-INF` 文件夹中（对于 `main` 源集为 `src/main/resources/META-INF/mods.toml`）。一个 `mods.toml` 文件可能如下所示：

```toml
modLoader="javafml"
loaderVersion="[46,)"

license="All Rights Reserved"
issueTrackerURL="https://github.com/MinecraftForge/MinecraftForge/issues"
showAsResourcePack=false

[[mods]]
  modId="examplemod"
  version="1.0.0.0"
  displayName="Example Mod"
  updateJSONURL="https://files.minecraftforge.net/net/minecraftforge/forge/promotions_slim.json"
  displayURL="https://minecraftforge.net"
  logoFile="logo.png"
  credits="我要感谢我的母亲和父亲。"
  authors="作者"
  description='''
  让你可以将泥土合成钻石。这是一个存在了千万年的传统模组。它是远古的。神圣的 Notch 创造了它。Jeb 将其彩虹化。Dinnerbone 把它颠倒了。等等。
  '''
  displayTest="MATCH_VERSION"

[[dependencies.examplemod]]
  modId="forge"
  mandatory=true
  versionRange="[46,)"
  ordering="NONE"
  side="BOTH"

[[dependencies.examplemod]]
  modId="minecraft"
  mandatory=true
  versionRange="[1.20]"
  ordering="NONE"
  side="BOTH"
```

`mods.toml` 分为三个部分：非模组特定属性（与模组文件关联）、模组属性（每个模组一个部分）和依赖配置（每个模组或模组组的依赖一个部分）。下面将解释与 `mods.toml` 文件相关的每个属性，其中 `required` 表示必须指定一个值，否则将抛出异常。

### 非模组特定属性

非模组特定属性是与 JAR 本身关联的属性，指示如何加载模组以及任何额外的全局元数据。

属性                 | 类型    | 默认值       | 描述 | 示例
:---                 | :---:   | :---:         | :---:       | :---
`modLoader`          | string  | **必填** | 模组使用的语言加载器。可用于支持替代语言结构，例如 Kotlin 对象作为主文件，或不同的入口点确定方法，例如接口或方法。Forge 提供了 Java 加载器 `"javafml"` 和低代码/无代码加载器 `"lowcodefml"`。 | `"javafml"`
`loaderVersion`      | string  | **必填** | 语言加载器的可接受版本范围，以 [Maven 版本范围][mvr] 表示。对于 `javafml` 和 `lowcodefml`，版本是 Forge 版本的主版本号。 | `"[46,)"`
`license`            | string  | **必填** | 此 JAR 中模组所使用的许可证。建议将其设置为你使用的 [SPDX 标识符][spdx] 和/或许可证链接。你可以访问 https://choosealicense.com/ 来帮助选择你想要使用的许可证。 | `"MIT"`
`showAsResourcePack` | boolean | `false`       | 当为 `true` 时，模组的资源将作为单独的资源包显示在 'Resource Packs' 菜单中，而不是与 'Mod resources' 包合并。 | `true`
`services`           | array   | `[]`          | 你的模组**使用**的服务数组。这是作为 Forge 实现的 Java 平台模块系统中为模组创建的模块的一部分而被消费的。 | `["net.minecraftforge.forgespi.language.IModLanguageProvider"]`
`properties`         | table   | `{}`          | 一个替换属性表。由 `StringSubstitutor` 用于将 `${file.<key>}` 替换为其对应的值。目前仅用于替换[模组特定属性][modsp]中的 `version`。 | `{ "example" = "1.2.3" }` 通过 `${file.example}` 引用
`issueTrackerURL`    | string  | *无*       | 代表报告和跟踪模组问题位置的 URL。 | `"https://forums.minecraftforge.net/"`

!!! important
    `services` 属性在功能上等同于在模块中指定 [`uses` 指令][uses]，允许[加载][serviceload]给定类型的服务。

### 模组特定属性

模组特定属性使用 `[[mods]]` 标题绑定到指定的模组。这是一个[表格数组][array]；所有键/值属性将附加到该模组，直到下一个标题。

```toml
# examplemod1 的属性
[[mods]]
modId = "examplemod1"

# examplemod2 的属性
[[mods]]
modId = "examplemod2"
```

属性              | 类型    | 默认值                 | 描述 | 示例
:---              | :---:   | :---:                   | :---:       | :---
`modId`           | string  | **必填**           | 表示此模组的唯一标识符。ID 必须匹配 `^[a-z][a-z0-9_]{1,63}$`（长度为 2-64 个字符；以小写字母开头；由小写字母、数字或下划线组成）。 | `"examplemod"`
`namespace`       | string  | `modId` 的值        | 模组的覆盖命名空间。命名空间必须匹配 `^[a-z][a-z0-9_.-]{1,63}$`（长度为 2-64 个字符；以小写字母开头；由小写字母、数字、下划线、点或短划线组成）。目前未使用。 | `"example"`
`version`         | string  | `"1"`                   | 模组的版本，最好使用 [Maven 版本控制的变体][mvnver]。当设置为 `${file.jarVersion}` 时，将替换为 JAR 清单中 `Implementation-Version` 属性的值（在开发环境中显示为 `0.0NONE`）。 | `"1.20-1.0.0.0"`
`displayName`     | string  | `modId` 的值        | 模组的友好名称。用于在屏幕上表示模组（例如模组列表、模组不匹配）。 | `"示例模组"`
`description`     | string  | `"MISSING DESCRIPTION"` | 模组列表中显示的模组描述。建议使用[多行字面量字符串][multiline]。 | `"这是一个示例。"`
`logoFile`        | string  | *无*               | 用于模组列表屏幕的图像文件的名称和扩展名。徽标必须位于 JAR 的根目录或源集的根目录中（例如 `main` 源集的 `src/main/resources`）。 | `"example_logo.png"`
`logoBlur`        | boolean | `true`                  | 是否使用 `GL_LINEAR*`（true）或 `GL_NEAREST*`（false）来渲染 `logoFile`。 | `false`
`updateJSONURL`   | string  | *无*               | 一个指向 JSON 文件的 URL，由[更新检查器][update]用于确保你正在玩的模组是最新版本。 | `"https://files.minecraftforge.net/net/minecraftforge/forge/promotions_slim.json"`
`features`        | table   | `{}`                    | 见 '[features]'。 | `{ java_version = "17" }`
`modproperties`   | table   | `{}`                    | 与此模组关联的键/值表。目前未被 Forge 使用，主要用于模组自身使用。 | `{ example = "value" }` 
`modUrl`          | string  | *无*               | 指向模组下载页面的 URL。目前未使用。 | `"https://files.minecraftforge.net/"`
`credits`         | string  | *无*               | 在模组列表屏幕上显示的模组鸣谢和致谢。 | `"那位在那里的某人。"`
`authors`         | string  | *无*               | 在模组列表屏幕上显示的模组作者。 | `"示例人物"`
`displayURL`      | string  | *无*               | 在模组列表屏幕上显示的模组展示页面 URL。 | `"https://minecraftforge.net/"`
`displayTest`     | string  | `"MATCH_VERSION"`       | 见 '[sides]'。 | `"NONE"`

#### 特性（Features）

特性系统允许模组在加载系统时要求某些设置、软件或硬件可用。当某个特性未满足时，模组加载将失败，并通知用户该需求。目前，Forge 提供以下特性：

特性            | 描述 | 示例
:---:          | :---:       | :---
`java_version` | Java 版本的可接受版本范围，以 [Maven 版本范围][mvr] 表示。这应为 Minecraft 所使用的受支持版本。 | `"[17,)"`

### 依赖配置

模组可以指定其依赖关系，Forge 在加载模组之前会检查这些依赖关系。这些配置使用[表格数组][array] `[[dependencies.<modid>]]` 创建，其中 `modid` 是依赖所属模组的标识符。

属性             | 类型    | 默认值       | 描述 | 示例
:---             | :---:   | :---:         | :---:       | :---
`modId`          | string  | **必填** | 作为依赖添加的模组的标识符。 | `"example_library"`
`mandatory`      | boolean | **必填** | 当此依赖未满足时游戏是否应崩溃。 | `true`
`versionRange`   | string  | `""`          | 语言加载器的可接受版本范围，以 [Maven 版本范围][mvr] 表示。空字符串匹配任何版本。 | `"[1, 2)"`
`ordering`       | string  | `"NONE"`      | 定义模组必须在此依赖之前（`"BEFORE"`）还是之后（`"AFTER"`）加载。如果排序不重要，返回 `"NONE"`。 | `"AFTER"`
`side`           | string  | `"BOTH"`      | 依赖必须存在的[物理端][dist]：`"CLIENT"`、`"SERVER"` 或 `"BOTH"`。 | `"CLIENT"`
`referralUrl`    | string  | *无*       | 指向依赖下载页面的 URL。目前未使用。 | `"https://library.example.com/"`

!!! warning
    两个模组的 `ordering` 可能会导致因循环依赖而崩溃：例如，模组 A 必须在模组 B `"BEFORE"` 加载，而模组 B 又必须在模组 A `"BEFORE"` 加载。

模组入口点
---------------

现在 `mods.toml` 已填写完成，我们需要提供一个入口点来开始编写模组程序。入口点本质上是执行模组的起点。入口点本身由 `mods.toml` 中使用的语言加载器决定。

### `javafml` 和 `@Mod`

`javafml` 是 Forge 为 Java 编程语言提供的语言加载器。入口点使用带有 `@Mod` 注解的公共类定义。`@Mod` 的值必须包含 `mods.toml` 中指定的模组 ID 之一。然后，所有初始化逻辑（例如[注册事件][events]、[添加 `DeferredRegister`][registration]）都可以在类的构造方法中指定。模组总线可以从 `FMLJavaModLoadingContext` 获取。

```java
@Mod("examplemod") // 必须与 mods.toml 匹配
public class Example {

  public Example() {
    // 在此处编写初始化逻辑
    var modBus = FMLJavaModLoadingContext.get().getModEventBus();

    // ...
  }
}
```

### `lowcodefml`

`lowcodefml` 是一种语言加载器，用于将数据包和资源包作为模组分发，而无需代码入口点。它指定为 `lowcodefml` 而非 `nocodefml`，是为了将来可能需要进行少量编码的小幅扩展。

[toml]: https://toml.io/
[mvr]: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html
[spdx]: https://spdx.org/licenses/
[modsp]: #模组特定属性
[uses]: https://docs.oracle.com/javase/specs/jls/se17/html/jls-7.html#jls-7.7.3
[serviceload]: https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ServiceLoader.html#load(java.lang.Class)
[array]: https://toml.io/en/v1.0.0#array-of-tables
[mvnver]: ./versioning.md
[multiline]: https://toml.io/en/v1.0.0#string
[update]: ../misc/updatechecker.md
[features]: #特性-features
[sides]: ../concepts/sides.md#编写单端模组
[dist]: ../concepts/sides.md#不同类型的端
[events]: ../concepts/events.md
[registration]: ../concepts/registries.md#deferredregister
