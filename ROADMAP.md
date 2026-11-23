# Roadmap（开发路线图）

> 目标：在 iOS 上本地运行 APK，不装传统虚拟机，不上架（侧载 + JIT）。

## M0 — 跑通基线 ✅（当前 fork）

- [x] fork Kernarium，获得可运行的 Android runtime + Andarium App
- [x] 中文上手 / 侧载 / 定制文档
- [x] CI 打包未签名 IPA（`.github/workflows/build.yml`）

## M1 — 可构建、可侧载

- [ ] 本地（Mac + Xcode 27）跑通 `xcodebuild`，产出 IPA
- [ ] AltStore / SideStore 侧载成功，StikDebug 开 JIT
- [ ] 从「文件」导入一个 APK 并成功运行

## M2 — 产品化改造（App 层）

- [ ] 换品牌 / 中文 UI
- [ ] 双击 APK 一键安装（`UIDocumentPickerViewController`）
- [ ] 自建"已装应用启动器"（图标 + 列表）
- [ ] 性能档位（省电/均衡/性能）一键切换
- [ ] 侧键浮层 + 性能浮层打磨

## M3 — 兼容性增强

- [ ] 覆盖 `.xapk` / `.apks` / `.apkm` 安装
- [ ] 集成 F-Droid 一键装源
- [ ] microG 可选（缓解 GMS 依赖）
- [ ] ADB over network 调试完善

## M4 — 工程化

- [ ] CI 支持 release 自动发布 IPA
- [ ] 侧载源 `sideload/source.json` 指向本仓库
- [ ] 崩溃/性能上报（可选，注意隐私）

## 已知硬限制（不做无谓投入）

- 只能前台运行（切后台暂停）
- 无完整 Google Play / GMS
- 性能约入门安卓机，3D 游戏吃力
- 银行/支付类 App 常检测模拟环境
