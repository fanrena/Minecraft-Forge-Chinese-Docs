资源
=========

资源是游戏使用的额外数据，存储在数据文件中，而不是代码中。
Minecraft 有两个主要的资源系统：逻辑客户端上的资源系统用于视觉内容，如模型、纹理和本地化，称为 `assets`；逻辑服务器上的资源系统用于游戏玩法内容，如配方和战利品表，称为 `data`。
[资源包][respack]控制前者，而[数据包][datapack]控制后者。

在默认的模组开发套件中，assets 和 data 目录位于项目的 `src/main/resources` 目录下。

当启用多个资源包或数据包时，它们会被合并。通常，包栈顶部的文件会覆盖下面的文件；但对于某些文件，如本地化文件和标签，数据实际上会进行内容层面的合并。模组在其 `resources` 目录中定义资源和数据包，但它们被视为"模组资源"包的子集。模组资源包不能被禁用，但可以被其他资源包覆盖。模组数据包可以使用原版的 `/datapack` 命令禁用。

所有资源应使用蛇形命名法（snake case）的路径和文件名（小写，使用 "_" 分隔单词），这在 1.11 及以上版本中是强制要求的。

`ResourceLocation`
------------------

Minecraft 使用 `ResourceLocation` 来标识资源。一个 `ResourceLocation` 包含两部分：命名空间（namespace）和路径（path）。它通常指向 `assets/<namespace>/<ctx>/<path>` 处的资源，其中 `ctx` 是取决于 `ResourceLocation` 使用方式的上下文相关路径片段。当 `ResourceLocation` 以字符串形式写入/读取时，它表示为 `<namespace>:<path>`。如果省略命名空间和冒号，则在将字符串读入 `ResourceLocation` 时，命名空间将默认为 `"minecraft"`。模组应将其资源放入与其模组 ID 同名的命名空间中（例如，ID 为 `examplemod` 的模组应将其资源分别放入 `assets/examplemod` 和 `data/examplemod`，指向这些文件的 `ResourceLocation` 看起来像 `examplemod:<path>`）。这不是强制要求，在某些情况下，使用不同的（甚至多个）命名空间可能更可取。`ResourceLocation` 也在资源系统之外使用，因为它恰好是唯一标识对象的好方法（例如[注册表][]）。

[respack]: ../resources/client/index.md
[datapack]: ../resources/server/index.md
[registries]: ./registries.md
