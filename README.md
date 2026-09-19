<div align="center">

<img src="assets/icon.png" width="96" alt="Sylphplay logo" />

# Sylphplay

**一款轻量、开源的本地多媒体播放器 —— 支持音频、视频与图片**

原生 Electron 桌面应用 · 不联网 · 无广告 · 无曲库

![Platform: Windows](https://img.shields.io/badge/platform-Windows-0078d6?style=flat\&logo=windows)
![Electron](https://img.shields.io/badge/Electron-33-47848f?style=flat\&logo=electron)
![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat)

</div>

***

## 项目定位

Sylphplay **不是**一个音乐播放器 —— 它同时面向**音频、视频、图片**三类媒体：

- 音频 / 视频：完整的播放控制（倍速、画质调节、全屏、进度条悬停预览）

- 图片：支持鹰眼图（Eagle Eye）缩略预览、视窗跟随拖动

- 歌词：支持 LRC / 卡拉 OK 逐词歌词，桌面歌词悬浮窗

它是一款**强调本地、坚守初心**的个人项目：**没有**在线曲库、音乐市场或任何付费/版权依赖，也不打算与 "云音乐" 类产品竞争。详见 [CONTRIBUTING.md](#贡献) 中的方向约束。

***

## 功能特性

- 多格式媒体播放：音频 / 视频 / 图片一体体验

- 本地优先：支持拖拽文件、文件夹、浏览器标签页 URL 进播放

- 播放队列：可持久化（重启保留）、支持拖拽排序、不切换当前播放

- 默认打开方式：通过内置 .NET helper 一键关联 38 种媒体格式（免管理员权限）

- 桌面歌词：可随窗口拖动、独立置顶小窗，双语 LRC 与逐字填充

- 全屏模式：仅显示媒体内容，鼠标移动至边缘才浮现工具栏，超时自动隐藏

- 图片鹰眼图：缩略图 + 视窗框拖动定位

- 深浅主题切换

- 实验特性（设置中开启）：倍速滑杆、全歌逐行播放

- 常驻托盘：关闭窗口后后台继续播放，支持单实例转发 `以 Sylphplay 打开`

***

## 运行环境与依赖

- OS：Windows 10 / 11

- Node.js：18+（推荐 20+）

- 包管理器：**pnpm**（项目强制使用，见 [.npmrc](.npmrc)）

- .NET SDK：**net10.0**（仅构建 `assoc-helper` 需要，用于生成文件关联工具）

### 技术栈

| 层      | 技术                                |
| ------ | --------------------------------- |
| 桌面框架   | Electron 33                       |
| 界面     | 原生 HTML / CSS / JavaScript（无前端框架） |
| 图标     | Font Awesome Free 6               |
| 文件外联工具 | C# / .NET（`assoc-helper`，自包含单文件）  |
| 打包     | electron-builder（NSIS 安装包）        |

***

## 快速开始

```bash
# 1. 安装依赖（务必使用 pnpm）
pnpm install

# 2. 开发模式启动
pnpm start          # 生产模式
pnpm dev            # 开发模式（带 --dev 标记，跳过文件关联工具）

# 3. 构建安装包（会自动先编译 .NET helper 再做 electron-builder）
pnpm dist
```

> Windows 下 Electron 二进制下载变慢时，可在 [.npmrc](.npmrc) 中调整 electron 镜像源。

***

## 项目结构

```
.
├── main.js                  # 主进程：窗口管理、系统集成、IPC、托盘、文件关联
├── preload.js               # 预加载脚本：安全暴露 window.sylph API
├── package.json             # 依赖 / 打包配置（electron-builder）
├── pnpm-workspace.yaml      # pnpm 工作区定义
├── .npmrc                   # pnpm / Electron 镜像配置
├── app/
│   ├── index.html           # 主界面（HTML 骨架）
│   ├── styles.css           # 主界面样式
│   ├── renderer.js          # 渲染进程逻辑（媒体/队列/歌词/实验特性）
│   ├── lyrics.html          # 桌面歌词悬浮窗
│   └── sylph.svg            # 矢量品牌资源
├── assets/                  # 图标（icon.ico / icon.png）等静态资源
├── assoc-helper/            # C# 文件关联工具源（net10，自包含免依赖）
└── dist/                    # electron-builder 输出目录（构建生成，勿提交）
```

***

## 设置默认打开方式

1. 打开应用 → 设置 → 「设为默认打开方式」
2. 应用会调用 `assoc-helper`，在**当前用户**级别注册 38 种媒体格式的关联（无需管理员权限）
3. 之后在资源管理器右键媒体文件选择 **以 Sylphplay 打开**，或右键 **加入到 Sylphplay 播放队列**

> 开发环境（`pnpm dev`）不会执行 .NET helper，点击该按钮会提示「仅打包版支持」，以保证零依赖的纯开发体验。

***

## 架构说明

Sylphplay 采用 Electron 经典三进程模型：

- **主进程（main.js）**：仅处理系统集成 —— 窗口生命周期、单实例锁、托盘、系统打开转发、文件关联 IPC、后台节流。

- **渲染进程（app/）**：全部业务 UI 与媒体逻辑，通过 `window.sylph`（[preload.js](preload.js) 暴露）与主进程通信。

- **辅助进程 corp（assoc-helper）**：独立 .NET 单文件可执行，仅在用户主动点击「设为默认打开方式」时被主进程调用。

音频后端基于 Web Audio / HTML5 media 元素。已知限制：独占模式或默认设备切换下的 WASAPI 端点抢占可能造成短暂音频中断，属 Chromium 硬限制，无法在 JS 层修复。

***

## 贡献

欢迎提交 Issue 与 Pull Request！在动手前请先阅读：

- [CONTRIBUTING.md](CONTRIBUTING.md) —— 贡献规范与产品方向约束

### 对「加曲库」类 PR 的说明

Sylphplay 是一个定位明确的本地播放器（音频 + 视频 + 图片）。**我们不会**添加音乐市场、在线曲库或付费/版权内容功能 —— 这既出于个人项目的版权与付费现实，也为了避免偏离「不只做音乐」的产品初心。此类请求会被友好地关闭，感谢理解。

***

## 开源协议

本项目基于 [MIT](LICENSE) 协议开源。

> 媒体内容版权归各自版权方所有，Sylphplay 仅提供本地播放能力，不包含任何受版权保护的内容或在线资源。

