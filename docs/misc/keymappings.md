按键映射
================

按键映射（Key Mapping）将特定操作绑定到输入（鼠标点击、按键等）。

### 注册
在 `RegisterKeyMappingsEvent`（模组事件总线，仅物理客户端）中注册。

### 创建
```java
new KeyMapping(
  "key.examplemod.example",      // 翻译键（显示名称）
  KeyConflictContext.IN_GAME,    // 冲突上下文
  KeyModifier.SHIFT,             // 修饰键（可选）
  InputConstants.Type.KEYSYM,    // 输入类型
  GLFW.GLFW_KEY_P,               // 默认键
  "key.categories.misc"          // 分类
)
```

### 冲突上下文
- `UNIVERSAL` — 所有上下文
- `GUI` — 仅在屏幕打开时
- `IN_GAME` — 仅在屏幕未打开时

### 按键修饰符
`KeyModifier.CONTROL` / `SHIFT` / `ALT` / `NONE`

### 检查按键
- **游戏中**：在 `ClientTickEvent`（Forge 事件总线）中使用 `#consumeClick()`
- **GUI 中**：在 `#keyPressed` / `#mouseClicked` 中使用 `#isActiveAndMatches`

[controls]: https://minecraft.wiki/w/Options#Controls
[modbus]: ../concepts/events.md
