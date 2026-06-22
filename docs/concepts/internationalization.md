国际化与本地化
=====================================

国际化（Internationalization，简称 i18n）是一种设计代码的方式，使其无需更改即可适应多种语言。本地化（Localization）是将显示的文本适应用户语言的过程。

i18n 通过**翻译键（translation keys）**实现。翻译键是一个字符串，用于标识一段不特定于某种语言的显示文本。例如，`block.minecraft.dirt` 是指草方块名称的翻译键。这样，显示文本可以在不关心特定语言的情况下被引用。代码无需更改即可适应新语言。

本地化将在游戏的语言环境中进行。在 Minecraft 客户端中，语言环境由语言设置指定。在专用服务器上，唯一支持的语言环境是 `en_us`。可用的语言环境列表可以在 [Minecraft Wiki][langs] 上找到。

语言文件
--------------

语言文件位于 `assets/[namespace]/lang/[locale].json`（例如，`examplemod` 提供的所有美式英语翻译都在 `assets/examplemod/lang/en_us.json` 中）。文件格式是一个从翻译键到值的 JSON 映射。文件必须使用 UTF-8 编码。旧的 .lang 文件可以使用[转换器][converter]转换为 json。

```js
{
  "item.examplemod.example_item": "Example Item Name",
  "block.examplemod.example_block": "Example Block Name",
  "commands.examplemod.examplecommand.error": "Example Command Errored!"
}
```

在方块和物品中的使用
---------------------------

Block、Item 以及其他一些 Minecraft 类具有用于显示其名称的内置翻译键。这些翻译键通过重写 `#getDescriptionId` 来指定。Item 还有 `#getDescriptionId(ItemStack)`，可以重写以根据 ItemStack NBT 提供不同的翻译键。

默认情况下，`#getDescriptionId` 会返回 `block.` 或 `item.` 后接方块或物品的注册名称，冒号替换为点。`BlockItem` 默认重写了此方法以使用其对应 `Block` 的翻译键。例如，一个 ID 为 `examplemod:example_item` 的物品实际上需要语言文件中包含以下行：

```js
{
  "item.examplemod.example_item": "Example Item Name"
}
```

!!! note
    翻译键的唯一用途是国际化。不要将其用于逻辑判断。请使用注册名称。

本地化方法
--------------------

!!! warning
    一个常见的问题是让服务器为客户端进行本地化。服务器只能在其自身的语言环境中进行本地化，这不一定会与连接的客户端的语言环境匹配。

    为了尊重客户端的语言设置，服务器应让客户端使用 `TranslatableComponent` 或其他保留语言中立翻译键的方法，在其自身的语言环境中进行本地化。

### `net.minecraft.client.resources.language.I18n`（仅客户端）

**此 I18n 类仅在 Minecraft 客户端上存在！** 它旨在供仅在客户端运行的代码使用。在服务器上尝试使用它会抛出异常并崩溃。

- `get(String, Object...)` 在客户端的语言环境中进行本地化并支持格式化。第一个参数是翻译键，其余参数是 `String.format(String, Object...)` 的格式化参数。

### `TranslatableContents`

`TranslatableContents` 是一个惰性本地化和格式化的 `ComponentContents`。它在向玩家发送消息时非常有用，因为它会自动在玩家自己的语言环境中进行本地化。

`TranslatableContents(String, Object...)` 构造方法的第一个参数是翻译键，其余参数用于格式化。唯一支持的格式说明符是 `%s` 和 `%1$s`、`%2$s`、`%3$s` 等。格式化参数可以是 `Component`，它们将插入到最终的格式化文本中，并保留其所有属性。

`MutableComponent` 可以通过传入 `TranslatableContents` 的参数，使用 `Component#translatable` 创建。也可以通过传入 `ComponentContents` 本身，使用 `MutableComponent#create` 创建。

### `TextComponentHelper`

- `createComponentTranslation(CommandSource, String, Object...)` 根据接收者创建一个本地化并格式化的 `MutableComponent`。如果接收者是原版客户端，则立即进行本地化和格式化。如果不是，则使用包含 `TranslatableContents` 的 `Component` 进行惰性本地化和格式化。这仅在服务器应允许原版客户端连接时有用。

[langs]: https://minecraft.wiki/w/Language#Languages
[converter]: https://tterrag.com/lang2json/
