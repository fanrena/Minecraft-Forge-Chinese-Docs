Forge 入门指南
==========================

如果你从未制作过 Forge 模组，本节将提供搭建 Forge 开发环境所需的最基本信息。本文档的其他部分将介绍后续的学习方向。

先决条件
-------------

* 安装 Java 17 开发工具包（JDK）和 64 位 Java 虚拟机（JVM）。Forge 推荐并官方支持 [Eclipse Temurin][jdk]。

    !!! warning
        确保你使用的是 64 位 JVM。验证方法之一是在终端中运行 `java -version`。使用 32 位 JVM 会在使用 [ForgeGradle] 时导致一些问题。

* 熟悉集成开发环境（IDE）。
    * 建议使用带有 Gradle 集成的 IDE。

从零开始模组开发
--------------------

1. 从 [Forge 文件站点][files] 下载模组开发工具包（MDK）：点击 'Mdk'，等待一段时间后点击右上角的 'Skip' 按钮。建议尽可能下载最新版本的 Forge。
1. 将下载的 MDK 解压到一个空目录中。这将作为你的模组目录，其中应包含一些 Gradle 文件和一个包含示例模组的 `src` 子目录。

    !!! note
        以下一些文件可以在不同模组之间重复使用：

        * `gradle` 子目录
        * `build.gradle`
        * `gradlew`
        * `gradlew.bat`
        * `settings.gradle`

        `src` 子目录不需要在工作区之间复制；不过，如果之后创建了 java（`src/main/java`）和资源（`src/main/resources`）目录，可能需要刷新 Gradle 项目。

1. 打开你选择的 IDE：
    * Forge 仅明确支持在 Eclipse 和 IntelliJ IDEA 上进行开发，但 Visual Studio Code 也有额外的运行配置。尽管如此，从 Apache NetBeans 到 Vim/Emacs 的任何环境都可以使用。
    * Eclipse 和 IntelliJ IDEA 的 Gradle 集成（默认已安装并启用）将在导入或打开时处理其余的工作区初始设置。这包括从 Mojang、MinecraftForge 等处下载必要的包。Visual Studio Code 需要 'Gradle for Java' 插件才能实现相同的功能。
    * 对于几乎所有对其关联文件（例如 `build.gradle`、`settings.gradle` 等）的更改，都需要调用 Gradle 来重新评估项目。某些 IDE 带有 'Refresh' 按钮来实现此操作；也可以通过终端使用 `gradlew` 完成。
1. 为你选择的 IDE 生成运行配置：
    * **Eclipse**：运行 `genEclipseRuns` 任务。
    * **IntelliJ IDEA**：运行 `genIntellijRuns` 任务。如果出现 "module not specified" 错误，请将 [`ideaModule` 属性][config] 设置为主模块（通常为 `${project.name}.main`）。
    * **Visual Studio Code**：运行 `genVSCodeRuns` 任务。
    * **其他 IDE**：你可以直接使用 `gradle run*` 运行配置（例如 `runClient`、`runServer`、`runData`、`runGameTestServer`）。这些也可以与支持的 IDE 一起使用。

自定义模组信息
--------------------------------

编辑 `build.gradle` 文件以自定义模组的构建方式（例如文件名、工件版本等）。

!!! important
    除非你知道自己在做什么，否则**不要**编辑 `settings.gradle`。该文件指定了 [ForgeGradle] 上传到的仓库。

### 推荐的 `build.gradle` 自定义设置

#### 模组 ID 替换

将所有出现的 `examplemod`（包括 [`mods.toml` 和主模组文件][modfiles]）替换为你的模组 ID。这还包括通过设置 `base.archivesName` 更改构建文件的名称（通常设置为你的模组 ID）。

```gradle
// 在 build.gradle 中
base.archivesName = 'mymod'
```

#### 组 ID

`group` 属性应设置为你的[顶级包][packaging]，它应该是你拥有的域名或你的电子邮件地址：

类型      | 值                | 顶级包
:---:     | :---:             | :---
域名      | example.com       | `com.example`
子域名    | example.github.io | `io.github.example`
电子邮件  | example@gmail.com | `com.gmail.example`

```gradle
// 在 build.gradle 中
group = 'com.example'
```

Java 源代码（`src/main/java`）中的包现在也应遵循此结构，并使用一个内部包表示模组 ID：

```text
com
- example（在 group 属性中指定的顶级包）
  - mymod（模组 ID）
    - MyMod.java（重命名后的 ExampleMod.java）
```

#### 版本

将 `version` 属性设置为当前模组版本。我们建议使用 [Maven 版本控制的一种变体][mvnver]。

```gradle
// 在 build.gradle 中
version = '1.20-1.0.0.0'
```

### 额外配置

其他配置可以在 [ForgeGradle] 文档中找到。

构建和测试你的模组
-----------------------------

1. 要构建模组，运行 `gradlew build`。默认情况下，这将在 `build/libs` 中输出一个名为 `[archivesBaseName]-[version].jar` 的文件。此文件可以放入已安装 Forge 的 Minecraft 设置的 `mods` 文件夹中进行分发。
1. 要在测试环境中运行模组，你可以使用生成的运行配置或使用关联的任务（例如 `gradlew runClient`）。这将从运行目录（默认为 'run'）启动 Minecraft，并附带指定的源集。默认 MDK 包含 `main` 源集，因此在 `src/main/java` 中编写的任何代码都将生效。
1. 如果你正在运行专用服务器，无论是通过运行配置还是 `gradlew runServer`，服务器最初都会立即关闭。你需要通过编辑运行目录中的 `eula.txt` 文件来接受 Minecraft EULA。接受后，服务器将加载，然后可以通过直接连接到 `localhost` 进行访问。

!!! note
    你应该始终在专用服务器环境中测试你的模组。这包括[仅客户端的模组][client]，因为它们在服务器上加载时不应执行任何操作。

[jdk]: https://adoptium.net/temurin/releases?version=17 "Eclipse Temurin 17 预构建二进制文件"
[ForgeGradle]: https://docs.minecraftforge.net/en/fg-6.x

[files]: https://files.minecraftforge.net "Forge 文件分发站点"
[config]: https://docs.minecraftforge.net/en/fg-6.x/configuration/runs/

[modfiles]: ./modfiles.md
[packaging]: ./structuring.md#包结构
[mvnver]: ./versioning.md
[client]: ../concepts/sides.md#编写单端模组
