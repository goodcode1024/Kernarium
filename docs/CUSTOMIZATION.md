# 定制开发指南（把 Andarium 改造成"自己的 App"）

## 一、先想清楚：不要从零造轮子

Andarium 能跑 APK 的根基是 **Kernarium runtime**（User-mode Linux 内核 +
CPU 解释器/JIT + Vulkan→Metal 图形栈），这是 9000+ commit、经年累月的工程。
**重写它不可行**，正确姿势是复用 runtime，只重做 App 层。

## 二、代码地图（改哪里）

```
Andarium/
├── Andarium/            # App 目标
│   ├── Shared/          # 两个 target 共用的界面与模型
│   ├── Live/            # 真机版的 runtime、存储、会话
│   └── Preview/         # 模拟器预览（mock 数据）
├── AndariumCore/        # Swift Package：模型 + ADB 客户端（无 UIKit），re-export GuestHostCore
├── Config/              # entitlements、Info.plist、隐私清单、构建设置
└── scripts/             # 系统镜像 staging、截图、本地化
GuestHost/
├── Sources/             # 与 Kernarium 共享的宿主代码：runtime loader、显示/触控视图、zstd、着色器缓存
└── Core/
Kernarium/               # 通用 Linux 模拟器 App（runtime 的开发者入口）
```

**关键分层**：`AndariumCore`（纯模型/逻辑，无 UI）→ `Andarium/Shared`（界面）→ `GuestHost`（runtime 桥接）。

## 三、常见改造方向与切入点

| 想做什么 | 改哪里 | 难度 |
|----------|--------|------|
| 换 UI / 品牌 / 语言 | `Andarium/Shared` 的 SwiftUI 视图、`Localization/` | 低 |
| 双击 APK 一键安装 | `Andarium/Shared` + `AndariumCore` 的 APK 处理，接 `UIDocumentPickerViewController` | 中 |
| 自建"已装应用启动器" | 读 ADB 输出的已装应用列表 + 图标，写 Launcher 视图 | 中 |
| 加性能档位 | `Andarium/Shared` 设置 + 映射到 VM 资源参数 | 中 |
| 集成 F-Droid / microG | `android/` 镜像层 + 安装流程 | 高 |
| 改 runtime 本身（新指令/新设备） | `darwin/` `kernel/`，严格按 `AGENTS.md` 契约 | 极高 |

## 四、工程规范（必须遵守）

1. **可执行内存只有一个入口**：`darwin/cpu/jit_memory/executable_memory.cpp`
   是全仓库唯一允许分配可执行内存的文件，别在别处 `mmap` 可执行内存。
2. **依赖走 `dependencies/prepare.sh`**：不要回退到 Homebrew / 全局 CMAKE_PREFIX_PATH。
3. **生成的产物不许手改**：内核补丁、ABI 副本、生成的 patch series，改完要重新生成。
4. **发布前跑检查**：`scripts/check-app-store-bundle.py`（上架用）、
   `Andarium/scripts/check-release-archive.py`（归档用）。侧载版要保留 JIT，别误跑上架安检。
5. **改工程文件后用格式化**：`xcrun xcprojformatter --update Andarium/Andarium.xcodeproj`。

## 五、构建镜像 vs 用预编译

- **预编译（推荐）**：直接下载 release 里的镜像，`stage-system-image.py` 装进去。
- **自建**：需高配 x86_64 Linux 机器，跑 `android/` 构建（数小时）。

## 六、发布（侧载）

- 推 tag → 本仓库 CI 产出未签名 IPA → 用户用 AltStore/SideStore 签名安装。
- 或本地 `xcodebuild archive` 后用你自己的证书导出，再发 IPA。

## 七、许可证注意

Kernarium 是 **GPL-2.0-or-later**。你的改动若链接/嵌入了它的 runtime，
整体需保持 GPL-2.0-or-later；**闭源商业发布需另购授权或规避 copyleft**。
