# ClashLite

基于 vernesong/mihomo (Alpha-Smart) 内核的飞牛 fnOS 代理管理应用，集成 metacubexd 官方管理面板，采用 Unix Socket 零端口通信。

## 功能特性

- **Smart 代理组**：基于 LightGBM 的智能节点选择（vernesong/mihomo Alpha-Smart 独有）
- **Unix Socket 零端口**：mihomo API 通过 Unix Socket 监听，不暴露任何 TCP 端口
- **统一网关访问**：通过飞牛应用中心统一网关访问 UI，无需记忆端口
- **metacubexd 前端**：自动配置后端地址，无需手动输入
- **多架构支持**：X86_64 / ARM64 双架构自动构建
- **OneSmart 配置**：内置一键智能策略组模板
- **预装数据**：CI 构建时预装 mihomo 内核 + LightGBM 模型 + GeoIP 数据，开箱即用
- **配置持久化**：配置文件存放在 data-share 共享目录，升级/重装不丢失

## 项目结构

```
.
├── .github/workflows/build-clashlite.yml  # GitHub Actions 双架构构建
├── app/
│   ├── server/
│   │   ├── config/OneSmart.yaml           # mihomo 配置模板（占位符）
│   │   ├── data/mihomo/                   # 预装内核 + 模型 + GeoData
│   │   ├── proxy/main.go                  # 网关反向代理（Go，Unix Socket 上游）
│   │   └── public/                        # metacubexd 前端文件
│   └── ui/
│       ├── config                         # 飞牛桌面入口（网关模式）
│       └── images/                        # 应用图标
├── cmd/
│   ├── main                               # 生命周期管理（mihomo + clashlite 代理）
│   ├── install_callback                   # 安装后：生成配置
│   ├── config_callback                    # 配置变更后重启
│   └── uninstall_callback                 # 卸载清理
├── config/
│   ├── privilege                          # 权限配置（run-as: package）
│   └── resource                           # data-share 共享目录
├── wizard/
│   ├── install                            # 安装向导
│   ├── config                             # 配置向导
│   └── uninstall                          # 卸载向导
└── manifest                               # 飞牛应用清单
```

## 架构说明

```
┌──────────────────────────────────────────────────────────┐
│  浏览器 → fnOS 网关(5666) → clashlite.sock              │
│                                    ↓                     │
│                          clashlite（网关反向代理）         │
│                         - 路径前缀剥离                     │
│                         - config.js 动态生成              │
│                         - 302 重定向改写                   │
│                                    ↓                     │
│                          mihomo.sock（Unix Socket）       │
│                                    ↓                     │
│  mihomo (vernesong/mihomo Alpha-Smart)                   │
│  ├─ Unix Socket: API + metacubexd 面板（零 TCP 端口）     │
│  ├─ 端口 7890/7891/7893: 代理服务                         │
│  ├─ Smart 代理组: LightGBM 智能选路                       │
│  └─ 配置: data-share/config.yaml                         │
└──────────────────────────────────────────────────────────┘
```

全程 Unix Socket 通信，mihomo API 不监听任何 TCP 端口，更安全更高效。

## 安装与使用

### 从 GitHub Release 安装

1. 前往 [Releases](https://github.com/cliii-one/FnDepot/releases) 下载对应架构的 FPK
2. 在飞牛应用中心手动安装
3. 安装向导中配置密钥和订阅源地址
4. 安装完成后自动启动，点击桌面图标打开面板

### 连接信息

| 项目 | 值 |
|------|-----|
| 访问地址 | 飞牛桌面图标或 `http://NAS-IP:5666/app/clashlite/ui` |
| 密钥 | 安装时设置（默认 `yyds666`） |

### 配置文件位置

```
/vol*/@appshare/clashlite/config.yaml
```

可通过飞牛文件管理器直接编辑，升级/重装不会丢失。

## ⚠️ 升级说明

| 操作 | 方式 |
|------|------|
| **内核升级** | ✅ 可直接在 metacubexd 面板操作（自动从 vernesong/mihomo 下载，保留 Smart 支持） |
| **面板升级** | ✅ 可直接在 metacubexd 面板操作 |

## 构建与发布

### 触发构建

在 GitHub Actions 页面手动触发 "构建 ClashLite" workflow。

### 版本号规则

| 触发方式 | 版本号 |
|---------|--------|
| 手动输入版本号 | 输入的版本号 |
| 不填版本 | 读取 manifest 默认版本 |

### 构建流程

1. 下载 mihomo Alpha-Smart 内核
2. 下载 LightGBM Model
3. 下载 GeoIP/GeoSite/ASN 数据
4. 下载 metacubexd 前端
5. 交叉编译 clashlite 网关反向代理（Go）
6. Python tarfile 打包 FPK
7. 创建 GitHub Release

## 关键文件说明

### cmd/main

管理 mihomo + clashlite 双进程：
- `copy_bundled()`：复制预装内核（cp -f 强制覆盖），首次复制模型和 GeoData
- `copy_ui()`：复制 metacubexd 到 mihomo ui 目录
- `ensure_config()`：确保配置文件存在，替换 MIHOMO_UNIX_SOCKET 占位符，删除旧 external-controller
- `start_mihomo` / `stop_mihomo`：启停 mihomo，通过 Unix Socket 文件检测就绪状态
- `start_gateway_proxy` / `stop_gateway_proxy`：启停 clashlite 网关代理

### mihomo 配置模板 (OneSmart.yaml)

基于 YYDS/OneSmart 精简版，使用占位符：
- `MIHOMO_UNIX_SOCKET` → Unix Socket 路径（启动时替换）
- `MB_SECRET` → API 密钥
- `AIRPORT_URL1` → 订阅源地址

安装时由 `install_callback` 替换为用户输入值。

### clashlite 网关反向代理 (app/server/proxy/main.go)

Go 编写的轻量反向代理，核心功能：
- **路径前缀剥离**：`/app/clashlite/ui/` → `/ui/`
- **config.js 动态生成**：注入 `window.location.origin + /app/clashlite` 作为后端地址
- **302 重定向改写**：mihomo 返回 `/ui/` → 改写为 `/app/clashlite/ui/`
- **Unix Socket 上游**：通过自定义 Transport 的 DialContext 连接 mihomo.sock

### 应用权限 (config/privilege)

```json
{
    "defaults": { "run-as": "package" },
    "username": "clashlite",
    "groupname": "clashlite"
}
```

## 维护指南

| 更新项 | 方式 |
|--------|------|
| mihomo 内核 | 修改 build-clashlite.yml 中的 `MIHOMO_TAG` |
| metacubexd | 构建时自动从 gh-pages 下载最新版 |
| 默认配置 | 编辑 `app/server/config/OneSmart.yaml` |
