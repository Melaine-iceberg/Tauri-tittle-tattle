# Tauri 应用开发中通知显示为 Windows PowerShell 5.1 的问题

## 问题描述

在 Tauri 应用开发阶段，通知显示的是 Windows PowerShell 5.1 的名称，而不是当前应用的名称。

## 原因

这是 **Windows 系统层面的限制**，不是 bug，且几乎所有 Tauri 应用在开发阶段都会遇到。

### Windows 通知系统要求

Windows 的原生 Toast 通知（WinRT）要求应用必须有一个已在系统中**注册过的 `AppUserModelID` (AUMID)**，通知才能显示成对应程序的名字和图标。而这个 AUMID 通常是由「安装程序」在创建开始菜单快捷方式时写入注册表的。

### Tauri 插件的实现逻辑

`tauri-plugin-notification` 在 Windows 上的实现逻辑如下：

1. **检测运行环境**：插件会检测当前运行的可执行文件是否位于 `target/` 目录下（判断是否为开发/未安装状态）
2. **开发模式回退**：如果是在开发模式下（如使用 `bun run tauri dev` 或直接运行 `src-tauri/target/debug/learn.exe` / `target/release/learn.exe`），插件无法注册有效的 AUMID，会**回退使用 Windows 系统自带的 PowerShell 5.1 的 AUMID**
   - 这是一个"保底方案"，因为 PowerShell 5.1 的 AUMID 在所有 Windows 系统上都存在且必定能弹出 Toast
3. **正式安装后正常使用**：只有当应用通过 `tauri build` 生成的 **NSIS/MSI 安装包正式安装**后，安装程序会在开始菜单创建带有正确 AUMID 的快捷方式（AUMID 对应 `tauri.conf.json` 里的 `identifier` 字段），插件才会用 `app_id(identifier)` 正确关联到应用

### 官方确认

这在 Tauri 官方仓库里是被反复确认过的已知行为，相关 issue 包括：

- tauri-apps/tauri#3700
- tauri-apps/tauri#5606
- plugins-workspace#2156

## 解决方案

在开发阶段，可以通过以下方式解决：

1. **使用正式安装包测试**：通过 `tauri build` 生成安装包并安装后测试通知功能
2. **配置开发环境 AUMID**：在开发阶段手动配置 AUMID（需要修改注册表，不推荐）
3. **接受开发阶段限制**：理解这是开发阶段的正常现象，正式安装后会自动解决
