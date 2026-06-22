Forge 更新检查器
====================

Forge 提供轻量的可选更新检查框架。如果有可用更新，主菜单的 'Mods' 按钮和模组列表会显示闪烁图标。**不会**自动下载更新。

### 配置
在 `mods.toml` 中设置 `updateJSONURL` 指向更新 JSON 文件。

### 更新 JSON 格式
```js
{
  "homepage": "<下载页面>",
  "<mcversion>": {
    "<modversion>": "<更新日志>"
  },
  "promos": {
    "<mcversion>-latest": "<modversion>",
    "<mcversion>-recommended": "<modversion>"
  }
}
```

### 获取检查结果
`VersionChecker#getResult(IModInfo)`：
- `FAILED` — 无法连接
- `UP_TO_DATE` — 当前版本等于推荐版本
- `OUTDATED` — 有新推荐或最新版本
- `BETA_OUTDATED` — 有新的最新版本
- `PENDING` — 检查尚未完成

[mvnver]: ../gettingstarted/versioning.md
