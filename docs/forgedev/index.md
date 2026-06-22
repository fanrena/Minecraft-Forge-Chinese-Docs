开始 Forge 开发
===============

如果你决定为 Forge 本身做贡献，需要一些特殊步骤来开始。

### Fork 和克隆
1. Fork [MinecraftForge 仓库][forgerepo]
2. 克隆到本地：`git clone https://github.com/<User>/MinecraftForge`
3. 为每个 PR 创建独立分支

### 设置开发环境
- **Eclipse**：`./gradlew setup` → `./gradlew genEclipseRuns` → 导入 Gradle 项目
- **IntelliJ IDEA**：导入项目 → 运行 `setup` 任务 → 运行 `genIntellijRuns` 任务

### 修改和提交 PR
- 修改代码必须在 "Forge" 子项目中进行，不要在 "Clean" 项目中修改
- 修改 Minecraft 代码后运行 `genPatches` 生成补丁
- 通过 GitHub 提交 Pull Request

[forgerepo]: https://www.github.com/MinecraftForge/MinecraftForge
[guidelines]: ./prguidelines.md
