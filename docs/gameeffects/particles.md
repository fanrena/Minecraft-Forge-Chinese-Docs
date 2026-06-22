粒子
=========

粒子是游戏中用于提升沉浸感的特效。

粒子系统分为四个主要组件：

| 类 | 端 | 描述 |
| :--- | :---: | :--- |
| `ParticleType` | 双端 | 粒子类型的注册对象，用于引用粒子 |
| `ParticleOptions` | 双端 | 用于从网络或命令同步粒子数据的数据持有者 |
| `ParticleProvider` | CLIENT | 工厂，由 `ParticleType` 注册，用于从 `ParticleOptions` 构造 `Particle` |
| `Particle` | CLIENT | 在客户端渲染的可渲染逻辑 |

### ParticleType
必须[注册][registration]。接受两个参数：`overrideLimiter`（是否无视距离渲染）和 `ParticleOptions$Deserializer`。简单粒子可继承 `SimpleParticleType`。

### ParticleOptions
包含三种方法：`getType`（获取 ParticleType）、`writeToNetwork`（写入网络缓冲区）、`writeToString`（写入字符串）。`SimpleParticleType` 同时实现了 `ParticleType` 和 `ParticleOptions`。

### Particle
实现 `render` 和 `getRenderType`。常用子类 `TextureSheetParticle`。
- `ParticleRenderType`：`TERRAIN_SHEET` / `PARTICLE_SHEET_OPAQUE` / `PARTICLE_SHEET_TRANSLUCENT` / `PARTICLE_SHEET_LIT` / `CUSTOM` / `NO_RENDER`

### ParticleProvider
在 `RegisterParticleProvidersEvent`（模组事件总线）中注册。
- `#registerSpecial` — 注册自定义 `ParticleProvider`
- `#registerSpriteSet` — 注册带精灵表的粒子（需在 `assets/<modid>/particles/<name>.json` 中定义纹理列表）

```js
// assets/mymod/particles/my_particle.json
{ "textures": [ "mymod:particle_texture", "mymod:particle_texture2" ] }
```

### 生成粒子
- `ClientLevel`：`#addParticle` / `#addAlwaysVisibleParticle`
- `ServerLevel`：`#sendParticles`（发送数据包到客户端）

[sides]: ../concepts/sides.md
[registration]: ../concepts/registries.md
