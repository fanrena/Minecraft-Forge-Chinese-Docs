访问转换器
===================

Access Transformer（简称 AT）用于扩大类、方法和字段的可见性以及修改 `final` 标志。允许模组访问和修改其他类中原本不可访问的成员。

完整规范见 [Minecraft Forge GitHub][specs]。

### 添加 AT
在 `build.gradle` 中添加一行：
```groovy
minecraft {
  accessTransformer = file('src/main/resources/META-INF/accesstransformer.cfg')
}
```
添加或修改后需刷新 Gradle 项目。生产环境中 Forge 仅搜索 `META-INF/accesstransformer.cfg`。

### 语法
```
# 注释（# 开头）
<access modifier> <fully qualified class>
<access modifier> <fully qualified class> <field name>
<access modifier> <fully qualified class> <method name>(<params>)<return>
```

- 访问修饰符：`public` > `protected` > `default` > `private`
- `+f` / `-f`：添加/移除 `final` 标志
- 字段和方法使用 **SRG 名称**
- 内部类用 `$` 分隔

### 类型描述符
`B`=byte, `C`=char, `D`=double, `F`=float, `I`=int, `J`=long, `S`=short, `Z`=boolean, `[`=数组, `L<类>;`=引用类型, `V`=void

### 示例
```
# public 化 Crypt 中的 ByteArrayToKeyFunction 接口
public net.minecraft.util.Crypt$ByteArrayToKeyFunction

# protected 化并移除 MinecraftServer 中 random 的 final
protected-f net.minecraft.server.MinecraftServer f_129758_

# public 化 Util 中的 makeExecutor 方法
public net.minecraft.Util m_137477_(Ljava/lang/String;)Ljava/util/concurrent/ExecutorService;
```

!!! warning
    AT 只修改直接引用的方法，重写方法不会被转换。安全的转换目标：`private`方法、`final`方法（或 `final` 类中的方法）、`static`方法。

[specs]: https://github.com/MinecraftForge/AccessTransformers/blob/master/FMLAT.md
