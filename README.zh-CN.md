# 中文说明（Fork 版）

> 本仓库是 [imxieyi/Kernarium](https://github.com/imxieyi/Kernarium) 的 fork。
> 目标是：在 iPhone/iPad 上**本地运行 Android 17（LineageOS 24）并安装运行 APK**，
> 走 **User-mode Linux** 架构（内核原生执行 + 用户态解释器/JIT + Vulkan→Metal），
> **不需要传统虚拟机（QEMU/Hypervisor）**，也不上架 App Store（侧载 + JIT）。

## 一、这个 fork 与上游的关系

| 目录 | 作用 |
|------|------|
| `Andarium/` | 面向普通用户的 Android 虚拟机 App（iOS，已上架后被 Apple 下架，可侧载） |
| `Kernarium/` | 通用 Linux 模拟器 App（开发中） |
| `GuestHost/` | App 与 runtime 共享的宿主代码（runtime loader、显示/触控视图、着色器缓存） |
| `darwin/` `kernel/` `android/` | runtime 核心：UML 内核、CPU 解释器/JIT、图形栈、Android 镜像构建 |
| `docs/` | 架构、图形、测试等文档 |
| `sideload/` | AltStore / SideStore 侧载源 |

## 二、硬性要求

- **A14 或更新芯片**（iPhone 12 及以后，或 A14/M 系列 iPad）
- **iOS 26 及以上**（侧载；App Store 版当时要求 iOS 27）
- 约 **10GB** 可用存储；App 本体约 1.2GB
- 虚拟机**只能前台运行**（切后台自动暂停）

## 三、快速开始

见 [docs/QUICKSTART.zh-CN.md](docs/QUICKSTART.zh-CN.md)（构建 + 侧载 + 装 APK 全流程）。

## 四、侧载（不上架）

本项目**已从 App Store 下架**，只能侧载。两种方式：

1. **加源（推荐）**：在 AltStore / SideStore 里添加
   `https://raw.githubusercontent.com/imxieyi/Kernarium/main/sideload/source.json`
2. **手动装 IPA**：从 [releases](https://github.com/imxieyi/Kernarium/releases) 下载
   `Andarium-<version>.ipa`（约 1GB），在 AltStore/SideStore 里用 `+` 安装。

**启用 JIT（性能关键）**：在 [StikDebug](https://github.com/stikdebug/stikdebug) 里把
universal 脚本指向 Andarium，之后从 StikDebug 启动。否则走纯解释器，速度明显更慢。

详见 [docs/SIDELOAD.zh-CN.md](docs/SIDELOAD.zh-CN.md)。

## 五、定制开发

想把它改造成「自己的 App」（换 UI、一键装 APK、加启动器等），见
[docs/CUSTOMIZATION.md](docs/CUSTOMIZATION.md) 与 [ROADMAP.md](ROADMAP.md)。

## 六、许可证

GPL-2.0-or-later（详见 `LICENSES/`）。基于本仓库做**闭源商业产品**需注意 copyleft 传染性。

> 独立项目，与 Google、Apple、LineageOS 均无隶属关系。
