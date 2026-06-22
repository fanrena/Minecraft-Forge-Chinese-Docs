声音定义生成
===========================

通过继承 `SoundDefinitionsProvider` 并实现 `#registerSounds` 生成 `sounds.json`。

```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeClient(),
        output -> new MySoundDefinitionsProvider(output, MOD_ID, event.getExistingFileHelper())
    );
}
```

### 方法
- `#add(soundName, definition)` — 添加声音定义
- `#definition()` — 创建 `SoundDefinition`
  - `.with(sound...)` — 添加声音文件
  - `.subtitle(key)` — 设置翻译键
  - `.replace(true)` — 替换而非追加
- `#sound(location, type)` — 创建 `SoundDefinition$Sound`
  - `type`：`SOUND`（声音文件）或 `EVENT`（其他声音事件）
  - `.volume(v)` / `.pitch(p)` / `.weight(w)` / `.stream()` / `.attenuationDistance(d)` / `.preload()`

```java
this.add(EXAMPLE_SOUND_EVENT, definition()
  .subtitle("sound.examplemod.example_sound")
  .with(
    sound(new ResourceLocation(MODID, "example_sound_1")).weight(4).volume(0.5),
    sound(new ResourceLocation(MODID, "example_sound_2")).stream()
  )
);
```

[datagen]: ../index.md
[soundevent]: ../../gameeffects/sounds.md
