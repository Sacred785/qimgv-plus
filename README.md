# qimgv-plus

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL%203.0-blue.svg)](LICENSE)
[![Qt 6](https://img.shields.io/badge/Qt-6.12-green.svg)](https://www.qt.io/)
[![C++](https://img.shields.io/badge/C%2B%2B-17-orange.svg)]()
[![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey.svg)]()

> 基于 [easymodo/qimgv](https://github.com/easymodo/qimgv) 的增强分支：补齐简体中文、加入点击窗口边缘翻页、重做视频进度条，并让视频默认有声。

qimgv 是一款快速、简洁的图片浏览器，支持可选的 libmpv 视频播放。本分支针对 Windows 日常使用做了本地化与交互改造，基于上游最新代码（1.0.3 / Qt 6）构建。

---

## 目录

- [相比上游改了什么](#相比上游改了什么)
- [主要特性](#主要特性)
- [从源码构建](#从源码构建)
- [使用方法](#使用方法)
- [目录结构](#目录结构)
- [致谢与许可证](#致谢与许可证)

---

## 相比上游改了什么

| 项目 | 上游行为 | qimgv-plus |
| --- | --- | --- |
| 简体中文 | 有语言下拉框，但 `zh_CN.ts` 缺 60 条未翻译 | **380 条全部补齐（0 条未完成）**，默认跟随系统语言 |
| 视频默认声音 | `playVideoSounds` 默认 `false`（静音） | **默认开启**，设置项明确为「播放视频时默认开启声音」 |
| 点击边缘翻页 | 默认关闭，区域固定 110px，且存在判断 bug | **默认开启**；区域按窗口宽度 15%（最小 110px）实时计算；事件判定全部重写 |
| 视频进度条 | 固定 200px、无滑块 | 宽度随窗口自适应（约 85%）、圆角轨道、强调色填充、圆形悬停滑块 |
| 编译兼容性 | 与 exiv2 0.28 / GCC 16 / UNICODE 路径不兼容 | 修正 exif 的 UTF-8 路径与 `GetFileAttributesW` 调用 |

## 主要特性

- 左右方向键 / 滚轮翻页，点击窗口左、右边缘切换上一张 / 下一张
- 三种缩放模式（`1` / `2` / `3`）、空格轮换、按住右键拖动缩放
- 视频控制条：播放暂停、进度条、静音、逐帧前进 / 后退
- 缩略图面板、文件夹视图、快速复制 / 移动（`C` / `M`）
- 基础编辑：裁剪、旋转、调整大小；支持自定义脚本
- 设置界面可切换 English / 简体中文（跟随系统）

## 从源码构建

**环境**：Windows + [MSYS2](https://www.msys2.org/) UCRT64 工具链。

```bash
# 1. 安装依赖
pacman -S --needed \
  mingw-w64-ucrt-x86_64-{gcc,cmake,ninja,pkgconf,qt6-base,qt6-svg,qt6-tools,exiv2,mpv}

# 2. 配置（在源码根目录）
export PATH=/ucrt64/bin:$PATH
export PKG_CONFIG_LIBDIR=/ucrt64/lib/pkgconfig
cmake -G Ninja -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DUSE_QT5=OFF -DVIDEO_SUPPORT=ON -DEXIV2=ON -DOPENCV_SUPPORT=OFF ..

# 3. 编译
cmake --build build -j4
```

> OpenCV 仅用于「高质量缩放」滤镜，默认关闭；如需启用，安装 `mingw-w64-ucrt-x86_64-opencv` 并把 `-DOPENCV_SUPPORT` 设为 `ON`。

编译产物在 `build/qimgv/qimgv.exe`。发布时需要用 `windeployqt` 补齐 Qt 运行库，并一并放置 `translations/*.qm`、`plugins/player_mpv.dll` 及其依赖。预编译二进制**不纳入本仓库**（体积约 260 MB）。

## 使用方法

| 操作 | 快捷键 / 方式 |
| --- | --- |
| 打开设置 | `P` |
| 上一张 / 下一张 | `←` / `→`、滚轮、点击窗口左 / 右边缘 |
| 切换缩放模式 | `1` / `2` / `3`，`空格` 轮换 |
| 全屏 | 双击 / `F` / `F11` |
| 快速复制 / 移动 | `C` / `M`，再按 `1`–`9` |
| 文件夹视图 | `Enter` |

设置 → 常规 → **语言** 可切换 English / 简体中文；设置 → 视频播放 里的「播放视频时默认开启声音」控制新视频是否静音。

## 目录结构

```
qimgv-plus/
├── qimgv/                 主程序源码、资源与翻译（res/translations/zh_CN.ts）
│   ├── gui/               UI：主窗口、设置对话框、覆盖层、视频控制条
│   ├── components/        目录管理、动作管理
│   ├── sourcecontainers/  图片信息与 EXIF 读取
│   └── res/styles/       主题样式表
├── plugins/player_mpv/    libmpv 视频播放插件
├── scripts/               辅助脚本
├── README_UPSTREAM.md     上游原始说明文档
└── LICENSE                GPL-3.0
```

## 致谢与许可证

- 上游项目 [easymodo/qimgv](https://github.com/easymodo/qimgv)（作者 easymodo），本项目的一切都建立在它之上。上游作者利用业余时间维护且不以此盈利，请尊重其声明。
- 翻译工具链：Qt LinguistTools；视频播放：libmpv；EXIF：Exiv2；构建环境：MSYS2。
- 本分支沿用上游的 **GPL-3.0** 许可证，完整文本见 [LICENSE](LICENSE)。衍生版本需保持开源并使用相同协议。

> 本项目为非官方的第三方修改版，与上游作者无关。原始使用说明见 [README_UPSTREAM.md](README_UPSTREAM.md)。
