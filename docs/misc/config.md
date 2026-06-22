配置
=============

Forge 使用基于 TOML 的配置系统，由 NightConfig 库读取。

### 创建配置
使用 `ForgeConfigSpec$Builder` 构建：
- `#push(name)` / `#pop()` — 配置分区
- `#build()` — 构建 `ForgeConfigSpec`
- `#configure(constructor)` — 构建并绑定到持有类

### 配置值
- `#define(path, defaultValue)` — 基本值
- `#defineInRange(path, min, max, type)` — 范围值
- `#defineInList(path, allowedValues, type)` — 白名单值
- `#defineList(path, validator)` — 列表值
- `#defineEnum(path, enumGetter, allowedValues)` — 枚举值
- 上下文方法：`.comment()`、`.translation()`、`.worldRestart()`

### 注册配置
在模组构造函数中：`ModLoadingContext.get().registerConfig(type, spec, fileName)`

配置类型：
| 类型 | 加载端 | 同步客户端 | 默认文件后缀 |
|:---:|:---:|:---:|:---:|
| CLIENT | 仅客户端 | 否 | `-client` |
| COMMON | 双端 | 否 | `-common` |
| SERVER | 仅服务端 | 是 | `-server` |

### 配置事件
`ModConfigEvent$Loading` / `ModConfigEvent$Reloading`（模组事件总线）

[toml]: https://toml.io/
[nightconfig]: https://github.com/TheElectronWill/night-config
