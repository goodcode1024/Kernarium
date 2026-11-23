# 构建与侧载指南（Andarium）

> 一句话结论：**能完整运行 APK 的 App 就是 Andarium**，它把 LineageOS 24（Android 17）整机打包进 App（约 1.2GB），首次打开即用、无需下载、无需传统虚拟机。这里给你三条从快到慢的路径。

## 三条路径对比

| 路径 | 耗时 | 需要 Mac/Xcode | 说明 |
|------|------|:---:|------|
| **① 直接用上游预编译 IPA** | 5 分钟 | ❌ | 下载 → AltStore/SideStore 侧载，立即可用 |
| **② 用侧载源（自动更新）** | 3 分钟 | ❌ | AltStore/SideStore 里加一个源，App 自动出现 |
| **③ 从源码自己构建** | 数小时 | ✅（Xcode 27） | 定制/改图标改名的唯一路径 |

## 路径①：直接侧载预编译 IPA

上游 release 已发布成品（`Andarium-1.1-16.ipa`，约 1.16GB）：

```
https://github.com/imxieyi/Kernarium/releases/download/andarium-1.1/Andarium-1.1-16.ipa
```

1. 在 iPhone/iPad 上下载这个 IPA（需要 iOS 26+，A14 及以上芯片）。
2. 打开 AltStore 或 SideStore，在「My Apps」点 `+` 选这个 IPA。
3. 用你的 Apple ID 签名安装（免费 Apple ID 7 天一签，需定期刷新）。

## 路径②：添加侧载源（推荐）

在 AltStore / SideStore 里添加「Sources」源，之后 App 和更新都会自动出现：

```
https://raw.githubusercontent.com/goodcode1024/Kernarium/main/sideload/source.json
```

（或上游原版源：`https://raw.githubusercontent.com/imxieyi/Kernarium/main/sideload/source.json`）

## 路径③：从源码构建（定制）

### 硬前提

- **Xcode 27**：Andarium 用 Xcode 27 的 JSON 工程格式（`project.xcproj`），旧版 Xcode 打不开。
- **CMake ≥ 3.27 + Ninja**
- Apple Silicon Mac（构建 target 是 arm64 iOS；x86 Mac 编译不了 device 切片）。
- 磁盘 ≥ 15GB（含 1GB 系统镜像 + C++ 运行库产物）。
- 依赖预编译静态前缀需联网下载（`dependencies/prepare.sh`）。

### 官方权威构建序列

```sh
# 1) 拉依赖（仓库唯一允许联网的入口）
./dependencies/prepare.sh

# 2) 配置 + 编译 Android 运行库（C++ 内核宿主，仅 Android，无需 Linux rootfs 构建）
cmake --preset kernarium-ios-device-android
cmake --build   --preset kernarium-ios-device-android

# 3) 打包运行库框架到 .build/kernarium-xcframeworks
Kernarium/scripts/package-ios-frameworks.sh --runtimes android

# 4) 下载并把「已接受的内核 bundle + 系统镜像」放进 .build/ 对应目录
#    内核 bundle → .build/kernel/artifacts/android-6.18-arm64-4k-20261119-6.18.60-ca1/
#    系统镜像    → .build/android/releases/lineage24-mainline6.18-4k-20261121-los24-r29/
#    （见下方「镜像资产」）

# 5) 把系统镜像 stage 进 App（内置镜像、即开即用）
Andarium/scripts/stage-system-image.py

# 6) 用 Xcode 构建 App
xcodebuild -project Andarium/Andarium.xcodeproj -scheme Andarium \
    -destination 'generic/platform=iOS' build
```

### 镜像资产（release `andarium-1.1`）

| 资产 | 大小 | 解压目标目录（accepted-*.env 指定） |
|------|------|------|
| `android-6.18-arm64-4k-20261119-6.18.60-ca1.tar.gz` | 11MB | `.build/kernel/artifacts/android-6.18-arm64-4k-20261119-6.18.60-ca1/` |
| `lineage24-mainline6.18-4k-20261121-los24-r29.tar.gz` | 1.09GB | `.build/android/releases/lineage24-mainline6.18-4k-20261121-los24-r29/` |

> 目录名必须与 `android/lineage-24/metadata/accepted-release.env`、`kernel/mainline-6.18/artifacts/accepted-android-4k.env` 里的 `*_DIRECTORY_NAME` 完全一致，否则 `stage-system-image.py` 找不到镜像。

### 签名

仓库刻意不签名。构建前复制并填写你的 Team ID：

```sh
cp GuestHost/Config/Signing.local.xcconfig.example GuestHost/Config/Signing.local.xcconfig
# 在文件里设置 DEVELOPMENT_TEAM = <你的 Team ID>
```

## GitHub Actions

- `.github/workflows/fetch-prebuilt.yml`：在托管 runner 上直接抓取上游预编译 IPA 并重托管（**无需 Mac，今天就能跑**）。
- `.github/workflows/build.yml`：完整源码构建，默认跑在**自建 runner**（`self-hosted, macOS, ARM64`）上，因为托管 runner 没有 Xcode 27。在 Mac 上挂一个 runner 即可。

## 硬门槛（务必知晓）

1. **iOS 26+、A14+ 芯片**（图形走 Metal 4，老机型起不来）。
2. **只能前台运行**：切后台即暂停（App 无后台模式）。
3. 免费 Apple ID 侧载下内存受限；7 天一签。
4. 性能约等于入门安卓机，3D 游戏吃力。
5. 无 Google Play；银行/支付类 App 常检测到非真机环境而拒绝运行。
6. JIT 需通过 StikDebug 启用（可选，加速）。

## 定制成你自己的 App（改名/换图标）

本仓库即完整可构建源码。要"做自己的 App"：

1. 改 `Andarium/` 下的 `PRODUCT_BUNDLE_IDENTIFIER`、显示名、图标（`Assets.xcassets`）。
2. 保持 GPL-2.0 兼容（派生作品需继续开源）。
3. 按上面的序列重新构建。
