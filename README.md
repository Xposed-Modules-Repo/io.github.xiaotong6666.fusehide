# FuseHide

FuseHide 是一个面向 Android 12+ 的 LSPosed 模块与 MediaProvider/FUSE 调试工具。

模块会在 MediaProvider 进程中加载 `libfusehide.so`，并在 `libfuse_jni.so` 加载后安装 native hook。当前实现同时覆盖两类问题：

- `/storage/emulated/0` 下普通路径的运行时隐藏
- `Android/data`、`Android/obb` 相关 Unicode 场景的调试与修复

## 模块简介

FuseHide 提供运行时可配置的隐藏策略，用于对指定应用隐藏 `/storage/emulated/0` 下的目标路径，并提供与 MediaProvider 注入进程联动的调试能力。

主要功能包括：

- 对指定包名对应的 UID 生效
- 支持隐藏一级目录名，例如 `xinhao`、`MT2`
- 支持隐藏相对路径，例如 `Download/private`
- 支持“隐藏所有一级目录”模式，并允许配置例外项
- 支持在应用侧编辑配置，并热同步到已注入的 MediaProvider 进程
- 支持读取当前注入进程中的 native 配置快照
- 支持对目标路径执行 `stat`、`access`、`list`、`open`、`mkdir`、`rename` 等检测

## 作用域

推荐作用域：

- `com.android.providers.media.module`
- `com.google.android.providers.media.module`

## 工作方式

模块入口由 `assets/xposed_init` 指向 `io.github.xiaotong6666.fusehide.Entry`。

命中 MediaProvider 作用域后，模块会在目标进程中加载 `libfusehide.so`。native 层在检测到 `libfuse_jni.so` 被加载后执行 hook 安装。Java 层与注入进程通过广播、`HideConfigProvider`、`HideConfigRequestReceiver` 同步状态与隐藏配置。

普通路径隐藏使用 MediaProvider 的 FUSE 处理链路完成，不依赖 fuse-bpf。`Android/data`、`Android/obb` 使用单独的访问控制路径；模块在该部分保留了特殊路径判断相关 hook，用于修复 Unicode 可忽略码点绕过。

## 当前默认配置

当前默认值来自 native 层：

- `enableHideAllRootEntries=false`
- `hideAllRootEntriesExemptions=[Android]`
- `hiddenRootEntryNames=[xinhao, MT2]`
- `hiddenRelativePaths=[]`
- `hiddenPackages=[com.eltavine.duckdetector, io.github.xiaotong6666.fusehide, io.github.a13e300.fusehide]`

## 使用说明

1. 安装 APK。
2. 在 LSPosed 中启用 FuseHide。
3. 勾选推荐作用域。
4. 重启 MediaProvider 作用域进程，或直接重启设备。
5. 打开应用，确认 Hook 状态显示已 Hook。
6. 在“配置”页面编辑隐藏策略，并使用“应用”按钮同步到注入进程。
7. 在“检测”页面执行路径检查，验证隐藏结果是否符合预期。

## 调试信息

应用界面会显示以下系统属性：

- `ro.fuse.bpf.is_running`
- `persist.sys.vold_app_data_isolation_enabled`
- `external_storage.sdcardfs.enabled`

建议反馈问题时附带设备型号、系统版本、MediaProvider 版本、目标路径、目标包名、配置文本和 `adb logcat -s FuseHide` 输出。

## 源码与发布

- 源码仓库：https://github.com/XiaoTong6666/FuseHide
- 发布页面：https://github.com/XiaoTong6666/FuseHide/releases

## 许可证

- `app/src/main/cpp/third_party/xz-embedded/*` 保持其上游 `0BSD` 许可证
- 本项目其余部分采用 Apache License 2.0
