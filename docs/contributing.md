贡献本文档
==================================

你可以通过 [GitHub] 上的 PR 进行贡献。

本文档旨在提供说明性内容。请解释如何操作，并将其拆分为合理的模块。
我们另有 Wiki 站点可以收录更全面的代码示例。

我们的受众是所有想要了解如何使用 Forge 构建模组的开发者。

请不要试图将本文档变成 Java 开发教程——它的目标读者是已经了解 Java 类以及其他 Java 基础结构的开发者。

风格指南
-----------

!!! important
    请使用**两个空格**进行缩进，而不是制表符。

标题应使用标准的标题格式进行大写。例如：

* Guide For Contributing to This Documentation
* Building and Testing Your Mod

基本上，除了非重要的单词外，所有单词都应大写。

拼写、语法和句法应遵循美式英语规范。此外，建议使用分立的单词而非缩略形式（例如使用"are not"而不是"aren't"）。

请使用等号和短划线作为标题下划线，而不是 `#` 和 `##`。对于 h3 及更低级别，可以使用 `###` 等标记。本文件的源代码包含了等号和短划线标题下划线的示例。等号下划线表示 h1 标题，短划线下划线表示 h2 标题。

在代码片段之外引用字段和方法时，应使用 `#` 分隔符（例如 `ClassName#methodName`）。内部类应使用 `$` 分隔符（例如 `ClassName$InnerClassName`）。

JSON 代码片段应使用 `js` 语法高亮。

所有链接的位置应在页面底部指定。任何内部链接应通过相对路径引用页面。

警告块（以 `!!! <type>` 表示）必须按照[文档][admonition]中的格式编写；否则可能导致渲染错误。

[GitHub]: https://github.com/MinecraftForge/Documentation
[admonition]: https://python-markdown.github.io/extensions/admonition/
