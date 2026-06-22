声音
======

术语：
- **Sound Event** — 触发声音效果的事件，如 `minecraft:block.anvil.hit`
- **Sound Category** — 声音类别（`player`、`block`、`master` 等），对应设置中的滑块
- **Sound File** — 实际的 .ogg 音频文件

`sounds.json` 位于 `assets/<namespace>/sounds.json`，定义声音事件及其关联的音频文件。

```js
{
  "open_chest": {
    "subtitle": "mymod.subtitle.open_chest",
    "sounds": [ "mymod:open_chest_sound_file" ]
  },
  "epic_music": {
    "sounds": [ { "name": "mymod:music/epic_music", "stream": true } ]
  }
}
```
- 长音频（背景音乐、唱片）应使用 `stream: true` 避免加载到内存
- 音频文件位于 `assets/<namespace>/sounds/<path>.ogg`
- `sounds.json` 可通过[数据生成][datagen]生成

### SoundEvent
必须在代码中创建并[注册][registration] `SoundEvent`，作为在服务器端播放声音的引用。

### 播放声音方法
- `Level#playSound(Player, x, y, z, SoundEvent, source, vol, pitch)` — 服务端：播放给附近**除**传入玩家外的所有人；客户端：仅播放给传入的客户端玩家
- `Level#playLocalSound(x, y, z, SoundEvent, source, vol, pitch, distDelay)` — **仅客户端**，用于自定义数据包或客户端特效
- `Entity#playSound(SoundEvent, vol, pitch)` — 服务端：在实体位置播放给所有人。客户端：无操作
- `Player#playSound(SoundEvent, vol, pitch)` — 服务端：播放给除自己外的附近玩家。客户端：见 `LocalPlayer`

[loc]: ../concepts/resources.md
[wiki]: https://minecraft.wiki/w/Sounds.json
[datagen]: ../datagen/client/sounds.md
[registration]: ../concepts/registries.md
