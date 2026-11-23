# 侧载指南（Sideload）

Andarium 已被 Apple 从 App Store 下架（违反 App Review Guidelines 2.5.2 + 1.1），
因此**只能侧载**。侧载不等于越狱，你的设备安全机制保持完整。

## 一、工具选择

| 工具 | 说明 |
|------|------|
| [AltStore](https://altstore.io) | 最常用，2.2+ 能保留更高内存上限 |
| [SideStore](https://sidestore.io) | 免电脑续签（默认不保留高内存上限） |

## 二、方式 A：加源（推荐，可自动更新）

在 AltStore / SideStore 的「Sources」里添加：

```
https://raw.githubusercontent.com/imxieyi/Kernarium/main/sideload/source.json
```

之后在源里找到 Andarium 安装即可。

## 三、方式 B：手动装 IPA

1. 从 [releases](https://github.com/imxieyi/Kernarium/releases) 下载 `Andarium-<version>.ipa`（约 1GB）；
2. 打开 AltStore / SideStore，在「My Apps」点 `+`，选中该 IPA；
3. 用你的 Apple ID 登录签名。

## 四、免费 Apple ID 的限制

- **7 天过期**：免费账号签名的 App 每 7 天要续签一次（AltStore/SideStore 会自动刷新）。
- **内存受限**：免费账号下 Android 可用内存更少；AltStore 2.2+ 保留更高上限，SideStore 默认不保留。
- **扩展虚拟地址**需要付费开发者账号。

## 五、启用 JIT（强烈建议）

上架版被 Apple 逼成纯解释器（慢）。侧载版可以开 **JIT**，性能显著提升：

1. 下载 [StikDebug](https://github.com/stikdebug/stikdebug)；
2. 在 StikDebug 里把 universal 脚本指向 Andarium；
3. **从 StikDebug 启动 Andarium**，等待「Preparing JIT」完成。

> 原理：iOS 不给普通 App 开 `MAP_JIT`，Kernarium 通过外部调试器 StikDebug，
> 用 `brk #0xf00d` 指令逐 16KiB 页「解锁」可执行内存，从而获得 JIT。

## 六、用本仓库 CI 打包的未签名 IPA

本仓库 `.github/workflows/build.yml` 会产出**未签名 IPA**（`Andarium-unsigned.ipa`）。
下载后同样在 AltStore/SideStore 里用你的 Apple ID 签名安装。

## 七、更新

- 加源方式：源里直接更新。
- 手动方式：用**同一个 Apple ID + 同一个工具**安装新 IPA，会替换旧版本并保留你的虚拟机数据。
