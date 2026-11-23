# 快速开始（Quickstart）

从「拿到代码」到「在 iPhone 上装好 APK」的完整流程。

## 0. 前置检查

- Mac（Apple Silicon）+ **Xcode 27**（Andarium 使用 JSON 工程格式 `project.xcproj`）
- 一台 **A14+ 的 iPhone/iPad**，系统 **iOS 26+**
- 免费 Apple ID（或付费开发者账号，付费账号能解锁更高内存与扩展虚拟地址）

## 1. 克隆并准备依赖

```sh
git clone --recurse-submodules https://github.com/goodcode1024/Kernarium.git
cd Kernarium
```

按 `AGENTS.md` 准备依赖。核心构建系统是 **CMake + Ninja**（C/C++/ObjC 部分），
Apple 侧消费「已接受的预编译内核 bundle」，不需要本地跑 Kbuild 编译内核。

## 2. 准备 Android 系统镜像（二选一）

**方式 A（推荐，快）**：从 [releases](https://github.com/imxieyi/Kernarium/releases)
下载预编译镜像（"Andarium 1.1 IPA + Prebuilts"），用脚本 stage：

```sh
python3 Andarium/scripts/stage-system-image.py
```

**方式 B（慢，自建）**：需要一台高配远程 x86_64 Linux 构建机
（参照 AOSP 官方要求：约 300GB 磁盘、32GB 内存、数小时），跑 `android/` 下的构建。

## 3. 构建 App

```sh
xcodebuild \
  -project Andarium/Andarium.xcodeproj \
  -scheme Andarium \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  -derivedDataPath build \
  build
```

（本地真机调试时，在 Xcode 里选好你的 team / 签名，直接 Run 到设备。）

## 4. 侧载到设备

见 [SIDELOAD.zh-CN.md](SIDELOAD.zh-CN.md)。

## 5. 装 APK

1. 把 `.apk`（或 `.xapk` / `.apks`）放进 iOS「文件」App；
2. 打开 Andarium → 创建虚拟机（分配 CPU/内存/分辨率/存储）→ 启动，进入 Android 桌面；
3. 在设置里选 **Install APK（From Files）**，或通过内置文件浏览器找到 APK 安装；
4. 安装完成后在 Android 桌面打开运行。

## 6. 常见坑

- **首次启动慢**：系统镜像要解压初始化，别急着装 App。
- **内存别贪多**：默认 4GB；超过 4.19GB 游戏可能耗尽显存，超过 4.5GB 可能被 iOS 杀掉。
- **切后台就暂停**：这是设计行为，不是 bug。
- **依赖 Google 服务的 App 大概率用不了**：内置 LineageOS 无 GMS，可用 microG 部分替代。
- **银行/支付/风控 App 常检测模拟环境**：不要拿它当主力支付设备。
