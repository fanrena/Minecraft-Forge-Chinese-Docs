调试分析器
================

Minecraft 提供调试分析器来查找耗时代码。按 `F3 + L` 启动，10 秒后自动停止（或再次按 `F3 + L` 提前停止）。

结果保存在 `debug/profiling/<日期>-<世界名>-<版本号>.zip` 中。

### 读取结果
`profiling.txt` 中每行格式：`[深度] 名称 - 父级耗时% / 总耗时%`

```
[00] levels - 96.70%/96.70%
[01] |   Level Name - 99.76%/96.47%
[02] |   |   tick - 99.31%/95.81%
[03] |   |   |   entities - 47.72%/45.72%
```

### 分析自定义代码
```java
ProfilerFiller#push("sectionName");
// 要分析的代码
ProfilerFiller#pop();
```
可从 `Level`、`MinecraftServer` 或 `Minecraft` 实例获取 `ProfilerFiller`。
