# FnStore

飞牛 fnOS 应用合集，统一管理、统一构建。

## 包含应用

| 应用 | 说明 | 最新版本 |
|------|------|----------|
| [ClashLite](ClashLite/) | 基于 mihomo 内核的代理管理应用 | [![ClashLite](https://img.shields.io/github/v/release/cliii-one/FnStore?filter=clashlite-v*&label=&color=blue)](https://github.com/cliii-one/FnStore/releases) |
| [SubStore](SubStore/) | 基于 Sub-Store 的订阅管理应用 | [![SubStore](https://img.shields.io/github/v/release/cliii-one/FnStore?filter=substore-v*&label=&color=blue)](https://github.com/cliii-one/FnStore/releases) |
| [fnnas-notes](fnnas-notes/) | 贴贴密笺 - 轻量安全的本地便签应用 | [![fnnas-notes](https://img.shields.io/github/v/release/cliii-one/FnStore?filter=fnnas-notes-v*&label=&color=blue)](https://github.com/cliii-one/FnStore/releases) |
| [MediaHub](MediaHub/) | Emby + 115 云盘一站式媒体管理面板 | [![MediaHub](https://img.shields.io/github/v/release/cliii-one/FnStore?filter=mediahub-v*&label=&color=blue)](https://github.com/cliii-one/FnStore/releases) |
| [OneServer](OneServer/) | 飞牛 NAS SSH 终端管理工具 | [![OneServer](https://img.shields.io/github/v/release/cliii-one/FnStore?filter=oneserver-v*&label=&color=blue)](https://github.com/cliii-one/FnStore/releases) |

## 构建

每个应用有独立的构建工作流，通过对应标签触发：

```bash
# 构建 ClashLite
git tag clashlite-v1.0.0 && git push origin clashlite-v1.0.0

# 构建 SubStore
git tag substore-v1.0.0 && git push origin substore-v1.0.0
```

也可以在 Actions 页面手动触发。
