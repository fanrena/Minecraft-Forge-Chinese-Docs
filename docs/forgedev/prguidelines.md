Pull Request 指南
=======================

Forge 代码分为两类：

### 补丁（Patches）
- 直接修改 Minecraft 源码，应**尽可能小**
- 策略：插入单行触发事件/钩子，主体代码放在补丁外
- Review 时会关注补丁大小

### Forge 代码
- 普通的 Java 代码（事件、兼容性功能等），无大小限制
- Review 时会关注代码整洁度和文档

### 准则
1. **解释必要性** — 说明为什么需要这个改动，提供具体用例
2. **证明能工作** — 添加示例模组或 JUnit 测试
3. **避免破坏性变更** — 不破坏旧版本的二进制兼容性。例外：新 Minecraft 版本初期、紧急修复
4. **保持耐心** — Code review 不是针对个人

[patches]: https://github.com/MinecraftForge/MinecraftForge/wiki
[forgeenv]: ./index.md
