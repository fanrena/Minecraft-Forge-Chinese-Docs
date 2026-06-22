游戏测试
==========

Game Test 是游戏内的单元测试系统，可并行运行大量测试。

### 创建测试
1. 创建结构模板（`.nbt` 文件），放置在 `data/<namespace>/structures`
2. 编写测试方法：`@GameTest + Consumer<GameTestHelper>`
3. 注册测试：`@GameTestHolder(modId)` 或 `RegisterGameTestsEvent`

### 关键方法
- `#succeed` / `#succeedIf` / `#succeedWhen` / `#succeedOnTickWhen` — 成功标记
- `#runAtTickTime` / `#runAfterDelay` / `#onEachTick` — 调度操作
- `#absolutePos` / `#relativePos` — 坐标转换

### 批处理
`@BeforeBatch`(setup) / `@AfterBatch`(teardown) / `@GameTest(batch="name")`

### 运行测试
`/test run <name>` / `runall` / `runthis` / `runthese` / `runfailed`

### 构建配置
- `forge.enabledGameTestNamespaces` — 启用的命名空间
- `gradlew runGameTestServer` — 自动运行并返回退出码

[test]: #running-game-tests
[buildscript]: ../gettingstarted/index.md
